---
layout: post
title: "F-Droid Termux에 uv로 Hermes Agent 설치하기"
description: "Python 3.14 기본 환경을 피하고 Python 3.13과 uv로 Hermes를 설치·검증하는 재현 가능한 절차"
date: 2026-08-31 09:00:00 +0900
tags: [hermes-agent, termux, android, uv, installation]
---

> 검증 환경: Samsung SM-F907N, Android 12, F-Droid Termux 0.119.0-beta.3, ARM64. 시스템 Python 3.14.6을 유지하면서 Python 3.13.13과 Termux uv 0.12.7로 Hermes Agent 0.20.6 설치 및 실행을 확인했다.

## 설치 전략

1. 기본 Python이 Hermes 지원 범위이면 [공식 설치 스크립트](https://hermes-agent.nousresearch.com/install.sh)를 우선한다.
2. Python 3.14가 Hermes의 `<3.14` 조건에 걸릴 때만 Python 3.13 병행 경로를 사용한다.
3. 처음에는 `.[termux]` 기본 번들을 설치·검증한다.
4. Google·Home Assistant·SMS·웹 문서 변환·PTY가 필요할 때만 `.[termux-all]`을 추가한다.
5. 오류가 발생하면 문제해결 문서를 확인한다.

## 0. 장시간 작업 준비

```bash
termux-wake-lock
sshd
```

SSH 사용 시 keepalive를 켠다.

```bash
ssh -o ServerAliveInterval=30 \\
    -o ServerAliveCountMax=20 \\
    -p 8022 USER@PHONE_IP
```

## 1. 환경 확인

```bash
termux-info
python --version
apt-cache policy python python3.13 uv
command -v git
command -v curl
command -v clang
command -v rustc
df -h "$HOME"
```

기본 Python이 Hermes 지원 범위이면 다음 공식 설치를 사용한다.

```bash
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash
```

다음 오류가 날 때만 수동 경로를 계속한다.

```text
Package 'hermes-agent' requires a different Python:
3.14.x not in '<3.14,>=3.11'
```

## 2. Python 3.13·uv·빌드 도구 설치

```bash
pkg install -y \\
  python3.13 uv git curl clang rust \\
  rust-std-aarch64-linux-android \\
  make pkg-config libffi openssl ca-certificates
```

```bash
python3.13 --version
uv --version
rustc --version
dpkg-query -W rust-std-aarch64-linux-android
```

`rustc`와 `rust-std-aarch64-linux-android`의 버전은 같아야 한다.

## 3. Hermes 소스 준비

```bash
test -d "$HOME/.hermes/hermes-agent/.git"
```

없을 때만 복제한다.

```bash
mkdir -p "$HOME/.hermes"
git clone https://github.com/NousResearch/hermes-agent.git \\
  "$HOME/.hermes/hermes-agent"
cd "$HOME/.hermes/hermes-agent"
```

## 4. Python 3.13 venv 생성

```bash
cd "$HOME/.hermes/hermes-agent"
rm -rf venv
uv venv --python "$(command -v python3.13)" venv
venv/bin/python --version
```

기대 결과는 `Python 3.13.x`이다.

## 5. Android 빌드 환경

```bash
. venv/bin/activate
export ANDROID_API_LEVEL="$(getprop ro.build.version.sdk)"
export VIRTUAL_ENV="$PWD/venv"
export CARGO_BUILD_JOBS=1
export CMAKE_BUILD_PARALLEL_LEVEL=1
export MAKEFLAGS=-j1
```

## 6. psutil Android 호환 설치

```bash
uv pip install \\
  -p venv/bin/python \\
  setuptools wheel cython \\
  --no-python-downloads \\
  --link-mode=copy

venv/bin/python scripts/install_psutil_android.py --uv
venv/bin/python -c 'import psutil; print(psutil.__version__)'
```

## 7. 기본 Termux 번들 설치

```bash
uv pip install \\
  -p venv/bin/python \\
  --python-platform aarch64-linux-android \\
  -e ".[termux]" \\
  -c constraints-termux.txt \\
  --no-python-downloads \\
  --link-mode=copy
```

`--python-platform aarch64-linux-android`는 필수다. 생략하면 uv가 빌드한 Android wheel을 스스로 거부할 수 있다.

## 8. 선택: termux-all 설치

기본 검증이 끝난 뒤 추가 기능이 필요할 때만 실행한다.

```bash
uv pip install \\
  -p venv/bin/python \\
  --python-platform aarch64-linux-android \\
  -e ".[termux-all]" \\
  -c constraints-termux.txt \\
  --no-python-downloads \\
  --link-mode=copy
```

`firecrawl-anydoc`의 Rust Thin-LTO 링크는 오래 걸릴 수 있다. CPU 사용이 계속된다면 중단하지 않는다.

```bash
pgrep -af "uv pip|cargo|rustc|maturin"
free -h
```

## 9. 런처·스킬·비밀정보 파일

```bash
cd "$HOME/.hermes/hermes-agent"
ln -sf "$PWD/venv/bin/hermes" "$PREFIX/bin/hermes"
mkdir -p "$HOME/.hermes/skills"
venv/bin/python tools/skills_sync.py

umask 077
touch "$HOME/.hermes/.env"
chmod 600 "$HOME/.hermes/.env"
```

## 10. 필수 검증

```bash
command -v hermes
hermes --version
hermes --help >/dev/null
```

```bash
venv/bin/python -c '
import cryptography, jiter, psutil, pydantic_core
import rpds, watchfiles
import importlib.util as u
print("cryptography", cryptography.__version__)
print("psutil", psutil.__version__)
print("pydantic_core", pydantic_core.__version__)
print("anydoc", bool(u.find_spec("anydoc")))
print("native imports: PASS")
'

hermes doctor
```

다음 항목이 통과해야 한다.

- Python 3.13
- 가상환경과 버전 파일
- SSL과 필수 패키지
- `venv/bin/hermes`
- `$PREFIX/bin/hermes` 링크

다음 메시지는 설치 오류가 아니라 후속 인증 단계다.

```text
Run 'hermes setup' to configure API keys
```

## 업데이트 주의

시스템 기본 Python이 3.14인 동안에는 업데이트 전에 Hermes의 Python 지원 범위를 확인한다. 업데이트 후에는 네이티브 import와 `hermes doctor`를 다시 실행한다.

## 참고

- [Hermes Agent 공식 문서](https://hermes-agent.nousresearch.com/docs)
- [Hermes Agent GitHub](https://github.com/NousResearch/hermes-agent)
