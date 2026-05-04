# Hermes Agent on Android/Termux

This repository records the first installation attempt and notes for `hermes-agent` on Android/Termux.

## Installation summary

Installed with:

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

### Detected environment

- Android / Termux
- Python 3.13.13
- Git 2.54.0
- Node.js installed via `pkg`
- ffmpeg 8.1 present

### Installer behavior on Termux

- Used Python's stdlib `venv` + `pip` instead of `uv`
- Installed required Termux packages:
  - `clang`
  - `rust`
  - `make`
  - `pkg-config`
  - `libffi`
  - `openssl`
  - `ripgrep`

### Notes

- This repository can be used to track Termux-specific setup notes, fixes, and workflows for Hermes Agent.
- Next steps may include verifying browser tools, auth, and any Android-specific quirks.

## Device notes

Installed on a Samsung S22 running Termux.

### Termux resource snapshot

- CPU: 4 cores
- Memory: 7.1Gi total / 4.5Gi used / 2.4Gi available
- Swap: 8.0Gi total / 2.5Gi used
- Disk (/): 6.1G used out of 6.1G total, 100%
- Load average: 4.78 / 4.23 / 4.11

Current state: the root disk is almost full.
