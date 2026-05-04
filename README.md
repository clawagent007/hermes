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
