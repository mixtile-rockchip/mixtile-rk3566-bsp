# HDMI eARC Audio Capture Guide

## 1. Overview

HDMI eARC can be used as an audio capture interface on the RK3566 SoM platform. After the TV and board are connected correctly, the incoming PCM audio stream from the TV can be recorded through the ALSA capture device.

## 2. Verify the Capture Device

Use the following command to check the available recording devices:

```bash
arecord -l
```

Example output:

```text
**** List of CAPTURE Hardware Devices ****
card 0: HDMIeARC [HDMI-eARC], device 0: fe410000.i2s-hdmi-earc hdmi-earc-0 [fe410000.i2s-hdmi-earc hdmi-earc-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: ES9821 [ES9821], device 0: fe430000.i2s-es9821 es9821-0 [fe430000.i2s-es9821 es9821-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

In this example, the HDMI eARC capture device is `card 0, device 0`.

## 3. Record Audio from HDMI eARC

Use `card 0, device 0` to record audio:

```bash
arecord -D hw:0,0 -r 48000 -f s32_le -c 2 output.wav
```

Parameter description:

- `-D hw:0,0`: use HDMI eARC capture device
- `-r 48000`: sample rate 48 kHz
- `-f s32_le`: 32-bit little-endian PCM
- `-c 2`: 2 channels

After recording, the generated `output.wav` can be:

- played back through another local audio output interface
- pulled to a host PC for playback and analysis

## 4. TV Side Configuration Notes

The TV must be configured correctly, otherwise the board may not capture valid audio.

Required settings:

- Enable `HDMI-eARC`
- Enable `HDMI-CEC`
- Set digital audio output format to `PCM`

Samsung TV example paths:

- `Settings -> External Device Manager` for `HDMI-CEC`
- `Settings -> Sound -> Expert Settings -> HDMI-eARC Mode`
- `Settings -> Sound -> Sound Output -> Receiver (HDMI-eARC)`

## 5. Notes

- The actual ALSA card index may vary depending on the full audio configuration of the system. Always confirm with `arecord -l` before recording.
- If no valid audio is captured, first check the HDMI cable, TV eARC/CEC settings, and whether the TV output format is PCM.
