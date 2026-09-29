
# reSpeaker XVF3800 Host Control Tool

A Python implementation of the `xvf_host` application, which provides the convenience of controlling and monitoring the reSpeaker Flex on any platform.

## System Requirements

- Python 3
- pyusb library
- libusb library (on Windows, install the `libusb-package` pip package)

## Installation & Dependencies

```bash
# Install Python dependencies
pip install pyusb
```

## Usage

### Basic Syntax

```bash
python xvf_host.py [options] COMMAND [--values value(s)...]
```

### Options

- `-l, --list`: List all supported commands with detailed information
- `--vid`: Set USB vendor ID (default: 0x2886)
- `--pid`: Set USB product ID; if omitted, auto-discover by VID
- `--values`: Provide values for write commands; supports decimal (`123`, `1.5`) and hex (`0x7B`, `$7B`) formats

### Usage Examples

#### 1. List all available commands

```bash
python xvf_host.py --list
```

#### 2. Read firmware version information

```bash
python xvf_host.py VERSION
```

#### 3. Read DOA (Direction of Arrival) values

```bash
python xvf_host.py DOA_VALUE
```

#### 4. Set LED color (hexadecimal format)

```bash
python xvf_host.py LED_COLOR --values 0xFF0000
```

#### 5. Set LED brightness

```bash
python xvf_host.py LED_BRIGHTNESS --values 50
```

#### 6. Read microphone array geometry

```bash
python xvf_host.py AEC_MIC_ARRAY_GEO
```

## Other Scripts in This Directory

- `respeaker_get_doa.py` — Minimal example that polls the DOA angle (0–359°) and the speech-detected flag in a loop:

  ```bash
  python respeaker_get_doa.py --interval 0.2
  ```

## USB Output Channel Selection

This firmware routes signals to the stereo USB output via `AUDIO_MGR_OP_L` / `AUDIO_MGR_OP_R`, each taking two values: `category` (a signal group) and `source` (a signal within that group). Microphone and beam source indexes are 0-based. In packed mode, three sources can be multiplexed per channel through the `AUDIO_MGR_OP_L_PK0/1/2` and `AUDIO_MGR_OP_R_PK0/1/2` slots; `AUDIO_MGR_OP_ALL` sets all six slots at once.

Omit `--values` to read the current routing; provide exactly two values to change it:

```bash
python xvf_host.py AUDIO_MGR_OP_L
python xvf_host.py AUDIO_MGR_OP_L --values 6 3
python xvf_host.py AUDIO_MGR_OP_R --values 1 0
```

The last two commands route the automatically selected beam to the left channel and raw microphone 0 to the right channel.

### Signal Categories and Sources

The following summarizes the [XMOS Output Selection reference](https://www.xmos.com/documentation/XM-014888-PC/html/modules/fwk_xvf/doc/user_guide/03_using_the_host_application.html#output-selection). Signal availability depends on the firmware configuration.

| Category | Signal                                               | Source                                                             |
| -------- | ---------------------------------------------------- | ------------------------------------------------------------------ |
| `0`    | Silence                                              | `0`                                                              |
| `1`    | Microphones without gain or delay                    | `0`–`3`                                                       |
| `2`    | Unpacked microphones; undefined without packed input | `0`–`3`                                                       |
| `3`    | Microphones with gain and delay                      | `0`–`3`                                                       |
| `4`    | Reference, converted to 16 kHz                       | `0`                                                              |
| `5`    | Reference with delay                                 | `0`                                                              |
| `6`    | Processed beams                                      | `0`, `1`: focused; `2`: scanning; `3`: automatic selection |
| `7`    | AEC residuals or ASR beams                           | `0`–`3`                                                       |
| `8`    | User-selected outputs                                | `0`, `1`                                                       |
| `9`    | Outputs after SHF DSP                                | `0`–`3`                                                       |
| `10`   | Reference at interface rate                          | `0`–`5` at 48 kHz; `0`, `1` at 16 kHz                     |
| `11`   | Microphones with gain, before delay                  | `0`–`3`                                                       |
| `12`   | Reference with gain and delay                        | `0`                                                              |

Raw microphone streams bypass voice enhancement and normally have lower levels than processed audio. For algorithm development, acoustic measurements, or calibration, preserve the original levels and relative microphone levels.

After checking the routing, save the current configuration to flash if it should survive a restart:

```bash
python xvf_host.py SAVE_CONFIGURATION --values 1
```

This saves the current configuration, including other supported persistent settings, not just the last channel selection. Without saving, a restart restores the previously saved configuration or the firmware defaults.

## Output Format

### Read Operation Output

- **LED commands** (LED_COLOR, LED_DOA_COLOR, LED_RING_COLOR):

  ```
  LED_COLOR: [0x00FF00]
  ```
- **Floating-point numbers**: Display with 3 decimal places

  ```
  AEC_MIC_ARRAY_GEO: [0.033, -0.033, 0.000, 0.033, 0.033, 0.000, -0.033, 0.033, 0.000, -0.033, -0.033, 0.000]
  ```
- **Integers and strings**: Maintain original format

  ```
  VERSION: [1, 0, 3]
  ```
