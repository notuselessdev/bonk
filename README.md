# bonk

Bonk your MacBook, it yells back. A [NotUseless](https://notuseless.com) project.

Uses the Apple Silicon accelerometer (Bosch BMI286 IMU via IOKit HID) to detect physical hits on your laptop and plays audio responses. Single binary, no dependencies.

## Requirements

- macOS on Apple Silicon (M2+)
- `sudo` (for IOKit HID accelerometer access)
- Go 1.26+ (if building from source)

## Install

### Homebrew

```bash
brew tap notuselessdev/tap
brew install bonk
```

### Download

Download from the [latest release](https://github.com/notuselessdev/bonk/releases/latest).

### Build from source

```bash
go install github.com/notuselessdev/bonk@latest
```

> **Note:** `go install` places the binary in `$GOBIN` or `$(go env GOPATH)/bin`. Copy it so `sudo bonk` works:
>
> ```bash
> sudo cp "$(go env GOPATH)/bin/bonk" /usr/local/bin/bonk
> ```

## Usage

```bash
# Normal mode — says "ow!" when bonked
sudo bonk

# Sexy mode — escalating responses based on bonk frequency
sudo bonk --sexy

# Halo mode — plays Halo death sounds when bonked
sudo bonk --halo

# Fast mode — faster polling and shorter cooldown
sudo bonk --fast
sudo bonk --sexy --fast

# Custom mode — plays your own MP3 files from a directory
sudo bonk --custom /path/to/mp3s

# Adjust sensitivity (lower = more sensitive)
sudo bonk --min-amplitude 0.1
sudo bonk --min-amplitude 0.25

# Set cooldown in milliseconds (default: 750)
sudo bonk --cooldown 600

# Set playback speed (default: 1.0)
sudo bonk --speed 0.7   # slower and deeper
sudo bonk --speed 1.5   # faster
```

### Modes

**Pain mode** (default): Randomly plays from 10 pain/protest audio clips when a hit is detected.

**Sexy mode** (`--sexy`): Tracks bonks within a rolling window. The more you bonk, the more intense the audio response. 60 levels of escalation.

**Halo mode** (`--halo`): Randomly plays death sound effects from the Halo video game series.

**Custom mode** (`--custom`): Randomly plays MP3 files from a directory you specify.

### Detection tuning

Use `--fast` for quicker detection: faster polling (4ms vs 10ms), shorter cooldown (350ms vs 750ms), higher sensitivity, and larger sample batches.

Override individual values with `--min-amplitude` and `--cooldown`.

### Sensitivity

Control detection sensitivity with `--min-amplitude` (default: `0.05`):

- `0.05-0.10`: Very sensitive, detects light taps
- `0.15-0.30`: Balanced sensitivity
- `0.30-0.50`: Only strong impacts trigger sounds

## Running as a Service

```bash
sudo tee /Library/LaunchDaemons/com.notuselessdev.bonk.plist > /dev/null << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.notuselessdev.bonk</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/bonk</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/tmp/bonk.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/bonk.err</string>
</dict>
</plist>
EOF

# Load
sudo launchctl load /Library/LaunchDaemons/com.notuselessdev.bonk.plist

# Unload
sudo launchctl unload /Library/LaunchDaemons/com.notuselessdev.bonk.plist
```

## How it works

1. Reads raw accelerometer data via IOKit HID (Apple SPU sensor)
2. Runs vibration detection (STA/LTA, CUSUM, kurtosis, peak/MAD)
3. When a significant impact is detected, plays an embedded MP3 response
4. Optional volume scaling (`--volume-scaling`) — light taps play quietly, hard hits play loud
5. Optional speed control (`--speed`) — adjusts playback speed and pitch
6. 750ms cooldown between responses (adjustable with `--cooldown`)

## License

MIT
