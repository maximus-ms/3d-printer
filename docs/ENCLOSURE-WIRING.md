# Enclosure Connector Pinout — 3D Printer

> **Board:** BTT SKR Mini E3 V3.0 (STM32G0B1RE)  
> **Display:** CR10_STOCKDISPLAY (EXP1, UART1)  
> **Power supply:** 12 V external PSU  
> **Firmware:** Marlin (this repo, branch `BTT_SKR_MINI_E3_V3__12_02_2026`)

---

## Enclosure connector overview

```
┌─────────────────────────────────────────────────────────────────────┐
│  PRINTER ENCLOSURE                                                  │
│                                                                     │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                           │
│  │ IDM10 #1 │  │ IDM10 #2 │  │ IDM10 #3 │                           │
│  │ Display  │  │ Display  │  │ Z Motors │                           │
│  │ (EXP1)   │  │ (SD/aux) │  │  ×2 par. │                           │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘                           │
│       │             │             │                                 │
│       └──────┬──────┘             │          ┌─────────────────┐    │
│              ▼                    │          │  BTT SKR Mini   │    │
│         To control                └─────────▶│  E3 V3.0        │    │
│         panel (knob,                         │                 │    │
│          LCD, SD)                            │  STM32G0B1RE    │    │
│                                              │                 │    │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐    │  TMC2209 ×4     │    │
│  │  IDM20   │  │  IDM30   │  │   XT30   │    │                 │    │
│  │  Frame   │  │  Head    │  │  Hotend  │    └────────┬────────┘    │
│  │ XY mot.  │  │  E0 mot. │  │  heater  │             │             │
│  │ sensors  │  │  fans    │  │  ~3-5 A  │             │             │
│  │ ESP32cam │  │  sensors │  │          │    ┌────────┴────────┐    │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘    │     XT60_in     │    │
│       │             │             │          │   12 V from PSU │    │
│       ▼             ▼             ▼          └─────────────────┘    │
│   To printer     To print     To hotend                             │
│   frame          head         (power)        ┌─────────────────┐    │
│                                              │  XT60 + 2.54    │    │
│                                              │  Bed: heater    │    │
│                                              │  + thermistor   │    │
│                                              └─────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Full board pinout — BTT SKR Mini E3 V3.0

### Steppers (TMC2209 UART, MSerial4)

| Axis | Step | Dir  | Enable | TMC Addr | Connector |
|------|------|------|--------|----------|-----------|
| X    | PB13 | PB12 | PB14   | 0        | XM        |
| Y    | PB10 | PB2  | PB11   | 2        | YM        |
| Z    | PB0  | PC5  | PB1    | 1        | ZAM/ZBM   |
| E0   | PB3  | PB4  | PD1    | 3        | E0M       |

TMC UART: PC10 (TX) / PC11 (RX) — UART4, shared bus, addressed by slave address.

### Endstops / sensors

| Function         | MCU Pin | Board header     | Trigger state    |
|------------------|---------|------------------|------------------|
| X endstop        | PC0     | X-STOP           | HIGH             |
| Y endstop        | PC1     | Y-STOP           | HIGH             |
| Z endstop        | PC2     | Z-STOP           | HIGH             |
| Filament runout  | PC15    | E0-STOP          | LOW, pull-up     |
| Z probe (3DTouch)| PC14    | PROBE            | HIGH             |
| Servo0 (3DTouch) | PA1     | SERVOS           | PWM              |

### Temperature (analog inputs)

| Sensor       | MCU Pin | Header |
|--------------|---------|--------|
| Hotend (TH0) | PA0     | TH0    |
| Bed (TB0)    | PC4     | TB0    |

### Heaters (MOSFET)

| Heater | MCU Pin | Header | Notes              |
|--------|---------|--------|--------------------|
| Hotend | PC8     | HE     | Power, via XT30    |
| Bed    | PC9     | HB     | Power, via XT60    |

### Fans (MOSFET, VIN-switched)

| Fan                | MCU Pin | Header | Purpose                  |
|--------------------|---------|--------|--------------------------|
| FAN0 (part cool)   | PC6     | FAN0   | Part cooling — on head   |
| FAN1 (hotend cool) | PC7     | FAN1   | Hotend cooling — on head |
| FAN2 (controller)  | PB15    | FAN2   | Board cooling — in case  |

Fan connector pinout: pin 1 = VIN (+12 V), pin 2 = MOSFET drain (switched ground).
**FAN V− lines are NOT GND — do not tie them together or to ground!**

### UART ports

| UART   | TX   | RX   | Board header     | Usage                      |
|--------|------|------|------------------|----------------------------|
| USB    | PA11 | PA12 | USB-C            | SERIAL_PORT −1 (primary)   |
| USART1 | PA9  | PA10 | EXP1             | LCD serial (reused for display) |
| USART2 | PA2  | PA3  | TFT              | SERIAL_PORT_2 (ESP3D/cam/klipper) |
| UART4  | PC10 | PC11 | —                | TMC2209 UART (internal)    |

### SPI1 port (accelerometer)

| Function | MCU Pin | SPI1 header pin |
|----------|---------|-----------------|
| CS       | PD9     | 3               |
| SCK      | PA5     | 4               |
| MOSI     | PA7     | 5               |
| MISO     | PA6     | 6               |
| 3.3 V    | —       | 7               |
| GND      | —       | 8               |

SPI1 bus is shared with the SD card (SD CS = PA4). Separate chip selects — no conflicts.

### Miscellaneous

| Function       | MCU Pin | Notes                            |
|----------------|---------|----------------------------------|
| NeoPixel       | PA8     | WS2812B, max 7 LEDs w/o DC-DC    |
| Status LED     | PD8     | Onboard LED                      |
| Power Loss Det | PC12    | PWR-DET (voltage divider)        |
| PS_ON          | PC13    | PSU control (unused)             |
| I2C SCL        | PB6     | Soft I2C, EEPROM                 |
| I2C SDA        | PB7     | Soft I2C, EEPROM                 |
| SD CS          | PA4     | Onboard SD card                  |
| SD Detect      | PC3     | Onboard SD card                  |

---

## Line-sharing rules

### Lines that CAN be shared

| Line     | Reason                                     | Limit                       |
|----------|--------------------------------------------|-----------------------------|
| **GND**  | Single copper plane on PCB                 | Min 2 per connector         |
| **+5 V** | Single 5 V regulator on board              | ≤1 A per IDC pin            |
| **+12 V**| FAN V+ from the same VIN rail              | ≤1 A per IDC pin            |
| **+3.3 V**| Single 3.3 V regulator                    | Accelerometer only (~3 mA)  |

### Doubling power lines

All power lines (except +3.3 V) are doubled using adjacent pin pairs to reduce resistance:

| Line     | Where doubled      | Current per wire | Rationale                                    |
|----------|--------------------|------------------|----------------------------------------------|
| **+12 V**| IDM30 pins 5-6     | ~0.3 A           | 2 connectors + long cable to head            |
| **+5 V** | IDM30 pins 13-14   | ~235 mA          | NeoPixel peaks + 2 connectors                |
| **+5 V** | IDM20 pins 16-17   | ~155 mA          | ESP32 WiFi TX peaks + reliability            |
| **+12 V**| IDM20 pins 14-15   | ~50 mA           | DC-DC for ESP32_3DCAM                        |
| **+3.3 V**| Not doubled       | ~3 mA            | Current negligible, doubling unnecessary     |

Benefits of doubling:
- Half the resistance → less voltage drop on long cables
- IDC contact resistance (~20 mΩ/pin) is halved
- Critical with 2 series connectors: 4 contacts instead of 2 per line

### Lines that MUST NOT be shared

| Line             | Reason                                                  |
|------------------|---------------------------------------------------------|
| **FAN0 V−**      | Switched by MOSFET (PC6). Tying together = loss of control |
| **FAN1 V−**      | Switched by MOSFET (PC7). Tying together = loss of control |
| **FAN V− to GND**| FAN V− is MOSFET drain, not ground!                     |
| **Signal lines** | Each signal is a dedicated GPIO                         |

### Ribbon cable zoning principle

For long cables (especially IDM30 to the head — through 2 connectors):

```
Ribbon edge (pin 1)             Center                    Ribbon edge (pin N)
┌──────────────────┐  ┌────────────────────┐  ┌──────────────────────┐
│   NOISY ZONE     │  │   GND WALL         │  │   SENSITIVE ZONE     │
│                  │  │   (spare pins      │  │                      │
│ Motors, FAN,     │  │    temporarily on  │  │ Thermistor, SPI,     │
│ +12V, endstops   │  │    GND — can be    │  │ NeoPixel, +3.3V      │
│                  │  │    reassigned)     │  │                      │
└──────────────────┘  └────────────────────┘  └──────────────────────┘
```

- Motor wires are the biggest noise source (TMC2209 chopping at 20-40 kHz).
- Thermistor is the most sensitive signal (analog, µA range).
- Spare wires tied to GND form a continuous EM shield (width = N × 1.27 mm).
- Any spare pin can be reassigned later: desolder from GND, connect a new signal.

---

## IDM10 #1 + #2 — Display (control panel)

**Display:** RepRapDiscount Full Graphic Smart Controller (robotlinking.com clone, ST7920 128×64).
The display has two 10-pin IDC connectors (EXP1 and EXP2). The board has a single EXP1 + a separate SPI1 header. Signals are split across the two panel connectors accordingly.

### Board-side: EXP1 (2×5 header)

```
                ------
(BEEPER) PB5   | 1  2 | PA15 (BTN_ENC)
(BTN_EN1) PA9  | 3  4 | RESET
(BTN_EN2) PA10   5  6 | PB9  (LCD_D4)
 (LCD_RS) PB8  | 7  8 | PD6  (LCD_EN)
          GND  | 9 10 | 5V
                ------
```

### Board-side: SPI1 (2×4 header)

```
        ------
   +5V | 1  2 | GND
    CS | 3  4 | SCK
  MOSI | 5  6 | MISO
 +3.3V | 7  8 | GND
        ------
```

Shared SPI1 bus with onboard SD card (SD CS = PA4). No conflict — different chip selects.

### Display-side: expected pinout (RAMPS-compatible)

**Display EXP1** — LCD + beeper:

```
          ------
 BEEPER  | 1  2 | BTN_ENC
 LCD_EN  | 3  4 | LCD_RS
 LCD_D4    5  6 | LCD_D5
 LCD_D6  | 7  8 | LCD_D7
     GND | 9 10 | +5V
          ------
```

**Display EXP2** — encoder + SD card SPI:

```
          ------
   MISO  | 1  2 | SCK
 BTN_EN1 | 3  4 | SD_SS
 BTN_EN2   5  6 | MOSI
 SD_DET  | 7  8 | RESET
     GND | 9 10 | KILL (will be used for +3.3V)
          ------
```

### IDM10 #1 → Display EXP1

LCD signals + beeper + encoder click. Wired from the board's EXP1 header.

Board EXP1 pins 3-8 do NOT map 1:1 to display EXP1 pins 3-8 — the BTT SKR Mini E3 V3 reorders signals compared to RAMPS. Wire according to this table:

| Pin | Display signal | MCU  | Board source | Notes                         |
|-----|----------------|------|--------------|---------------------------------|
| 1   | BEEPER         | PB5  | EXP1-1       | Buzzer                          |
| 2   | BTN_ENC        | PA15 | EXP1-2       | Encoder click                   |
| 3   | LCD_EN         | PD6  | EXP1-8       | ST7920 SID (data)               |
| 4   | LCD_RS         | PB8  | EXP1-7       | ST7920 CS                       |
| 5   | LCD_D4         | PB9  | EXP1-6       | ST7920 SCLK                     |
| 6   | —              | —    | —            | LCD_D5 (unused in SPI mode)     |
| 7   | —              | —    | —            | LCD_D6 (unused in SPI mode)     |
| 8   | —              | —    | —            | LCD_D7 (unused in SPI mode)     |
| 9   | GND            | —    | EXP1-9       | Ground                          |
| 10  | +5 V           | —    | EXP1-10      | Power                           |

### IDM10 #2 → Display EXP2 (+ SPI1 for accelerometer)

Encoder + SPI1 bus. Wired from the board's EXP1 (encoder) and SPI1 header (SPI signals).

The display's SD card SPI pins (1, 2, 4, 6) are repurposed for the board's SPI1 accelerometer bus. This works because:
- The BTT board has its own onboard SD card — the display's SD reader is unused
- SPI1 signals sit harmlessly on pins 1/2/4/6 when the display is connected (no SD card inserted = no bus conflict)
- When disconnected for calibration, the same pins serve the accelerometer directly

| Pin | Display signal     | Accel signal | MCU  | Board source | Notes                           |
|-----|--------------------|--------------|------|--------------|---------------------------------|
| 1   | ~~MISO~~ (SD n/u)  | SPI1_MISO    | PA6  | SPI1-6       | Accel / display SD MISO         |
| 2   | ~~SCK~~ (SD n/u)   | SPI1_SCK     | PA5  | SPI1-4       | Accel / display SD SCK          |
| 3   | BTN_EN1            | —            | PA9  | EXP1-3       | Encoder A                       |
| 4   | ~~SD_SS~~ (n/u)    | SPI1_CS      | PD9  | SPI1-3       | Accel chip select               |
| 5   | BTN_EN2            | —            | PA10 | EXP1-5       | Encoder B                       |
| 6   | ~~MOSI~~ (SD n/u)  | SPI1_MOSI    | PA7  | SPI1-5       | Accel / display SD MOSI         |
| 7   | ~~SD_DET~~ (n/u)   | —            | —    | —            | Spare (SD_DET → GND if card inserted!) |
| 8   | RESET              | —            | —    | EXP1-4       | Display reset button (not used by Klipper) |
| 9   | GND                | GND          | —    | —            | Ground                          |
| 10  | ~~KILL~~ (n/u)     | +3.3 V       | —    | SPI1-7       | Accel power (KILL not populated) |

**Accelerometer workflow:** disconnect both display ribbons from IDM10 #1 + #2 → plug ADXL345 adapter cable into IDM10 #2 → run input shaping calibration → reconnect display.

### ADXL345 adapter cable (IDM10 to GY-291 breakout)

The GY-291 (ADXL345) breakout has 8 pins in a row. Only 5 are needed for SPI. Solder a short ribbon (≤15 cm) from the IDM10 plug to the breakout header:

```
IDM10 #2                      GY-291 breakout
(2×5 IDC plug)               (8-pin header, top view)

Pin  9  GND ───────────────── GND
Pin 10  +3.3V ─────────────── VCC
Pin  4  SPI1_CS ───────────── CS
                         n/c  INT1
                         n/c  INT2
Pin  1  SPI1_MISO ─────────── SDO
Pin  6  SPI1_MOSI ─────────── SDA
Pin  2  SPI1_SCK ──────────── SCL
```

| IDM10 #2 pin | Signal    | GY-291 pin | Notes                                |
|--------------|-----------|------------|--------------------------------------|
| 10           | +3.3 V    | VCC        | Use 3.3 V, not 5 V (bypass on-board regulator) |
| 9            | GND       | GND        |                                      |
| 4            | SPI1_CS   | CS         | Active low                           |
| 1            | SPI1_MISO | SDO        | Data out from accelerometer          |
| 6            | SPI1_MOSI | SDA        | Data in to accelerometer             |
| 2            | SPI1_SCK  | SCL        | SPI clock                            |
| —            | —         | INT1       | Not connected (interrupt, optional)  |
| —            | —         | INT2       | Not connected                        |

**Important:** pins 3 (BTN_EN1), 5 (BTN_EN2), 7, 8 on the IDM10 plug are not connected to the breakout — leave the corresponding ribbon wires unsoldered or cut them.


---

## IDM10 #3 — Z Motors (2 motors in parallel)

**Status: DONE — cables from previous build.**

| Pin | Signal      | Notes              |
|-----|-------------|--------------------|
| 1   | Z_MOT1 A+   | Motor 1, coil A    |
| 2   | Z_MOT1 A−   |                    |
| 3   | Z_MOT1 B+   | Motor 1, coil B    |
| 4   | Z_MOT1 B−   |                    |
| 5   | Z_MOT2 A+   | Motor 2, coil A    |
| 6   | Z_MOT2 A−   |                    |
| 7   | Z_MOT2 B+   | Motor 2, coil B    |
| 8   | Z_MOT2 B−   |                    |
| 9   | GND         | Shield / spare     |
| 10  | —           | Not connected      |

Both motors are wired in parallel to ZAM/ZBM connectors on the board.

---

## IDM20 — Frame (XY motors, sensors, ESP32-CAM)

**Principle: motors on the left, GND wall in the middle, signals on the right.**

### Components and requirements

| Component           | Signals                    | Power    | Notes                       |
|---------------------|----------------------------|----------|-----------------------------|
| X motor             | 4 wires (A+, A−, B+, B−)   | —        | Via TMC2209, ~0.58-0.8 A    |
| Y motor             | 4 wires (A+, A−, B+, B−)   | —        | Via TMC2209, ~0.58-0.8 A    |
| Filament sensor     | 1 signal (PC15)            | +5V, GND | Optical/encoder, 3 wires    |
| Y endstop           | 1 signal (PC1)             | GND      | Internal pull-up in MCU     |
| Z endstop (spare)   | 1 signal (PC2)             | GND      | Backup, 3DTouch available   |
| ESP32_3DCAM         | TX (PA2), RX (PA3)         | +5V, GND | USART2, 250000 baud         |

### IDM20 pinout

```
←── physical ribbon edge (pin 1)            physical ribbon edge (pin 20) ──→

  NOISY ZONE      WALL   SIGNAL ZONE
 ┌───────────────┐ ┌──┐ ┌──────────────────────────────────────┐
 │ 1  2  3  4    │ │9 │ │11 12 13 14 15 16 17 18 19 20         │
 │ X mot  X mot  │ │G │ │FL YE ZE +12 +12 +5 +5 TX RX  G       │
 │ 5  6  7  8    │ │10│ │          └───┘   └──┘                │
 │ Y mot  Y mot  │ │G │ │        power pairs                   │
 └───────────────┘ └──┘ └──────────────────────────────────────┘
```

| Pin | Signal | MCU / Source | Zone | Notes |
|-----|--------|--------------|------|-------|
| **1** | X_MOT A+ | XM connector | NOISY | Motor X, coil A (pair 1-2) |
| **2** | X_MOT A− | XM connector | | |
| **3** | X_MOT B+ | XM connector | | Motor X, coil B (pair 3-4) |
| **4** | X_MOT B− | XM connector | | |
| **5** | Y_MOT A+ | YM connector | | Motor Y, coil A (pair 5-6) |
| **6** | Y_MOT A− | YM connector | | |
| **7** | Y_MOT B+ | YM connector | | Motor Y, coil B (pair 7-8) |
| **8** | Y_MOT B− | YM connector | | |
| **9** | **GND** | GND | **WALL** | Motor ground |
| **10** | **GND** | GND | | Wall boundary / signal ground |
| **11** | FIL_SIGNAL | PC15 (E0-STOP) | SIGNAL | Filament sensor — signal |
| **12** | Y_ENDSTOP | PC1 (Y-STOP) | | Y endstop — signal |
| **13** | Z_ENDSTOP | PC2 (Z-STOP) | | Z endstop — signal (spare) |
| **14** | **+12 V (a)** | VIN | | ⚡ Power pair: DC-DC for ESP32 |
| **15** | **+12 V (b)** | VIN | | ⚡ ~50 mA per wire (0.1 A / 2) |
| **16** | **+5 V (a)** | +5V | | ⚡ Power pair: filament + ESP32 (if no DC-DC on +12 V) |
| **17** | **+5 V (b)** | +5V | | ⚡ ~155 mA per wire (310 mA / 2) |
| **18** | ESP_TX (MCU→) | PA2 (USART2 TX) | | MCU transmits → ESP receives |
| **19** | ESP_RX (→MCU) | PA3 (USART2 RX) | | ESP transmits → MCU receives |
| **20** | **GND** | GND | | Edge ground (ribbon edge = shield) |

### IDM20 summary

- **Signal/power pins:** 17 (8 motor + 2× +12 V + 2× +5 V + 3 signal + 2 UART)
- **GND pins:** 3 (pins 9-10 + 20), no spares
- **GND wall:** 2 wires (pins 9-10) = 2.54 mm — minimal, but cable is shorter than IDM30
- **+12 V doubled:** pins 14-15 (adjacent), ~50 mA/wire — for ESP32_3DCAM DC-DC
- **+5 V doubled:** pins 16-17 (adjacent), ~155 mA/wire
- **Power grouped:** pins 14-15-16-17 (+12 V, +12 V, +5 V, +5 V) — power block
- **Motor current:** ~0.58-0.8 A RMS per wire ✓ (IDC limit 1 A)

---

## IDM30 — Print Head

**Long cable through 2 connectors — maximum zoning.**

### Components and requirements

| Component            | Signals                        | Power       | Notes                         |
|----------------------|--------------------------------|-------------|-------------------------------|
| E0 motor             | 4 wires (A+, A−, B+, B−)      | —           | Via TMC2209, ~0.58-0.8 A      |
| FAN0 (part cooling)  | 1 wire V− (switched by PC6)   | +12 V (V+)  | MOSFET on board               |
| FAN1 (hotend cooling)| 1 wire V− (switched by PC7)   | +12 V (V+)  | MOSFET on board               |
| X endstop            | 1 signal (PC0)                 | GND         | Internal pull-up in MCU       |
| 3DTouch              | Servo (PA1) + Probe (PC14)     | +5 V, GND   | 5 wires, GND shared           |
| Hotend thermistor    | 1 analog (PA0)                 | GND         | Most sensitive signal!        |
| NeoPixel lighting    | 1 data (PA8)                   | +5 V, GND   | Max 7 LEDs w/o DC-DC          |

### IDM30 pinout

```
←── physical ribbon edge (pin 1)                        physical edge (pin 30) ──→

 NOISY ZONE              GND WALL (spare ×6)     SENSITIVE ZONE
┌──────────────────────────┐ ┌─────────────────┐ ┌──────────────────────────────┐
│ 1  2  3  4  5  6  7  8   │ │15 16 17 18 19 20│ │21 22 23 24 25 26 27 28 29 30 │
│   E0mot    +12 +12 F0 F1 │ │ G  G  G  G  G  G│ │ G THR G G  G  G  5V  G NE G  │
│ 9 10 11 12 13 14         │ │                 │ │                              │
│ G XE SR PR +5 +5         │ │                 │ │                              │
└──────────────────────────┘ └─────────────────┘ └──────────────────────────────┘
```

| Pin | Signal | MCU / Source | Zone | Notes |
|-----|--------|--------------|------|-------|
| **1** | E0_MOT A+ | E0M connector | **NOISY** | Extruder, coil A (pair 1-2) |
| **2** | E0_MOT A− | E0M connector | | |
| **3** | E0_MOT B+ | E0M connector | | Extruder, coil B (pair 3-4) |
| **4** | E0_MOT B− | E0M connector | | |
| **5** | **+12 V (a)** | VIN | | ⚡ Power pair: FAN0 V+, FAN1 V+, LED 12 V |
| **6** | **+12 V (b)** | VIN | | ⚡ ~0.3 A per wire (0.6 A / 2) |
| **7** | FAN0_V− | PC6 MOSFET | | Part cooling — **DO NOT tie together!** |
| **8** | FAN1_V− | PC7 MOSFET | | Hotend cooling — **DO NOT tie together!** |
| **9** | GND | GND | | Power section ground |
| **10** | X_ENDSTOP | PC0 (X-STOP) | | X endstop — signal |
| **11** | 3DT_SERVO | PA1 (SERVO0) | | 3DTouch — servo PWM (orange) |
| **12** | 3DT_PROBE | PC14 (PROBE) | | 3DTouch — probe signal (white) |
| **13** | **+5 V (a)** | +5V | | ⚡ Power pair: 3DTouch + NeoPixel |
| **14** | **+5 V (b)** | +5V | | ⚡ ~235 mA per wire (470 mA / 2) |
| | | | | |
| **15** | **GND (spare)** | GND | **WALL** | ← Start of GND wall |
| **16** | **GND (spare)** | GND | | 6 continuous GND wires |
| **17** | **GND (spare)** | GND | | = 7.62 mm EM shield |
| **18** | **GND (spare)** | GND | | All can be reassigned |
| **19** | **GND (spare)** | GND | | in the future (desolder |
| **20** | **GND (spare)** | GND | | from GND) → End of wall |
| | | | | |
| **21** | GND | GND | **SENSITIVE** | Boundary, analog ground for thermistor |
| **22** | THERM_SIG | PA0 (TH0) | | Hotend thermistor — **most sensitive!** |
| **23** | G | | | |
| **24** | G | | | |
| **25** | G | | | |
| **26** | G | | | |
| **27** | +5 V | | | |
| **28** | GND | GND | | Ground before NeoPixel |
| **29** | NEO_DATA | PA8 (NEOPIXEL) | | NeoPixel — data (330 Ω resistor on head) |
| **30** | GND | GND | | Edge ground (ribbon edge = shield) |

### IDM30 summary

- **Signal/power pins:** 14 (4 motor + 2× +12 V + 2 FAN_V− + 5 signal + 2× +5 V + 1 NeoPixel)
- **GND pins:** 16 (pins 9, 15-26, 28, 30), of which 6 are spare (15-20)
- **GND wall:** 6 wires (pins 15-20) = 7.62 mm continuous EM shield
- **+12 V doubled:** pins 5-6 (adjacent), ~0.3 A/wire — next to FAN V− to minimize current loop area
- **+5 V doubled:** pins 13-14 (adjacent), ~235 mA/wire — buffer before GND wall
- **Physical separation:** motor (pins 1-4) and thermistor (pin 22) separated by 18 wires

### Sensitive signal protection diagram

```
Pin:  ...20│21│22│23│24│25│26│27│28│29│30
         G │G │TH│G │G │G │G │5V│G │NE│G
           │  │  │           │     │  │
           │  │  │           │     │  │
           │  │  │           │     │  │
           │  │  │           │     │  │
           │  └──── GND on both sides of thermistor
           │     │           │     │  │
           └──── GND wall ends here
                                   │  │
                             GND ──┘  └── GND
                             NeoPixel in a "pocket" between two GNDs
```

### Why +12 V is adjacent to FAN V−

```
Pin:  5│ 6│ 7│ 8│ 9
     +12 +12 F0 F1  G
      │   │   │  │
      └───┤   │  │    Fan current path:
          │   │  │    +12V → FAN → FAN_V- → MOSFET → GND
          └───┤  │
              │  │    Adjacent placement of +12V and FAN V-
              └──┤    minimizes current loop area
                 │    → less EM radiation
                 └── GND nearby as shield
```

---

## XT30 — Hotend heater

**Separate power connector. MOSFET switches on the board (PC8).**

| Contact | Signal       | Notes              |
|---------|--------------|--------------------|
| +       | HEATER_0 V+  | From HE+ on board  |
| −       | HEATER_0 V−  | From HE− on board  |

Typical current: 3-5 A at 12 V (40-60 W hotend). XT30 is rated for 30 A.

---

## XT60 — Bed heater + thermistor (2.54)

### XT60 — power

| Contact | Signal       | Notes              |
|---------|--------------|--------------------|
| +       | HEATER_BED V+| From HB+ on board  |
| −       | HEATER_BED V−| From HB− on board  |

Typical current: 10-15 A at 12 V (120-180 W bed). XT60 is rated for 60 A.

### 2.54 (2-pin) — bed thermistor

| Contact | Signal         | MCU  | Notes                 |
|---------|----------------|------|-----------------------|
| 1       | TEMP_BED signal| PC4  | Analog input (TB0)    |
| 2       | TEMP_BED GND   | GND  | Analog ground         |

---

## XT60_in — 12 V power input

| Contact | Signal    | Notes                     |
|---------|-----------|---------------------------|
| +       | +12 V     | From external PSU         |
| −       | GND       | From external PSU         |

---

## Firmware changes for new components

### 3DTouch / BLTouch

In `Marlin/Configuration.h`, uncomment:
```cpp
#define BLTOUCH
```

Enable auto bed leveling (one of):
```cpp
#define AUTO_BED_LEVELING_BILINEAR
// or
#define AUTO_BED_LEVELING_UBL
```

Verify the offset:
```cpp
#define NOZZLE_TO_PROBE_OFFSET { 10, 10, 0 }  // Measure the actual values!
```

### NeoPixel LED

In `Marlin/Configuration.h`, uncomment:
```cpp
#define NEOPIXEL_LED
#define NEOPIXEL_TYPE   NEO_GRB   // Depends on strip type (GRB/GRBW/RGB)
#define NEOPIXEL_PIXELS 6         // Number of LEDs on head (max 7 w/o DC-DC)
```

### Accelerometer (ADXL345 via SPI)

Support needs to be added in `Marlin/Configuration_adv.h` for input shaping frequency auto-calibration. SPI1 is routed through IDM10 #2 (display panel connector) — see the ADXL345 adapter cable section above.

### ESP32_3DCAM

Already configured (see `ESP3D/ESP3D-Setup.md`):
```cpp
#define SERIAL_PORT_2 2       // USART2
#define BAUDRATE_2 250000     // Must match ESP32
```

Photo sync via `M118 P1 TAKE_PHOTO` or a custom command.

---

## Soldering tips

### General

- **Continuity-test every connector** with a multimeter after soldering (beep mode).
- **Label cables** with colored tape or heat shrink on both ends.
- The first wire in a ribbon (pin 1) is usually marked with a red stripe — use it for orientation.

### IDC connectors

- Rated contact current: **1 A** (at 20 °C).
- TMC2209 motors at 0.58 A RMS are within spec. Above 0.8 A — double the motor pins.
- **Strain relief:** on moving cables (especially IDM30 to the head), always secure the ribbon against tension.

### Thermistor

- Solder twisted-pair thermistor wires as close to the GND line in the ribbon as possible.
- On the head side: 100 nF capacitor between THERM_SIG and GND for filtering.

### NeoPixel

- **330-470 Ω** series resistor on the DATA line on the head side (near the first LED).
- **100-1000 µF** capacitor between +5 V and GND near the strip on the head.

### 3DTouch

- Original cable colors: orange (servo), red (+5 V), brown (GND), white (probe), black (GND).
- Servo GND and Probe GND are shared through common connector ground.

### SPI accelerometer

- Accelerometer connects via IDM10 #2 (display panel). See ADXL345 adapter cable section.
- Total SPI cable length (board SPI1 → IDM10 #2 → breakout) should be ≤50 cm. If unstable, use software SPI with reduced clock.
- Only used for calibration (not during printing) — temporary connection is acceptable.

---

## Current load verification

### IDM20

| Pin(s) | Load              | Max current/wire   | IDC limit (1 A) |
|--------|-------------------|--------------------|------------------|
| 1-2    | X motor coil A    | 0.8 A RMS          | ✓ (0.2 A margin) |
| 3-4    | X motor coil B    | 0.8 A RMS          | ✓                |
| 5-6    | Y motor coil A    | 0.8 A RMS          | ✓                |
| 7-8    | Y motor coil B    | 0.8 A RMS          | ✓                |
| 14+15  | +12 V ×2          | ~50 mA (0.1 A / 2) | ✓✓ (large margin)|
| 16+17  | +5 V ×2           | ~155 mA (310 / 2)  | ✓✓ (large margin)|

### IDM30

| Pin(s) | Load              | Max current/wire   | IDC limit (1 A)  |
|--------|-------------------|--------------------|-------------------|
| 1-2    | E0 motor coil A   | 0.8 A RMS          | ✓ (0.2 A margin) |
| 3-4    | E0 motor coil B   | 0.8 A RMS          | ✓                 |
| 5+6    | +12 V ×2          | ~0.3 A (0.6 / 2)   | ✓✓ (large margin)|
| 7      | FAN0 V−           | ~0.3 A             | ✓                 |
| 8      | FAN1 V−           | ~0.3 A             | ✓                 |
| 13+14  | +5 V ×2           | ~235 mA (470 / 2)  | ✓✓ (large margin)|
| 27     | +5 V ×1           | ~60 mA             | ✓ (NeoPixel only)      |

