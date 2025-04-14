# OpenBook - E-Reader Project  
**Developed by Emanuel-Stefan Stiuj - TSC 2025**

The OpenBook is an innovative, open-source e-reader designed for low power consumption, expandability, and ease of use. Built around the ESP32-C6-WROOM-1-N8 microcontroller, this project integrates an e-paper display, environmental sensing, real-time clock functionality, and robust storage options into a portable, battery-powered device. This README provides a detailed breakdown of the hardware design, functionality, and implementation, ensuring all critical aspects are covered.

---

![board](https://github.com/user-attachments/assets/a47ef2bb-df79-4e96-be9a-821cbd2bd45c)

## Project Overview

OpenBook is a fully functional e-reader that leverages modern hardware to deliver a seamless reading experience. Key features include:
- A 7.5-inch e-paper display for crisp, low-power text rendering.
- Wi-Fi 6 and Bluetooth 5 connectivity via the ESP32-C6 for wireless content updates.
- SD card and external flash storage for e-books and logs.
- Environmental monitoring with the BME688 sensor (temperature, humidity, pressure, air quality).
- Precise timekeeping with the DS3231 RTC module.
- USB-C for power, charging, and data transfer, with a Li-Po battery for portability.
- Expandability through a Qwiic I²C connector.

The design prioritizes energy efficiency, modularity, and user interaction via three tactile buttons, making it ideal for avid readers and hardware enthusiasts alike.

---

## Hardware Architecture

The OpenBook hardware architecture integrates several key components powered through a USB-C input and a Li-Po battery. The system uses a battery charger (MCP73831) to manage the 2500mAh Li-Po battery, which supplies power to an LDO regulator outputting 3.3V. This voltage powers the ESP32-C6-WROOM-1-N8 microcontroller, a 7.5-inch e-paper display, an SD card module, a 64MB NOR Flash, a DS3231 RTC, a BME688 sensor, and a MAX17048 fuel gauge. The ESP32 connects to the display, SD card, and flash via SPI, to the RTC, sensor, and fuel gauge via I²C, and to three tactile buttons via GPIO. The USB-C also provides data connectivity, and a Qwiic connector enables I²C expansion.

---

## Hardware Components and Functionality

### 1. Main Microcontroller: ESP32-C6-WROOM-1-N8
- **Processor**: 32-bit RISC-V, up to 160 MHz.
- **Memory**: 512 KB SRAM + 64 MB external NOR Flash (W25Q512JVEIQ).
- **Connectivity**: Wi-Fi 6 (2.4 GHz), Bluetooth 5 LE, USB 2.0.
- **Interfaces**: SPI, I²C, UART, GPIO.
- **Power**: 3.3V via LDO; supports deep sleep for minimal power use (<10 µA).
- **Why Chosen?**: Offers robust wireless capabilities, ample GPIO, and low-power modes, perfect for an e-reader with connectivity needs.

### 2. E-Paper Display
- **Type**: 7.5-inch Waveshare e-ink display (800x480 resolution).
- **Interface**: 4-wire SPI (MOSI, SCK, CS, DC) + control signals (RST, BUSY).
- **Pins**:
  - `EPD_CS` → `IO10`
  - `EPD_DC` → `IO5`
  - `EPD_RST` → `IO23`
  - `EPD_BUSY` → `IO3`
  - Shared SPI: `MOSI` (`IO7`), `SCK` (`IO6`).
- **Power**: 3.3V, ~20-50 mA during refresh, near-zero in static mode.
- **Why Chosen?**: E-paper’s ultra-low static power consumption makes it ideal for prolonged reading without draining the battery.

### 3. Storage
- **SD Card Module**:
  - **Interface**: SPI.
  - **Pins**: `SS_SD` → `IO4`, `MOSI` → `IO7`, `MISO` → `IO2`, `SCK` → `IO6`.
  - **Purpose**: External storage for e-books and firmware updates.
- **External Flash (W25Q512JVEIQ)**:
  - **Interface**: SPI (Quad I/O).
  - **Pins**: `FLASH_CS` → `IO11`, shared `MOSI`, `MISO`, `SCK`.
  - **Capacity**: 64 MB for firmware and content.
- **Power**: Both operate at 3.3V, with ~10-100 mA during active use.

### 4. Real-Time Clock (RTC): DS3231SN
- **Interface**: I²C.
- **Pins**: `SDA` → `IO21`, `SCL` → `IO22`, `INT_RTC` → `IO8`, `32KHZ` → `IO1`, `RTC_RST` → `IO18`.
- **Power**: 3.3V, <1 µA with backup (super-capacitor or coin cell).
- **Why Chosen?**: Provides accurate timekeeping, critical for timestamped logs or scheduled tasks, even when powered off.

### 5. Environmental Sensor: BME688
- **Measurements**: Temperature, humidity, pressure, VOC/eCO2.
- **Interface**: I²C (shared bus with RTC and Fuel Gauge).
- **Pins**: `SDA` → `IO21`, `SCL` → `IO22`.
- **Power**: 3.3V, ~0.8-3.6 mA active, <1 µA in sleep.
- **Why Chosen?**: Enhances user experience by monitoring reading conditions.

### 6. Power Management
- **Battery**: 2500 mAh Li-Po (3.7V nominal).
- **Charger**: MCP73831, charges via USB-C at up to 1A.
- **Fuel Gauge**: MAX17048, I²C (`SDA`/`SCL`), monitors battery level (~50 µA).
- **LDO Regulator**: XC6220A331MR-G, steps 5V down to 3.3V for all components.
- **USB-C**: Power input (5V) and data (USB_D+ → `IO13`, USB_D- → `IO12`).
- **Protection**: Reverse polarity diodes, ESD protection (USBLC6-2SC6Y).

---

## Communication Interfaces

| Interface | Components             | ESP32-C6 Pins                   | Notes                       |
|-----------|------------------------|---------------------------------|-----------------------------|
| SPI       | E-Paper, SD Card, Flash| `MOSI` (`IO7`), `MISO` (`IO2`), `SCK` (`IO6`) | Shared bus with individual CS pins. |
| I²C       | RTC, BME688, Fuel Gauge| `SDA` (`IO21`), `SCL` (`IO22`) | 400 kHz bus, pull-ups included. |
| UART      | Debugging              | `TXD0`, `RXD0` (test pads)     | Optional, for development.  |
| GPIO      | Buttons, Status        | `IO0`-`IO23`                   | Flexible assignments.       |
| USB       | Power/Data             | `USB_D+` (`IO13`), `USB_D-` (`IO12`) | USB 2.0 full-speed.   |

---

## Power Consumption

| Component           | Typical Current (mA) | Notes                              |
|---------------------|----------------------|------------------------------------|
| ESP32-C6 (Idle)     | ~10                  | <10 µA in deep sleep.             |
| ESP32-C6 (Wi-Fi)    | 80-160               | Peak during wireless activity.    |
| E-Paper (Refresh)   | 20-50                | Near-zero when static.            |
| SD Card             | 30-100               | Active during read/write.         |
| Flash (NOR)         | 10-20                | Intermittent use.                 |
| BME688              | ~0.8-3.6             | <1 µA in sleep.                   |
| DS3231 RTC          | ~0.2-3.5             | <1 µA with backup.                |
| MAX17048            | ~0.05                | Continuous monitoring.            |
| **Total (Typical)** | **150-250 mA**       | Varies with usage; ~50 µA in sleep. |

**Battery Life**: With a 2500 mAh battery, OpenBook can last weeks in typical use (mostly static display) or hours during heavy Wi-Fi and refresh activity.

---

| **Component Name**                     | **Component Type**        | **Link** |
|----------------------------------------|--------------------------------------|----------|
| MicroSD_112ATA                         | microSD                              | [Attend SnapEDA](https://www.snapeda.com/parts/112A-TAAR-R03/Attend/view-part/) |
| LED_ADAFRUIT0603                       | LED                                  | [Kingbright SnapEDA](https://www.snapeda.com/parts/KP-1608SURCK/Kingbright/view-part/?ref=search&t=LED%200603) |
| CAP_CPH3225                            | C                                    | [Seiko SnapEDA](https://www.snapeda.com/parts/CPH3225A/Seiko/view-part/) |
| VREG_XC6220A                           | Voltage Regulator                    | [Torex Mouser](https://ro.mouser.com/ProductDetail/Torex-Semiconductor/XC6220A331MR-G?qs=AsjdqWjXhJ8ZSWznL1J0gg%3D%3D&utm_source=octopart&utm_medium=aggregator&utm_campaign=865-XC6220A331MR-G&utm_content=Torex%20Semiconductor) |
| FUEL_MAX17048                          | Cell Fuel Gauge with ModelGauge      | [Analog Devices SnapEDA](https://www.snapeda.com/parts/MAX17048G+T10/Analog%20Devices/view-part/) |
| ESP32C6_MODULE                         | ESP32                                | [Espressif SnapEDA](https://www.snapeda.com/parts/ESP32-C6-WROOM-1-N8/Espressif%20Systems/view-part/?ref=search&t=ESP32-C6-WROOM-1-N8) |
| DIODE_MBR0530                          | Diode Schottky                       | [Onsemi SnapEDA](https://www.snapeda.com/parts/MBR0530/Onsemi/view-part/) |
| BME680_SENSOR                          | Env Sensor                           | [Bosch SnapEDA](https://www.snapeda.com/parts/BME680/Bosch%20Sensortec/view-part/?ref=search&t=bme680) |
| INDUCTOR_744043                        | L                                    | [Wurth Mouser](https://eu.mouser.com/ProductDetail/Wurth-Elektronik/744043680?qs=PGXP4M47uW6VkZq%252BkzjrHA%3D%3D) |
| CONNECTOR_QWIIC                        | PRT-14417 QWIIC_CONNECTOR            | [SparkFun SnapEDA](https://www.snapeda.com/parts/PRT-14417/SparkFun/view-part/) |
| VARSISTOR_ESP                          | Varsistor (B72520T0350K062)          | [TDK Mouser](https://ro.mouser.com/ProductDetail/EPCOS-TDK/B72520T0350K062?qs=dEfas%2FXlABIszF52uu7vrg%3D%3D) |
| RES_RCL0402                            | R                                    | [YAGEO ComponentSearch](https://componentsearchengine.com/part-view/R0402%201%25%20100%20K%20(RC0402FR-07100KL)/YAGEO) |
| TVS_PGB1010603                         | Ipp Tvs Diode Surface Mount 0603     | [Littelfuse SnapEDA](https://www.snapeda.com/parts/PGB1010603MR/Littelfuse/view-part/) |
| MOSFET_SI1308                          | MOSFET Transistor                    | [Vishay SnapEDA](https://www.snapeda.com/parts/SI1308EDL-T1-GE3/Vishay/view-part/) |
| BUTTON_EVQP7                           | Button                               | [SnapEDA Button](https://www.snapeda.com/search/?q=EVQP7L01P&search-type=parts) |
| USBLC6_TVS                             | Ipp Tvs Diode Surface Mount          | [STMicro SnapEDA](https://www.snapeda.com/parts/USBLC6-2SC6Y/STMicroelectronics/view-part/?ref=dk&t=USBLC6-2SC6Y&con_ref=None) |
| FLASH_W25Q512                          | FLASH - NOR Memory                   | [Winbond SnapEDA](https://www.snapeda.com/parts/W25Q512JVEIQ/Winbond%20Electronics/view-part/?ref=search&t=W25Q512JVEIQ) |
| CAP_ESP32C0402                         | C                                    | [YAGEO ComponentSearch](https://componentsearchengine.com/part-view/CC0402MRX5R5BB106/YAGEO) |
| RTC_DS3231                             | I²C-Integrated RTC/TCXO/Crystal      | [Analog Devices SnapEDA](https://www.snapeda.com/parts/DS3231SN%23/Analog%20Devices/view-part/?ref=search&t=DS3231SN%23) |
| VDET_BD5229                            | Voltage Detector                     | [Rohm SnapEDA](https://www.snapeda.com/parts/BD5229G-TR/Rohm/view-part/?ref=search&t=BD5229G-TR) |
| SCHOTTKY_SD0805                        | Diode Schottky                       | [AVX ComponentSearch](https://componentsearchengine.com/part-view/SD0805S020S1R0/Kyocera%20AVX) |
| MOSFET_DMG2305                         | T DMG2305UX-7                        | [Diodes ComponentSearch](https://componentsearchengine.com/part-view/DMG2305UX-7/Diodes%20Incorporated) |
| CHARGE_MCP73831                        | Tiny Integrated Li-Ion/Li-Poly Charge Mgnt Controller | [Microchip ComponentSearch](https://componentsearchengine.com/part-view/MCP73831T-2ACI%2FOT/Microchip) |
| CONNECTOR_FH34SRJ                      | FH34SRJ-24S-0.5SH(99)               | [Hirose ComponentSearch](https://componentsearchengine.com/part-view/FH34SRJ-24S-0.5SH(99)/Hirose) |
| USB4110_GF                             | USB4110-GF-A                         | [GCT SnapEDA](https://www.snapeda.com/parts/USB4110-GF-A./Global%20Connector%20Technology/view-part/) |
| CAP_RCLPOL                             | C pol                                | [GrabCAD Tantalum](https://grabcad.com/library/tantalum-smd-capacitor-type-b-3528-1) |
| JUMPER_SJ                              | Jumper-SolderPasteJumper3way         | [GrabCAD Jumper](https://grabcad.com/library/solder-jumpers-1) |

---

## ESP32-C6 Pin Assignments

| Pin    | Signal       | Function                          | Reason for Assignment                  |
|--------|--------------|-----------------------------------|----------------------------------------|
| EN     | RESET        | System reset                     | Standard reset pin for ESP32.          |
| IO0    | INT_RTC      | RTC interrupt                    | Low-power interrupt capability.        |
| IO1    | 32KHZ        | 32 kHz RTC clock output          | Drives accurate timekeeping.           |
| IO2    | MISO         | SPI data input (SD/Flash/Display)| Shared SPI bus input.                  |
| IO3    | EPD_BUSY     | E-Paper busy status              | Monitors display readiness.            |
| IO4    | SS_SD        | SD Card chip select              | Dedicated CS for SD card.              |
| IO5    | EPD_DC       | E-Paper data/command             | Distinguishes data vs. commands.       |
| IO6    | SCK          | SPI clock                        | Shared SPI clock signal.               |
| IO7    | MOSI         | SPI data output                  | Shared SPI output for peripherals.     |
| IO9    | IO/BOOT      | Boot button                      | Triggers boot mode or power-on.        |
| IO10   | EPD_CS       | E-Paper chip select              | Dedicated CS for display.              |
| IO11   | FLASH_CS     | Flash chip select                | Dedicated CS for NOR Flash.            |
| IO12   | USB_D-       | USB differential data -          | Fixed USB pin.                         |
| IO13   | USB_D+       | USB differential data +          | Fixed USB pin.                         |
| IO15   | IO/CHANGE    | Page change button               | User input for navigation.             |
| IO18   | RTC_RST      | RTC reset                        | Hardware reset for RTC.                |
| IO21   | SDA          | I²C data line                    | Shared I²C bus for sensors/RTC.        |
| IO22   | SCL          | I²C clock line                   | Shared I²C clock signal.               |
| IO23   | EPD_RST      | E-Paper reset                    | Hardware reset for display.            |
| TXD0   | TX           | UART transmit (debug)            | Test pad for debugging.                |
| RXD0   | RX           | UART receive (debug)             | Test pad for debugging.                |

---

## Design Decisions

1. **Power Routing**: Power traces were prioritized and routed manually to ensure stability, followed by autorouting for signal lines.
2. **USB-C Integration**: Modified to avoid DRC errors, supporting both power and data seamlessly.
3. **E-Paper Choice**: Selected for its low power and readability, with a dedicated 3.3V supply controlled via a MOSFET (`EPD_3v3_c`) for efficiency.
4. **Modularity**: Qwiic connector and test pads enhance expandability and debuggability.
5. **ESD Protection**: Added to SPI, USB, and power lines to ensure durability.

---

## Implementation Steps

1. **Schematic Design**: Created in Autodesk Fusion, integrating all components with clear pin mappings.
2. **PCB Layout**: 
   - Defined GND plane and power net classes.
   - Applied custom routing rules for signal integrity.
   - Adjusted enclosure dimensions to fit the PCB.
3. **3D Modeling**: Designed battery and display mounts based on datasheets.
4. **Testing**: Used test pads for UART debugging and verified functionality.



