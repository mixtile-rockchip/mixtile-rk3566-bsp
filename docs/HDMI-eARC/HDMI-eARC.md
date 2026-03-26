# HDMI eARC Audio Capture Guide

## 1. Overview

HDMI eARC can be used as an audio capture interface on the RK3566 SoM platform. After the TV and board are connected correctly, the incoming PCM audio stream from the TV can be recorded through the ALSA capture device.

## 2. Device Tree (DTS) Configuration

The RK3566 BSP already includes an HDMI eARC audio capture path in the board DTS. The relevant source file is:

`kernel-6.1/arch/arm64/boot/dts/rockchip/rk3566-autonomic-m1.dts`

Key points in the DTS:

- `earc_sound` is defined as a `simple-audio-card` and exposed as `HDMI-eARC`
- the CPU DAI uses `i2s1_8ch`
- the eARC receiver codec uses `earc_rx_codec`
- `earc_rx_codec` is implemented by the `ite,it6620` device on `i2c2`
- the codec IRQ pin is configured through `it6620_irq`

Example DTS snippet:

```dts
earc_sound: earc-sound {
    status = "okay";
    compatible = "simple-audio-card";
    simple-audio-card,name = "HDMI-eARC";

    simple-audio-card,dai-link@0 {
        reg = <0>;
        format = "i2s";
        bitclock-master = <&link0_codec>;
        frame-master = <&link0_codec>;
        mclk-fs = <1024>;

        link0_cpu: cpu {
            sound-dai = <&i2s1_8ch>;
        };

        link0_codec: codec {
            sound-dai = <&earc_rx_codec>;
        };
    };
};

&i2s1_8ch {
    status = "okay";
    #sound-dai-cells = <0>;
    rockchip,mclk-fs = <1024>;
};

&i2c2 {
    status = "okay";

    earc_rx_codec: hdmi-earc@4e {
        compatible = "ite,it6620";
        reg = <0x4e>;
        #sound-dai-cells = <0>;
        ite,audio-addr = <0x63>;
        ite,cec-addr = <0x65>;
        interrupts = <RK_PB6 IRQ_TYPE_LEVEL_LOW>;
        reset-gpios = <&gpio4 RK_PA0 GPIO_ACTIVE_LOW>;
        status = "okay";
    };
};
```

## 3. Driver Implementation

HDMI eARC on this BSP is not only a DTS configuration. It also depends on the following ALSA SoC drivers in the kernel source:

- `sound/soc/generic/simple-card.c`
  - provides the `simple-audio-card` machine driver used by `earc_sound`
- `sound/soc/codecs/it6620.c`
  - implements the `ITE IT6620 eARC CODEC` driver
- `sound/soc/codecs/Kconfig`
  - contains `config SND_SOC_IT6620`
- `sound/soc/codecs/Makefile`
  - builds `snd-soc-it6620.o` from `it6620.o`

Relevant kernel configuration:

```text
config SND_SOC_IT6620
    tristate "ITE IT6620 eARC CODEC"
    depends on I2C
```

Makefile mapping:

```make
snd-soc-it6620-objs := it6620.o
obj-$(CONFIG_SND_SOC_IT6620) += snd-soc-it6620.o
```

Driver source header:

```c
// it6620.c -- IT6620 HDMI eARC receiver codec driver
//
// IT6620 is an HDMI receiver chip with eARC support.
// It uses I2C for control and has multiple I2C slave addresses
// that need to be activated via register writes.
```

In the current RK3566 BSP source tree, the option is already enabled:

```text
CONFIG_SND_SOC_IT6620=y
```

This means the DTS is expected to bind against the kernel-side `it6620` codec driver directly. If the audio device still does not appear in the OS, the problem is more likely to be probe failure, I2C communication, clocking, interrupt wiring, reset sequencing, or TV-side configuration rather than a missing DTS node.

## 4. Verify the Capture Device

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

## 5. Record Audio from HDMI eARC

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

## 6. TV Side Configuration Notes

The TV must be configured correctly, otherwise the board may not capture valid audio.

Required settings:

- Enable `HDMI-eARC`
- Enable `HDMI-CEC`
- Set digital audio output format to `PCM`

Samsung TV example paths:

- `Settings -> External Device Manager` for `HDMI-CEC`
- `Settings -> Sound -> Expert Settings -> HDMI-eARC Mode`
- `Settings -> Sound -> Sound Output -> Receiver (HDMI-eARC)`

## 7. Troubleshooting

If no HDMI eARC audio device appears in the OS, check the following items first:

1. Confirm the board DTS really contains the `earc_sound`, `i2s1_8ch`, and `earc_rx_codec` nodes and that they are all `status = "okay"`.
2. Confirm the `ite,it6620` driver is enabled in the running kernel.
3. Check whether the `it6620` device is detected on `i2c2`.
4. Check boot logs for `it6620`, `earc`, `simple-audio-card`, `asoc`, and `i2s1` related probe errors.
5. Confirm the TV side has `HDMI-eARC` and `HDMI-CEC` enabled and the digital output format is set to `PCM`.

Useful commands:

```bash
arecord -l
dmesg | grep -Ei 'it6620|earc|simple-audio-card|asoc|i2s1'
i2cdetect -y 2
```

If the DTS is correct but the device is still missing, the issue is usually in driver enablement, codec probe failure, I2C communication, clocking, interrupt wiring, or TV-side configuration.

## 8. Notes

- The actual ALSA card index may vary depending on the full audio configuration of the system. Always confirm with `arecord -l` before recording.
- If no valid audio is captured, first check the HDMI cable, TV eARC/CEC settings, and whether the TV output format is PCM.
