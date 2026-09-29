# reSpeaker Flex XVF3800 Firmware

## Introduction

This directory contains firmware images for the reSpeaker Flex XVF3800, in `usb/` (USB DFU images) and `i2s/` (I2S slave images). Select the firmware image that matches the required interface, mic geometry, sample rate, and number of capture channels.

Firmware file names follow the pattern:

```
respeaker_flex_<interface>_<geometry><sample-rate><channels>_v<x.y.z>.bin
```

- **interface**: `usb` or `i2s`
- **geometry**: `c` — 4-mic circular array (44 mm spacing); `l` — 4-mic linear array (33 mm spacing)
- **sample-rate**: `16k` — 16 kHz; `48k` — 48 kHz
- **channels**: `2ch` — processed stereo output; `6ch` — USB only, processed stereo plus raw microphones

The six-channel USB firmware exposes the normal configured stereo outputs on channels 1 and 2 (1-based numbering), and the four raw microphone signals on channels 3 through 6. The raw microphone data has not undergone voice-enhancement processing, so its level is normally lower than that of the processed outputs. If the captured audio is used only for listening or speech recognition, its output level can be increased manually in the host software. For algorithm development, acoustic measurements, or calibration, keeping the original level is recommended to preserve the raw signal characteristics and the relative levels between microphones.

## Changelog

### v1.0.4 (Current)

#### Added

- Added playback level control for the on-board AIC3104 audio codec. The new `AIC3104_HP_LEVEL` and `AIC3104_LINEOUT_LEVEL` commands set or get the headphone and line-out output levels independently. Valid range for both commands: [0 .. 9].

### v1.0.3

#### Fixed

- Fixed restoration of saved fixed-beam settings after startup. Fixed-beam enable and gating settings are reapplied after AEC initialization, and changes to fixed-beam azimuth, elevation, and gating are retained in the runtime configuration.

### v1.0.2

#### Added

- Added configurable USB direct-output routing for channels 3 through 6 on the six-channel firmware. The new `AUDIO_MGR_OP_CH3`, `AUDIO_MGR_OP_CH4`, `AUDIO_MGR_OP_CH5`, and `AUDIO_MGR_OP_CH6` commands allow each channel's source to be selected independently, with support for product defaults and saved configurations. Legacy saved audio configurations are migrated automatically by filling the new fields from defaults.
- Updated the bootloader to v4.

#### Fixed

- Improved button handling during initialization to make entering safe mode more reliable.

### v1.0.1

#### Fixed

- Fixed USB audio recovery after bus resets and speed changes. Endpoint descriptors, packet sizes, FIFOs, and buffers are now restored for the active bus speed.

### v1.0.0

#### Added

- Initial release for reSpeaker Flex.
- Added per-scenario control parameter profiles (`i2s_aitool_v1`, `i2s_smarthome_v2`, `usb_aitool_v1`, `usb_smarthome_v2`) so tuning values can be selected for different applications.
- Added the `BOOT_VERSION`, `JUMP_TO_SAFEMODE`, and `JUMP_TO_APP` commands, and updated the bootloader to v3.
- Updated the acoustic model (nlmodel) based on the Seeed Bazaar 4o5w speaker.

## Releasing

Pushing a version tag (`v*`) triggers the [release workflow](../.github/workflows/release.yml), which creates the GitHub release with all firmware binaries matching the tag attached and uses the tag's changelog section above as the release notes. To publish a new version: commit the binaries and the changelog entry, then tag and push, for example `git tag v1.0.5 && git push origin v1.0.5`.
