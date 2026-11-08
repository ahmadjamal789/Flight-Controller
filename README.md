# Flight-Controller
``` A Rocket Filght Controller```

The electronics for a small **flight controller**: the board that acts as the "brain" of a drone or similar vehicle. It reads motion and pressure sensors to work out how the vehicle is moving, runs the control software on an STM32 microcontroller, drives servos, logs data to a microSD card, and runs from a rechargeable battery that charges over USB-C.

This folder contains the KiCad design. **Current status: the schematic is drafted, and PCB layout hasn't started yet.**

<img width="1366" height="728" alt="Screenshot from 2026-11-05 16-14-31" src="https://github.com/user-attachments/assets/134dd9ee-3ec3-402e-9948-eef0c9b1e98f" />


## Repository layout

```
Hardware/
├── Hardware.kicad_sch    Schematic (this README explains i
├── Hardware.kicad_pcb    PCB layout (not started)

├── Hardware.kicad_pro    KiCad project file
└── lib/
    ├── lcsc.txt          List of LCSC part numbers used in the design
    ├── lcsc.py           Script that converts those parts into KiCad libraries
    ├── lcsc.kicad_sym    Generated symbols
    ├── lcsc.pretty/      Generated footprints
    └── lcsc.3dshapes/    Generated 3D models
```

## How to read the schematic

A **schematic** is a diagram of every component and how they're electrically connected. This one is a single sheet split into labelled boxes, one per function, so each block can be read, checked and debugged on its own. Wires between blocks are mostly joined with net labels (for example `+3V3`, `VBUS`, `GND`) instead of long drawn lines.

| Block | What it does |
|---|---|
| USB-C | Power input and USB data connection |
| Battery Charger | Charges the battery from USB and manages power |
| Buck-Boost Converter | Turns the changing battery voltage into a steady rail |
| 3.3V Boost | Produces the 3.3 V supply for the logic |
| STM32 | The microcontroller that runs everything |
| Crystal | Accurate clock source for the microcontroller |
| Decoupling | Small capacitors that keep the supplies clean |
| Buttons | Manual inputs, such as reset and boot |
| Servos | Connectors for driving servo motors |
| IMU | Motion sensor (gyroscope and accelerometer) |
| Pressure Sensor | Barometer for altitude |
| microSD Card | Data logging storage |

## Block-by-block

### USB-C
A USB-C connector brings in 5 V (`VBUS`) for charging and carries USB data to the microcontroller, so the board can be programmed and talked to from a computer.
<img width="597" height="526" alt="Screenshot from 2026-11-08 08-54-14" src="https://github.com/user-attachments/assets/b54be109-9ae5-4d53-8dee-7b2ebc3c2a43" />


### Battery Charger
A charger IC with an inductor charges the battery from `VBUS`. It has an I2C connection to the microcontroller and a thermistor input for battery temperature, so the firmware can monitor and control charging. *(Confirm: part number and charge current.)*
<img width="481" height="454" alt="image" src="https://github.com/user-attachments/assets/e5a1bac1-9e8d-4f6f-b3de-cad3dde1ddde" />


### Buck-Boost Converter
A buck-boost converter keeps its output steady whether the battery voltage is above or below the target as the battery drains. *(Confirm: part number and output voltage.)*
<img width="481" height="219" alt="image" src="https://github.com/user-attachments/assets/fa4ce800-b1f5-4681-80e8-8b9723bef28d" />


### 3.3V Boost
A small switching regulator that provides the `+3V3` rail used by the microcontroller and the sensors. *(Confirm: input source and part number.)*
<img width="999" height="317" alt="image" src="https://github.com/user-attachments/assets/14021df6-75f6-43bf-a657-60280f63d195" />


### STM32
The central microcontroller. Every other block connects to it:
- **I2C / SPI** to the IMU, the pressure sensor and the charger
- **SDIO or SPI** to the microSD card
- **Timer outputs** to the servo headers
- **USB** to the USB-C connector
- **Buttons** and the **crystal** for reset/boot and clocking

  
  <img width="327" height="496" alt="image" src="https://github.com/user-attachments/assets/16a4cd8e-62d9-4e01-bded-d8ea96346996" />



### Crystal
An external crystal gives the microcontroller a precise clock. The block holds a high-speed crystal for the main clock, plus a second, low-speed one for timekeeping. *(Confirm: frequencies and load capacitors.)*
<img width="585" height="520" alt="image" src="https://github.com/user-attachments/assets/e2a83a35-cffd-4814-9b1a-d432f71be797" />


### Decoupling
Every chip needs small capacitors placed right next to its power pins to absorb sudden current spikes and keep the supply steady. This block groups the capacitors for the `+3V3` rail. When the PCB is laid out, each one must sit as close as possible to the pin it serves.
<img width="495" height="512" alt="image" src="https://github.com/user-attachments/assets/595c12b0-a116-47d8-b66c-3b27c6c56e7b" />

### Buttons
Push buttons for resetting the microcontroller and entering its bootloader (boot mode), with a resistor and capacitor to give a clean, debounced signal. *(Confirm: which button does what.)*
<img width="541" height="523" alt="image" src="https://github.com/user-attachments/assets/253d204a-ae54-4b96-8f78-aeec72f5b956" />


### Servos
Header connectors for two servo motors. Each has a power pin (`+5V`), ground, and a control signal driven by a timer pin on the STM32.
<img width="446" height="508" alt="image" src="https://github.com/user-attachments/assets/4995cf86-6fdb-4fb6-b79d-997b9b773e87" />

### IMU
An inertial measurement unit: a gyroscope (rotation) and accelerometer (acceleration) in one chip. This is the main sensor for working out the vehicle's orientation. *(Confirm: part number and whether it's connected by SPI or I2C.)*
<img width="1171" height="402" alt="image" src="https://github.com/user-attachments/assets/8210305a-90c4-4f1c-bb9b-3c73f60e148c" />


### Pressure Sensor
A barometric pressure sensor, used to estimate altitude because air pressure drops as height increases. *(Confirm: part number and interface.)*
<img width="1201" height="291" alt="image" src="https://github.com/user-attachments/assets/f2be20ff-6e98-40b1-9043-87d2fa8ded4d" />

### microSD Card
A card slot for logging flight data. The data lines have pull-up resistors, which the SD standard requires so the lines sit at a known level when nothing is driving them.
<img width="1121" height="492" alt="image" src="https://github.com/user-attachments/assets/d877c890-98fd-43e9-8a33-5fe1672bcc45" />

## Power overview
```
USB-C (5 V) --> Battery Charger --> Battery
                                       |
                                       +--> Buck-Boost Converter --> regulated rail (servos, etc.)
                                       |
                                       +--> 3.3V Boost --> +3V3 --> STM32, sensors, microSD
```

*(Check this against the schematic and correct the arrows if the real power path differs.)*

## Parts library

Parts come from LCSC and are converted to KiCad symbols, footprints and 3D models with [easyeda2kicad](https://github.com/uPesy/easyeda2kicad.py). To add or update parts:

1. Put the LCSC part numbers in `lib/lcsc.txt`, one per line (`#` starts a comment).
2. Install the tool once: `pipx install easyeda2kicad`.
3. Run `python3 lib/lcsc.py`.
4. In KiCad, add `lib/lcsc.kicad_sym` under *Manage Symbol Libraries* and `lib/lcsc.pretty` under *Manage Footprint Libraries* (project-specific).

## Status and next steps

- [x] Schematic drafted, split into labelled blocks
- [ ] Check every part against its datasheet (pinouts, voltages, recommended circuits)
- [ ] Run ERC (Electrical Rules Check) and fix all errors
- [ ] Assign footprints and verify pad sizes against the datasheets
- [ ] PCB layout
- [ ] Order and assemble the board

## Tools

- [KiCad](https://www.kicad.org/) for schematic and PCB design
- [easyeda2kicad](https://github.com/uPesy/easyeda2kicad.py) for converting LCSC parts to KiCad libraries
- Git for version control
