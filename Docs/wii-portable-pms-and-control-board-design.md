# Wii Portable Power Management Board

> A custom all-in-one power and management board for a handheld Nintendo Wii, featuring 2S Li-ion charging, synchronous step-down regulators, an MCU-based state machine, OLED telemetry, and isolated audio.

## Design Overview

| Item | Details |
|------|---------|
| Product Type | Wii Portable Power Management Board |
| Key Functions | 7.4V 2S Li-ion battery charging and BMS protection with fuel gauge, Synchronous step-down power rails (1.0V, 1.15V, 1.8V, 3.3V, 5.0V), RP2040/ESP32 MCU for power state machine and telemetry, Monochrome OLED display and power button pre-boot menu, Class-D audio amplifier and I2S DAC with isolated power planes |
| Power Source | 7.4V 2S Li-Ion Battery / USB-C |
| Connectivity | USB-C |
| Size | Under 80mm x 50mm stackable sandwich layout |
| Environment | handheld gaming device |
| Difficulty | Advanced |
| Est. Time | 6-8 hours |

## System Architecture

**Power Topology:** USB-C input feeds BQ25606 charger and XB7608A BMS protection for 2S Li-ion battery pack. VBAT (7.4V nominal) distributes to synchronous step-down regulators generating 1.0V, 1.15V, 1.8V, 3.3V, and 5.0V rails. An isolated analog ground plane with ferrite beads powers the PCM5102A DAC and TPA3110D2 amplifier.

### Module List

| Module ID      | Type    | Name                   | Function                                                                         | Voltage |
| -------------- | ------- | ---------------------- | -------------------------------------------------------------------------------- | ------- |
| USB_C_CHARGER  | POWER   | BQ25606                | High-efficiency 2S synchronous switch-mode charger with USB-C input              | 9V      |
| BMS_PROTECTION | POWER   | XB7608A                | Dual-cell Li-ion protection IC for overcharge, over-discharge, and short-circuit | 7.4V    |
| FUEL_GAUGE     | SENSOR  | MAX17044               | Precision fuel gauge IC tracking State-of-Charge (SoC) via ModelGauge algorithm  | 7.4V    |
| BUCK_1V0_1V15  | POWER   | TPS62840 / MP2143      | Generates tight-tolerance 1.0V and 1.15V core rails for Hollywood/Broadway       | 7.4V    |
| BUCK_1V8_3V3   | POWER   | TPS62130               | Generates 1.8V and 3.3V auxiliary and peripheral power rails                     | 7.4V    |
| BUCK_5V0       | POWER   | TPS62130A              | Generates clean 5.0V system rail for display backlight, USB, and audio           | 7.4V    |
| RP2040_MCU     | MCU     | RP2040 + W25Q16JV      | Power state machine, sequencing, OLED UI handler, and telemetry streaming        | 3.3V    |
| OLED_DISPLAY   | DISPLAY | SSD1306 0.96 inch OLED | Displays pre-boot menu, battery status, temperature, and power telemetry         | 3.3V    |
|                |         |                        |                                                                                  |         |
| AUDIO_AMP      | AUDIO   | TPA3110D2              | Drives portable speakers with filtered Class-D amplification                     | 5V      |
## Design Schemes

### RP2040 Integrated Power Management & Isolated Audio Scheme ★

Features an onboard RP2040 controlling synchronous step-down rails (1.0V, 1.15V, 1.8V, 3.3V, 5.0V), 2S USB-C charging with fuel gauge, OLED menu, and isolated I2S audio.

**Trade-offs:** Highest integration and best telemetry/OLED control, slightly more complex layout required for audio isolation.

## Key IC selection

**2S Synchronous Switch-Mode Charger**

- Need: Fast and safe charging for a 7.4V 2S Li-ion battery pack from a 5V/9V USB-C input source with thermal regulation.
- Class: Switch-mode 2-cell Li-ion battery charger IC
- Shortlist: TI BQ25606, MPS MP2656
- Pick: Select based on input voltage range, switching frequency, I2C programmability, and available package footprint in BOM.

**Power State Machine and UI Controller**

- Need: Multi-channel GPIO control for power rail sequencing, I2C/SPI interfaces for telemetry and OLED UI, and low power consumption.
- Class: Dual-core ARM Cortex-M0+ microcontroller with integrated flash support
- Shortlist: Raspberry Pi RP2040 + W25Q16JV, STM32G030F6P6
- Pick: Select based on peripheral availability (I2C/SPI), cost, package size, and developer ecosystem familiarity in BOM.

**Core Voltage Regulator for Wii Hollywood/Broadway**

- Need: Ultra-low ripple, high-efficiency step-down conversion to deliver tight-tolerance 1.0V and 1.15V rails under rapid load transients.
- Class: High-efficiency synchronous step-down DC-DC converter
- Shortlist: TI TPS62840, MPS MP2143
- Pick: Select based on output current rating, transient response performance, quiescent current, and inductor footprint constraints.

## Risk register

| ID | Area | Severity | Detail | Mitigation |
|----|------|----------|--------|------------|
| r1 | emi | high | High-frequency switching noise from multiple synchronous buck converters (TPS62130, TPS62840) and Class-D audio amplifier (TPA3110D2) can couple into the sensitive I2S audio DAC and Wii core power rails. | Ensure strict ground plane separation between analog audio and digital/power planes, keep switching loops extremely tight, and place Pi-filters on the 5.0V audio supply rail. |
| r2 | thermal | med | Thermal dissipation in a compact portable chassis from the 2S charging IC (BQ25606) and Class-D amplifier (TPA3110D2) during high-load simultaneous charging and gaming. | Include adequate thermal vias under the charger and amplifier IC pads connecting to internal ground copper planes to act as a heat spreader. |
| r3 | firmware | high | Incorrect power-up or power-down sequencing between the 1.0V/1.15V core rails and the 1.8V/3.3V auxiliary rails can cause latch-up or permanent hardware damage to the Wii Hollywood/Broadway chips. | Implement a rigorous, hardware-verified state machine inside the RP2040 firmware with strict timing delays between rail enables monitored via power-good feedback pins. |

## Key Components

> 10/10 modules matched to real parts. See the **BOM** tab for the full component table.

**BQ25606** (`BQ25606RGER`)
  > Candidate 2 (BQ25606RGER) matches the primary MPN BQ25606 and is manufactured by Texas Instruments (the original manufac

**XB7608A** (`XB7608A`)
  > The candidate matches the primary MPN XB7608A for a dual-cell Li-ion protection IC.

**TPS62840 / MP2143** (`MP2143DJ`)
  > Candidate 2 (MP2143DJ) matches the primary MPN with a standard ordering suffix (DJ) and has the correct manufacturer (Mo

**TPS62130** (`TPS62130A`)
  > Candidate 2 is manufactured by Texas Instruments and its MPN (TPS62130A) matches the primary MPN with an ordering suffix

**TPS62130A** (`TPS62130A`)
  > Exact match for primary MPN TPS62130A and correct manufacturer Texas Instruments.

**RP2040 + W25Q16JV** (`W25Q16`)
  > The candidate W25Q16 matches the primary MPN W25Q16JV (Winbond flash memory IC) required for the module.

## Build Summary

- **Difficulty:** Advanced
- **Estimated time:** 6-8 hours
- **Tools required:** Digital Multimeter, Reflow Oven, Stereo Microscope, Soldering Iron, DC Bench Power Supply, Oscilloscope

> See the **Build** tab for the full step-by-step assembly guide.

---
*Generated by YuanQi AI Design System · Query: Custom All-In-One Wii Portable Power & Management Board for a handheld Nintendo Wii (OMGWTF-trimmed motherboard). Features: 7.4V 2S Li-ion battery pack, USB-C charging & 2S BMS protection, synchronous step-down regulators for 1.0V (CPU), 1.15V (GPU), 1.8V, 3.3V, 5.0V, RP2040/ESP32 MCU for power state machine and telemetry, monochrome OLED display, power button / pre-boot menu, Class-D audio amplifier/I2S DAC with isolated power planes. JLCPCB SMT compatible.*