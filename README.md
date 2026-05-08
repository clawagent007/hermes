# hermes on termux

삼성 S22의 Termux 환경에 `hermes-agent`를 설치한 기록과 초기 설정 메모를 정리한 문서입니다.

참고한 설치 안내: [Termux 설치 및 초기 설정 방법](https://lv12.tistory.com/m/147)

## Termux 설치 먼저 하기

Termux는 *Google Play 버전*보다 *GitHub Release*나 *F-Droid* 버전을 사용하는 것이 좋습니다.

- GitHub Release: https://github.com/termux/termux-app/releases
- F-Droid: https://f-droid.org/ko/packages/com.termux/

설치 후 처음 실행하면 아래처럼 기본 안내가 나옵니다.

```bash
pkg search <query>
pkg install <package>
pkg upgrade
```

### 먼저 해둘 것

1. Termux를 실행합니다.
2. 저장소와 파일 접근 권한을 준비합니다.

```bash
termux-setup-storage
termux-change-repo
```

3. 파란 화면에서 `Mirror Group > All Mirrors`를 선택합니다.
4. 기본 패키지를 업데이트합니다.

```bash
pkg update && pkg upgrade -y
```

## Hermes Agent 설치

설치는 아래 명령으로 진행했습니다.

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

### Termux에서 감지된 환경

- Android / Termux
- Python 3.13.13
- Git 2.54.0
- Node.js는 `pkg`로 설치
- ffmpeg 8.1 존재

### Termux에서의 설치 특징

- `uv` 대신 Python 표준 `venv` + `pip` 사용
- 필요한 Termux 패키지 설치:
  - `clang`
  - `rust`
  - `make`
  - `pkg-config`
  - `libffi`
  - `openssl`
  - `ripgrep`

## 기기 정보

- 기기: Samsung S22
- 실행 환경: Termux

### Termux 리소스 스냅샷

- CPU: 4코어
- 메모리: 7.1Gi total / 4.5Gi used / 2.4Gi available
- Swap: 8.0Gi total / 2.5Gi used
- 디스크(`/`): 6.1G 중 6.1G 사용, 100%
- 로드 평균: 4.78 / 4.23 / 4.11

현재 상태: 루트 디스크가 거의 가득 찼습니다.

## 메모

- 이 저장소는 Termux 전용 설치 노트, 문제 해결 기록, Hermes 설정 메모를 관리하는 용도로 사용할 수 있습니다.
- 다음 단계로는 브라우저 도구, 인증, Android/Termux 특이사항을 확인하면 좋습니다.
