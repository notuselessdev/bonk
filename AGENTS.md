# AGENTS.md

> Guidelines for AI agents working in this repository.

## Project Overview

**bonk** is a macOS CLI tool that detects physical hits/slaps on Apple Silicon MacBooks via the accelerometer and plays audio responses. Single-file Go application with embedded MP3 assets. A NotUseless project.

- **Platform**: macOS on Apple Silicon (M2+) only
- **Runtime requirement**: `sudo` (for IOKit HID accelerometer access)
- **Architecture**: Single `main.go` file with embedded audio assets
- **Distribution**: Homebrew via `notuseless/tap`, GoReleaser for releases

## Commands

### Build & Run

```bash
go build -o bonk .
sudo ./bonk
sudo ./bonk --sexy
sudo ./bonk --halo
sudo ./bonk --custom /path/to/mp3s
```

### Install

```bash
brew tap notuselessdev/tap
brew install bonk
```

Or from source:

```bash
go install github.com/notuselessdev/bonk@latest
```

### Release

Releases are automated via GitHub Actions + GoReleaser Pro when a `v*` tag is pushed:

```bash
git tag v1.0.0
git push origin v1.0.0
```

GoReleaser also publishes the Homebrew formula to `notuselessdev/homebrew-tap`.

## Code Organization

```
bonk/
├── main.go              # All application code (single file)
├── stdin_test.go        # Tests for stdin JSON commands + amplitudeToVolume
├── audio/
│   ├── pain/            # Default "ow!" responses (10 MP3s)
│   ├── sexy/            # Escalating responses (60 MP3s)
│   └── halo/            # Halo death sounds (9 MP3s)
├── go.mod
├── .goreleaser.yaml     # Release config (includes Homebrew tap)
└── .github/workflows/   # CI/CD
```

## Key Dependencies

| Package | Purpose |
|---------|---------|
| `github.com/taigrr/apple-silicon-accelerometer` | Reads accelerometer via IOKit HID |
| `github.com/gopxl/beep/v2` | Audio playback (MP3 decoding, speaker output) |
| `github.com/spf13/cobra` | CLI framework |
| `github.com/charmbracelet/fang` | CLI config/execution wrapper |

## Homebrew Distribution

The `.goreleaser.yaml` has a `brews` section that auto-pushes a formula to `notuselessdev/homebrew-tap`. Required secrets:
- `HOMEBREW_TAP_GITHUB_TOKEN`: PAT with write access to the tap repo
- `GH_PAT`: PAT for private accelerometer dependency
- `GORELEASER_KEY`: GoReleaser Pro license key

## Gotchas

1. **Root required**: Must run with `sudo` for IOKit HID access.
2. **Apple Silicon only**: Only builds for `darwin/arm64`.
3. **Private dependency**: `github.com/taigrr/apple-silicon-accelerometer` requires `GOPRIVATE` and a GitHub PAT.
4. **Single file**: All code is in `main.go`.
5. **Mutually exclusive modes**: `--sexy`, `--halo`, and `--custom` cannot be combined.
