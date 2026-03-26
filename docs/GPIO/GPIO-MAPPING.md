# GPIO Mapping Guide

## 1. Overview

This document provides the board-level GPIO mapping for Autonomic M1, including front LEDs and other control-related signals that are already confirmed from the schematic and DTS.

The mapping is described in four forms:

- board signal name
- SoC GPIO name
- DTS representation
- Linux runtime view

## 2. Confirmed GPIO Mapping

| Function | Board Signal | SoC GPIO | DTS | Linux Runtime |
| --- | --- | --- | --- | --- |
| Front LED 1 | `LED_HEARTBEAT1` | `GPIO0_B7` | `&gpio0 RK_PB7` | `gpio-15`, `sys_status_led` |
| Front LED 2 | `LED_HEARTBEAT2` | `GPIO0_A0` | `&gpio0 RK_PA0` | `gpio-0`, `power_status_led` |
| TSADC shutdown | `TSADC_SHUT_M0` | `GPIO0_A1` | `&gpio0 RK_PA1` | no dedicated consumer in current BSP |
| PMIC sleep request | `PMIC_SLEEP_H` | `GPIO0_A2` | `&gpio0 RK_PA2` | no dedicated consumer in current BSP |
| PMIC interrupt | `PMIC_INT_L` | `GPIO0_A3` | `&gpio0 RK_PA3` | no dedicated consumer in current BSP |
| SD card detect | `SDMMC0_DET_L` | `GPIO0_A4` | `&gpio0 RK_PA4` | no dedicated consumer in current BSP |
| Trigger out enable | `TRIGGER_EN` | `GPIO1_A5` | `&gpio1 RK_PA5` | no dedicated consumer in current BSP |
| Power control | `power_ctrl` | `GPIO1_A5` | `&gpio1 RK_PA5` | `gpio-37`, `power_ctrl` |
| USB3 host power enable | `USB30_HOST_PWREN` | `GPIO1_A6` | `&gpio1 RK_PA6` | `gpio-38`, `vcc5v0-usb30-host` |
| USB3 mux select | `USB30_MUX` | `GPIO1_B0` | `&gpio1 RK_PB0` | `gpio-40`, `mux` |
| Type-C over-current feedback | `TYPE_C_OCB` | schematic confirmed, GPIO mapping not finalized in current BSP | not exposed in current DTS | no dedicated consumer in current BSP |
| SD card power enable | `SDMMC0_PWR` | `GPIO0_A5` | `&gpio0 RK_PA5` | `gpio-5`, `vcc3v3-sd` |
| Analog 3.3V rail enable 1 | `vcca1_3v3` | `GPIO0_C3` | `&gpio0 RK_PC3` | `gpio-19`, `vcca1-3v3` |
| Analog 3.3V rail enable 2 | `vcca2_3v3` | `GPIO0_C4` | `&gpio0 RK_PC4` | `gpio-20`, `vcca2-3v3` |
| BT reset | `BT,reset_gpio` | `GPIO2_B7` | `&gpio2 RK_PB7` | `gpio-79`, `bt_default_reset` |
| BT wake | `BT_WAKE_HOST_H` | `GPIO2_C0` | `&gpio2 RK_PC0` | `gpio-80`, `bt_default_wake` |
| BT host wake IRQ | `HOST_WAKE_BT_H` | `GPIO2_C1` | `&gpio2 RK_PC1` | `gpio-81`, `bt_default_wake_host` |
| WIFI enable | `WIFI_REG_ON_H` | `GPIO2_B1` | `&gpio2 RK_PB1` | no dedicated consumer name in current runtime |
| WIFI host wake IRQ | `WIFI_WAKE_HOST_H` | `GPIO2_B2` | `&gpio2 RK_PB2` | runtime name depends on WLAN driver |
| UART1 RTS GPIO | `UART1_RTSn_M0` | `GPIO2_B5` | `&gpio2 RK_PB5` | runtime name depends on Bluetooth driver |
| SDMMC0 CMD | `SDMMC0_CMD` | `GPIO2_A1` | `&gpio2 RK_PA1` | per schematic |
| SDMMC0 CLK | `SDMMC0_CLK` | `GPIO2_A2` | `&gpio2 RK_A2` | per schematic |

## 3. Trigger Out

The trigger-out control signal is present in the schematic as:

- `TRIGGER_EN`

From the schematic, `TRIGGER_EN` is connected to:

- `GPIO1_A5`

The corresponding DTS form is:

```dts
&gpio1 RK_PA5
```

At the moment, the current BSP DTS does not expose a dedicated trigger-out function node for this signal, so it is recommended to add it to both:

- this GPIO mapping table
- DTS `gpio-line-names` or a dedicated trigger-out function node

## 4. Source References

Relevant DTS source:

- `kernel-6.1/arch/arm64/boot/dts/rockchip/rk3566-autonomic-m1.dts`

Relevant schematic:

- `AUTONOMIC_M1_V1_0_0.pdf`

Useful schematic signal names:

- `LED_HEARTBEAT1`
- `LED_HEARTBEAT2`
- `TRIGGER_EN`
- `TYPE_C_OCB`
- `USB30_HOST_PWREN`
- `SDMMC0_PWR`

## 5. Linux Runtime View

After boot, GPIO usage can be checked from Linux with:

```bash
cat /sys/kernel/debug/gpio

gpiochip0: GPIOs 0-31, parent: platform/fdd60000.gpio, gpio0:
 gpio-0   (GPIO0_A0 / LED_HEART|power_status_led    ) out lo 
 gpio-1   (GPIO0_A1            )
 gpio-2   (GPIO0_A2 / PMIC_SLEE)
 gpio-3   (GPIO0_A3 / PMIC_INT_)
 gpio-4   (GPIO0_A4 / SDMMC0_DE)
 gpio-5   (GPIO0_A5 / SDMMC0_PW|vcc3v3-sd           ) out lo ACTIVE LOW
 gpio-6   (GPIO0_A6 / E2PROM_WP)
 gpio-7   (GPIO0_A7 / FLASH_VOL)
 gpio-8   (GPIO0_B0            )
 gpio-9   (GPIO0_B1 / I2C0_SCL_)
 gpio-10  (GPIO0_B2 / I2C0_SDA_)
 gpio-11  (GPIO0_B3 / I2C1_SCL_)
 gpio-12  (GPIO0_B4 / I2C1_SDA_)
 gpio-13  (GPIO0_B5 / ADC_SPI_S)
 gpio-14  (GPIO0_B6 / ADC_SPI_M)
 gpio-15  (GPIO0_B7 / LED_HEART|sys_status_led      ) out lo 
 gpio-16  (GPIO0_C0 / UART0_RX_)
 gpio-17  (GPIO0_C1 / UART0_TX_)
 gpio-18  (GPIO0_C2 / RTC_INTN )
 gpio-19  (GPIO0_C3 / PWR_ADC_E|vcca1-3v3           ) out hi 
 gpio-20  (GPIO0_C4 / PWR_DAC_E|vcca2-3v3           ) out hi 
 gpio-21  (GPIO0_C5 / ADC_SPI_M)
 gpio-22  (GPIO0_C6 / ADC_SPI_S)
 gpio-23  (GPIO0_C7 / TYPE_C_IN)
 gpio-24  (GPIO0_D0 / UART2_RX_)
 gpio-25  (GPIO0_D1 / UART2_TX_)
 gpio-26  (GPIO0_D2            )
 gpio-27  (GPIO0_D3            )
 gpio-28  (GPIO0_D4            )
 gpio-29  (GPIO0_D5            )
 gpio-30  (GPIO0_D6            )
 gpio-31  (GPIO0_D7            )
```

## 6. Recommended Next Step

To make the mapping directly visible from the OS, add `gpio-line-names` to the `gpio0`, `gpio1`, `gpio2`, and other GPIO controller nodes in the board DTS. Then the signal names can be inspected more directly through:

```bash
cat /sys/kernel/debug/gpio
```

## 7. gpio-line-names Candidate

If the goal is to cover all GPIO pins shown in the schematic, the recommended style is:

- every line gets at least its SoC pin name
- append a board-level function name only when it is already confirmed
- each gpio controller must list all 32 positions in order without omission

Naming format:

- `GPIOX_YN`
- or `GPIOX_YN / FUNCTION_NAME`

Example candidate:

```dts
&gpio0 {
	gpio-line-names =
		"GPIO0_A0 / LED_HEARTBEAT2 / POWER_STATUS_LED",
		"GPIO0_A1",
		"GPIO0_A2 / PMIC_SLEEP_H",
		"GPIO0_A3 / PMIC_INT_L",
		"GPIO0_A4 / SDMMC0_DET_L",
		"GPIO0_A5 / SDMMC0_PWR",
		"GPIO0_A6 / E2PROM_WP",
		"GPIO0_A7 / FLASH_VOL_SEL",
		"GPIO0_B0",
		"GPIO0_B1 / I2C0_SCL_PMIC",
		"GPIO0_B2 / I2C0_SDA_PMIC",
		"GPIO0_B3 / I2C1_SCL_PMUIO",
		"GPIO0_B4 / I2C1_SDA_PMUIO",
		"GPIO0_B5 / ADC_SPI_SCLK",
		"GPIO0_B6 / ADC_SPI_MOSI",
		"GPIO0_B7 / LED_HEARTBEAT1 / SYS_STATUS_LED",
		"GPIO0_C0 / UART0_RX_M0",
		"GPIO0_C1 / UART0_TX_M0",
		"GPIO0_C2 / RTC_INTN",
		"GPIO0_C3 / PWR_ADC_EN / VCCA1_3V3_EN",
		"GPIO0_C4 / PWR_DAC_EN / VCCA2_3V3_EN",
		"GPIO0_C5 / ADC_SPI_MISO",
		"GPIO0_C6 / ADC_SPI_SS",
		"GPIO0_C7 / TYPE_C_INTN",
		"GPIO0_D0 / UART2_RX_M0_DEBUG",
		"GPIO0_D1 / UART2_TX_M0_DEBUG",
		"GPIO0_D2",
		"GPIO0_D3",
		"GPIO0_D4",
		"GPIO0_D5",
		"GPIO0_D6",
		"GPIO0_D7";
};

&gpio1 {
	gpio-line-names =
		"GPIO1_A0 / I2C3_SDA_M0",
		"GPIO1_A1 / I2C3_SCL_M0",
		"GPIO1_A2",
		"GPIO1_A3",
		"GPIO1_A4 / SPDIF_TX",
		"GPIO1_A5 / POWER_CTRL / TRIGGER_EN",
		"GPIO1_A6 / USB30_HOST_PWREN",
		"GPIO1_A7 / TYPE_C_OCB",
		"GPIO1_B0 / USB30_MUX",
		"GPIO1_B1",
		"GPIO1_B2",
		"GPIO1_B3",
		"GPIO1_B4 / eMMC_D0/FLASH_D0",
		"GPIO1_B5 / eMMC_D1/FLASH_D1",
		"GPIO1_B6 / eMMC_D2/FLASH_D2",
		"GPIO1_B7 / eMMC_D3/FLASH_D3",
		"GPIO1_C0 / eMMC_D4/FLASH_D4",
		"GPIO1_C1 / eMMC_D5/FLASH_D5",
		"GPIO1_C2 / eMMC_D6/FLASH_D6",
		"GPIO1_C3 / eMMC_D7/FLASH_D7",
		"GPIO1_C4 / eMMC_CMD/FLASH_WRn",
		"GPIO1_C5 / eMMC_CLKOUT/FLASH_DQS",
		"GPIO1_C6 / eMMC_DATA_STROBE/FLASH_CLE",
		"GPIO1_C7 / eMMC_RSTn/FSPI_D2/FLASH_WPn",
		"GPIO1_D0",
		"GPIO1_D1",
		"GPIO1_D2",
		"GPIO1_D3",
		"GPIO1_D4",
		"GPIO1_D5 / SDMMC0_D0",
		"GPIO1_D6 / SDMMC0_D1",
		"GPIO1_D7 / SDMMC0_D2";
};

&gpio2 {
	gpio-line-names =
		"GPIO2_A0 / SDMMC0_D3",
		"GPIO2_A1 / SDMMC0_CMD",
		"GPIO2_A2 / SDMMC0_CLK",
		"GPIO2_A3 / SDMMC1_D0",
		"GPIO2_A4 / SDMMC1_D1",
		"GPIO2_A5 / SDMMC1_D2",
		"GPIO2_A6 / SDMMC1_D3",
		"GPIO2_A7 / SDMMC1_CMD",
		"GPIO2_B0 / SDMMC1_CLK",
		"GPIO2_B1 / WIFI_REG_ON_H",
		"GPIO2_B2 / WIFI_WAKE_HOST_H",
		"GPIO2_B3 / UART1_RX_M0",
		"GPIO2_B4 / UART1_TX_M0",
		"GPIO2_B5 / UART1_RTSn_M0",
		"GPIO2_B6 / UART1_CTSn_M0",
		"GPIO2_B7 / BT_REG_ON_H",
		"GPIO2_C0 / BT_WAKE_HOST_H",
		"GPIO2_C1 / HOST_WAKE_BT_H",
		"GPIO2_C2 / I2S2_SCLK_TX_M0",
		"GPIO2_C3 / I2S2_LRCK_TX_M0",
		"GPIO2_C4 / I2S2_SDO_M0",
		"GPIO2_C5 / I2S2_SDI_M0",
		"GPIO2_C6",
		"GPIO2_C7",
		"GPIO2_D0",
		"GPIO2_D1",
		"GPIO2_D2",
		"GPIO2_D3",
		"GPIO2_D4",
		"GPIO2_D5",
		"GPIO2_D6",
		"GPIO2_D7";
};

&gpio3 {
	gpio-line-names =
		"GPIO3_A0",
		"GPIO3_A1 / DAC_SPI_SS",
		"GPIO3_A2 / GMAC1_TXD2",
		"GPIO3_A3 / GMAC1_TXD3",
		"GPIO3_A4 / GMAC1_RXD2",
		"GPIO3_A5 / GMAC1_RXD3",
		"GPIO3_A6 / GMAC1_TXCLK",
		"GPIO3_A7 / GMAC1_RXCLK",
		"GPIO3_B0 / ETH1_REFCLKO_25M",
		"GPIO3_B1 / GMAC1_RXD0",
		"GPIO3_B2 / GMAC1_RXD1",
		"GPIO3_B3 / GMAC1_RXDV_CRS",
		"GPIO3_B4",
		"GPIO3_B5 / GMAC1_TXD0",
		"GPIO3_B6 / GMAC1_TXD1",
		"GPIO3_B7 / GMAC1_TXEN",
		"GPIO3_C0 / GMAC1_MCLKINOUT",
		"GPIO3_C1 / DAC_SPI_MOSI",
		"GPIO3_C2 / DAC_SPI_MISO",
		"GPIO3_C3 / DAC_SPI_SCLK",
		"GPIO3_C4 / GMAC1_MDC",
		"GPIO3_C5 / GMAC1_MDIO",
		"GPIO3_C6 / DAC_MCLK",
		"GPIO3_C7 / DAC_BCK",
		"GPIO3_D0 / DAC_WS",
		"GPIO3_D1 / DAC_SDO",
		"GPIO3_D2 / EARC_I2S1_D0",
		"GPIO3_D3 / EARC_I2S1_D1",
		"GPIO3_D4 / EARC_I2S1_D2",
		"GPIO3_D5 / EARC_I2S1_D3",
		"GPIO3_D6 / EARC_I2S1_SCK_DCLK",
		"GPIO3_D7 / EARC_PDM_DL2";
};

&gpio4 {
	gpio-line-names =
		"GPIO4_A0 / EARC_RSTn",
		"GPIO4_A1 / EARC_PDM_DR2",
		"GPIO4_A2 / EARC_PDM_DL3",
		"GPIO4_A3 / EARC_PDM_DR3",
		"GPIO4_A4 / EARC_I2S1_MUTE",
		"GPIO4_A5 / GMAC1_INTB",
		"GPIO4_A6 / EARC_I2S1_SCK_DCLK",
		"GPIO4_A7 / EARC_I2S1_WS",
		"GPIO4_B0 / DAC_MODE",
		"GPIO4_B1 / DAC_EN",
		"GPIO4_B2 / I2C4_SDA_M0",
		"GPIO4_B3 / I2C4_SCL_M0",
		"GPIO4_B4 / EARC_I2C2_SDA",
		"GPIO4_B5 / EARC_I2C2_SCL",
		"GPIO4_B6 / EARC_INT",
		"GPIO4_B7 / ADC_MODE",
		"GPIO4_C0 / ADC_EN",
		"GPIO4_C1 / GMAC1_RESET",
		"GPIO4_C2 / ADC_CLK_IN",
		"GPIO4_C3 / ADC_I2S_BCK",
		"GPIO4_C4 / ADC_I2S_WS",
		"GPIO4_C5",
		"GPIO4_C6 / ADC_I2S_SDO",
		"GPIO4_C7 / HDMITX_SCL",
		"GPIO4_D0 / HDMITX_SDA",
		"GPIO4_D1 / HDMITX_CEC_M0",
		"GPIO4_D2",
		"GPIO4_D3",
		"GPIO4_D4",
		"GPIO4_D5",
		"GPIO4_D6",
		"GPIO4_D7";
};
```
