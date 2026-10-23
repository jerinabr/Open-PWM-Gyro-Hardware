# Description
6-channel PWM gyro for fixed wing RC planes. Meant to be connected to an ELRS RX or other RX that outputs CRSF serial data.

## Layer Stackup
This is a 4-layer board with the following stackup:
1. Signal + Power
2. Ground
3. Ground
4. Signal + Power

Both ground layers are solid ground pours and aren't shown in the images section below

## Features
- STM32C551 Arm Cortex-M33 MCU (144 MHz, FPU) in a 5 x 5 mm UFQFPN32 package
- Crystal-less clocking: internal 144 MHz oscillator (±1 %), USB clock trimmed from USB SOF
- USB-C for device firmware upgrade and configuration
- SWD header for device programming and debugging
- 2-color status LED (red, blue) controlled by the MCU
- Boot switch to allow the device to enter the bootloader on power-up
- 6-axis IMU (3-axis accelerometer, 3-axis gyroscope)
- Pin header for servo/ESC connections with one 3-pin group for serial data from the RX (CRSF, IBUS, SBUS); the serial pin supports single-wire half-duplex for telemetry
- Servo rail (V_ESC) voltage sensing through the MCU ADC

## Firmware
Firmware for this gyro can be found [in this repo](https://github.com/jerinabr/Open-PWM-Gyro)

# Revisions
| Revision | MCU            | Package           | Clock                     | Status                                   |
| -------- | -------------- | ----------------- | ------------------------- | ---------------------------------------- |
| v1.0     | STM32G431KBT6  | LQFP32 7 x 7 mm   | 16 MHz crystal + PLL      | Built, gerbers in `Gerbers/` (tag `v1.0`) |
| v2.0     | STM32C551KCU6  | UFQFPN32 5 x 5 mm | Internal HSI 144 MHz      | Schematic done, PCB layout pending       |

> [!IMPORTANT]
> The schematic and BOM are Revision 2. The PCB, gerbers, iBOM and images are still Revision 1 until the board is re-laid out.

## Revision 2 changes
- U2 is now the STM32C551KCU6 (Cortex-M33 + FPU, 144 MHz, 256 KB flash, 128 KB SRAM). The symbol is embedded in the schematic as `MCU_ST_STM32C5_Custom:STM32C551KCUx` (KiCad has no STM32C5 symbols yet). The STM32C551KEU6 (512 KB) is a drop-in alternative.
- The 16 MHz crystal (Y1) and its load caps (C10, C11) are removed. The HSI is specified at ±1 % over T<sub>J</sub> -20 to 130 °C (±1.5 % down to -40 °C), which is well inside UART tolerance. USB needs ±0.25 %, so the firmware lets the clock recovery system (CRS) trim the HSI from the USB start-of-frame packets.
- C13 (2.2 µF) on VCAP for the internal LDO. It must have an ESR below 20 mΩ at 3 MHz and be rated for at least 10 V.
- C14 (4.7 µF) bulk decoupling on +3.3V next to the MCU, as the datasheet power scheme asks for 4.7 µF + 100 nF per VDD pin. C5/C6/C7 now decouple VDD (pin 1), VDD (pin 17) and VREF+ (pin 5).
- R7/R8/C15 divide V_ESC by 5.7 into PB1 (ADC2_IN7), so the gyro can measure the servo rail (up to 18.8 V full scale).
- The RX serial pin moved to PB6 (USART1_TX). Receive-only protocols use the USART TX/RX swap so the receiver sits on this pin, and single-wire half-duplex mode makes CRSF telemetry possible without another wire. The net was renamed from `CRSF_RX_To_MCU` to `RX_SERIAL`.
- The status LEDs moved to PA8/PA9, which are TIM1_CH1/CH2, so they can be dimmed or blinked in hardware.
- LED current fix: in Rev 1 the blue LED had no series resistor and R4 (68 Ω) set the red LED to about 20 mA, so both GPIO pins sank close to their 20 mA absolute maximum. R6 (100 Ω) now sits in series with the blue anode and R4 is 220 Ω, which gives about 4 mA (blue) and 6 mA (red), and stays under the 8 mA I/O rating at worst-case forward voltage.
- BOOT0 is now the dedicated PH2-BOOT0 pin. The boot switch circuit is unchanged. The pin is only used when the `BOOT_SEL` option bit (FLASH_OPTSR) selects the BOOT0 pin; otherwise the BOOT0 option bit decides. Check this option byte the first time the board is programmed over SWD.
- The UFQFPN32 exposed pad is the only VSS connection and must be soldered to GND.

## Revision 2 pinout
| Pin | Port      | Function            | Net             |
| --- | --------- | ------------------- | --------------- |
| 1   | VDD       | Supply              | +3.3V           |
| 2   | PH0/PC14  | Spare               | NC              |
| 3   | PH1       | Spare               | NC              |
| 4   | NRST      | Reset               | N_RESET         |
| 5   | VREF+     | ADC reference       | +3.3V           |
| 6   | PA0       | TIM2_CH1 (AF1)      | CH3_PWM         |
| 7   | PA1       | TIM2_CH2 (AF1)      | CH4_PWM         |
| 8   | PA2       | TIM2_CH3 (AF1)      | CH5_PWM         |
| 9   | PA3       | TIM2_CH4 (AF1)      | CH6_PWM         |
| 10  | PA4       | IMU chip select     | IMU CS          |
| 11  | PA5       | SPI1_SCK (AF5)      | IMU SCLK        |
| 12  | PA6       | SPI1_MISO (AF5)     | IMU SDO         |
| 13  | PA7       | SPI1_MOSI (AF5)     | IMU SDI         |
| 14  | PB0       | EXTI0               | IMU_INT         |
| 15  | PB1       | ADC2_IN7            | V_ESC_SENSE     |
| 16  | VCAP      | LDO output          | VCAP (C13)      |
| 17  | VDD       | Supply              | +3.3V           |
| 18  | PB15      | Spare               | NC              |
| 19  | PA8       | GPIO / TIM1_CH1     | STATUS_LED_RED  |
| 20  | PA9       | GPIO / TIM1_CH2     | STATUS_LED_BLUE |
| 21  | PA11      | USB_DM              | USB_D-          |
| 22  | PA12      | USB_DP              | USB_D+          |
| 23  | PA13      | SWDIO               | SWD_DIO         |
| 24  | PA14      | SWCLK               | SWD_CLK         |
| 25  | PA15      | Spare               | NC              |
| 26  | PB3       | TRACESWO            | SWD_SWO         |
| 27  | PB4       | TIM5_CH1 (AF2)      | CH2_PWM         |
| 28  | PB5       | TIM5_CH2 (AF2)      | CH1_PWM         |
| 29  | PB6       | USART1_TX (AF7)     | RX_SERIAL       |
| 30  | PB7       | Spare (I2C1_SDA)    | NC              |
| 31  | PH2-BOOT0 | BOOT0               | BOOT0           |
| 32  | PB8       | Spare (I2C1_SCL)    | NC              |
| EP  | VSS       | Ground              | GND             |

All six PWM outputs are on 32-bit timers (TIM2 and TIM5). SER, CH1 and CH2 are next to each other on pins 29 to 27, and CH3 to CH6 are on pins 6 to 9, both in header order. PB7 and PB8 are left free as a spare I2C1 bus.

## Revision 2 layout notes
- Footprint: `Package_DFN_QFN:QFN-32-1EP_5x5mm_P0.5mm_EP3.6x3.6mm`. Stitch the exposed pad to both inner ground planes with a via array. The pad is also the MCU's only ground return.
- Put C13 (VCAP) closest to pin 16 with a short return to the exposed pad, then the 100 nF caps on pins 1, 17 and 5.
- The crystal and its keep-out are gone, which frees the area around pins 2 and 3.
- Route USB_D+/USB_D- (pins 21/22) as a 90 Ω differential pair to J1.
- Run "Update PCB from Schematic" in KiCad first. U2 keeps its UUID, so it is swapped in place. Y1, C10 and C11 are deleted, and C13, C14, C15, R6, R7 and R8 are added.
- Part numbers marked TBD in the BOM still need DigiKey numbers. The schematic has manufacturer part numbers for them.

# BOM
| Reference                 | Qty | Value                       | Footprint                                                             | DigiKey Part #          |
| ------------------------- | --- | --------------------------- | --------------------------------------------------------------------- | ----------------------- |
| C1,C5,C6,C7,C8,C9,C12,C15 | 8   | 0.1u                        | Capacitor_SMD:C_0402_1005Metric_Pad0.74x0.62mm_HandSolder             | 1276-6720-1-ND          |
| C2,C4                     | 2   | 1u                          | Capacitor_SMD:C_0402_1005Metric_Pad0.74x0.62mm_HandSolder             | 1276-1445-1-ND          |
| C3                        | 1   | 22u                         | Capacitor_SMD:CP_Elec_4x5.4                                           | 732-8413-1-ND           |
| C13                       | 1   | 2.2u                        | Capacitor_SMD:C_0402_1005Metric_Pad0.74x0.62mm_HandSolder             | TBD                     |
| C14                       | 1   | 4.7u                        | Capacitor_SMD:C_0603_1608Metric_Pad1.08x0.95mm_HandSolder             | TBD                     |
| D1                        | 1   | PMEG2005CT,215              | Package_TO_SOT_SMD:SOT-23                                             | 1727-5188-2-ND          |
| J1                        | 1   | USB_C_Receptacle_USB2.0_16P | Connector_USB:USB_C_Receptacle_GCT_USB4105-xx-A_16P_TopMnt_Horizontal | 2073-USB4105-GF-ATR-ND  |
| J2                        | 1   | Conn_01x21                  | Connector_PinHeader_2.54mm_Custom:PinHeader_3x07_P2.54mm_Vertical     | TSW-107-26-F-T-ND       |
| LED1                      | 1   | LED_Dual_AKKA               | LED_SMD_Custom:LED_APHBM2012QBDSURKC                                  | 754-1506-1-ND           |
| MT1                       | 1   | ICM-42605                   | Package_LGA_Custom:XDCR_ICM-42605                                     | 1428-1152-1-ND          |
| R1,R2                     | 2   | 5k1                         | Resistor_SMD:R_0402_1005Metric_Pad0.72x0.64mm_HandSolder              | 13-RC0402FR-135K1LCT-ND |
| R3,R5                     | 2   | 10k                         | Resistor_SMD:R_0402_1005Metric_Pad0.72x0.64mm_HandSolder              | 311-10KJRCT-ND          |
| R4                        | 1   | 220                         | Resistor_SMD:R_0402_1005Metric_Pad0.72x0.64mm_HandSolder              | TBD                     |
| R6                        | 1   | 100                         | Resistor_SMD:R_0402_1005Metric_Pad0.72x0.64mm_HandSolder              | TBD                     |
| R7                        | 1   | 47k 1%                      | Resistor_SMD:R_0402_1005Metric_Pad0.72x0.64mm_HandSolder              | TBD                     |
| R8                        | 1   | 10k 1%                      | Resistor_SMD:R_0402_1005Metric_Pad0.72x0.64mm_HandSolder              | TBD                     |
| SW1                       | 1   | SW_SPST                     | Button_Switch_SMD_Custom:R-667995_MIT                                 | 2508-R-667995TR-ND      |
| SWD1                      | 1   | Conn_02x05_Odd_Even         | Connector_PinHeader_1.27mm:PinHeader_2x05_P1.27mm_Vertical            | S9015E-05-ND            |
| U1                        | 1   | TCR2EF33_LMCT               | Package_TO_SOT_SMD_Custom:SOT-25 SMV_TOS                              | TCR2EF33LM(CTTR-ND      |
| U2                        | 1   | STM32C551KCU6               | Package_DFN_QFN:QFN-32-1EP_5x5mm_P0.5mm_EP3.6x3.6mm                   | TBD                     |

> [!NOTE]
> J2 is a 3x7 2.54mm pitch pin header. This part can be replaced with three 7-pin headers or seven 3-pin headers or some other combination to save cost

# Images
> [!NOTE]
> These images show Revision 1.

## Schematic
### Page 1: Power + Connectors
![Schematic Page 1: Power + Connectors](Images/Schematic_Pg_1.svg)
### Page 2: MCU + IMU
![Schematic Page 2: MCU + IMU](Images/Schematic_Pg_2.svg)
## PCB
### Top Layer
![PCB Top Layer](Images/PCB_Top.png)
### Bottom Layer
![PCB Bottom Layer](Images/PCB_Bottom.png)
### Top + Bottom Layer
![PCB Top + Bottom Layer](Images/PCB_Top_Bottom.png)
## 3D View
### Top
![3D Top View](Images/3D_Top.png)
### Bottom
![3D Bottom View](Images/3D_Bottom.png)
