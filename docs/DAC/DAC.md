# DAC (Digital-to-Analog Converter) Interface Guide

## 1. Schematic

![DAC](./pic/dac.png)

## 2. Device Tree (DTS) Configuration

```dts
/{
    es9033_sound: es9033-sound {
        status = "okay";
        compatible = "simple-audio-card";
        simple-audio-card,name = "ES9033";
        simple-audio-card,format = "i2s";
        simple-audio-card,mclk-fs = <1024>;

        simple-audio-card,cpu {
            sound-dai = <&i2s1_8ch>;
        };

        simple-audio-card,codec {
            sound-dai = <&es9033_codec>;
        };
    };
};

/* DAC ES9033Q */
&spi1 {
    status = "okay";
    pinctrl-names = "default";
    pinctrl-0 = <&spi1m1_pins &spi1m1_cs0>;
    num-cs = <1>;

    es9033_codec: es9033-codec@0 {
        status = "okay";
        compatible = "ess,es9033";
        reg = <0>;
        #sound-dai-cells = <0>;
        clocks = <&cru I2S1_MCLKOUT>;
        clock-names = "mclk";

        assigned-clocks = <&cru I2S1_MCLKOUT>, <&cru I2S1_MCLK_TX_IOE>;
        //assigned-clocks = <&cru I2S1_MCLKOUT>, <&cru I2S1_MCLK_TX_IOE>, <&cru I2S1_MCLK_RX_IOE>;
        assigned-clock-rates = <49152000>;
        assigned-clock-parents = <&cru I2S1_MCLKOUT_TX>, <&cru I2S1_MCLKOUT_TX>;

        pinctrl-names = "default";
        pinctrl-0 = <&i2s1m1_mclk>;
    };
};


&i2s1_8ch {
    status = "okay";
    rockchip,trcm-sync-tx-only;
    i2s-lrck-gpio = <&gpio3 RK_PD0 GPIO_ACTIVE_HIGH>;
    pinctrl-names = "default";
    pinctrl-0 = <&i2s1m1_sclktx
                 &i2s1m1_lrcktx
                 &i2s1m1_sdo0>;

    //pinctrl-names = "default";
    //pinctrl-0 = <&i2s1m1_sclktx &i2s1m1_sclkrx
    //             &i2s1m1_lrcktx &i2s1m1_lrckrx
    //             &i2s1m1_sdi0   &i2s1m1_sdi1
    //             &i2s1m1_sdi2   &i2s1m1_sdi3
    //             &i2s1m1_sdo0>;
};
```

## 3. Testing

Connect an external amplifier to the J8 interface. Use the following command to play audio.The test audio file `test.wav` is located in the `resource` directory:

```bash
aplay -D hw:1,0 test.wav
```