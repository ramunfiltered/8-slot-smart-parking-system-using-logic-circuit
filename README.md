# 8-Slot Smart Parking System Using Digital Logic Circuits

A digital-logic-based smart parking monitoring system for **8 parking slots**.

Each parking slot is monitored by an IR sensor. The eight sensor states are processed by a centralized multi-stage binary adder network to calculate:

- **Occupied slots: 0–8**
- **Available slots: 8–0**
- Individual slot status using LEDs
- Occupied and available counts using two 7-segment displays

This implementation is designed as an **S3 ECE Logic Circuit Design mini-project** and keeps the core operation independent of a microcontroller.

---

## Project Overview

```text
8 IR Sensors
     ↓
Signal Inversion / Logic Conditioning
     ↓
74LS283 Adder Tree
     ↓
Occupied Count (0–8)
     ├──────────────→ 74LS47 → Occupied Display
     ↓
2's-Complement Subtraction
     ↓
Available = 8 − Occupied
     ↓
74LS47 → Available Display
```

The project is adapted from the published **Smart Car Parking System based on Digital Logic Design** reference project, which uses eight slot sensors, a multi-stage 4-bit adder network, two's-complement subtraction, and seven-segment displays.

Reference:
https://www.researchgate.net/publication/391049243_Smart_Car_Parking_System_based_on_Digital_Logic_Design

---

## Final Project Scope

### Included

- 8 parking slots
- 8 IR sensor inputs
- 74LS283-based occupancy counting
- 7404-based inversion
- 2's-complement calculation for available slots
- Two 74LS47 display-driver stages
- Two common-anode 7-segment displays
- Individual slot-status LEDs
- Proteus simulation
- Breadboard hardware implementation

### Intentionally excluded

The published reference also includes additional gate-control hardware. This implementation does **not** include:

- Extra entrance sensor
- Gate motors
- Gate limit switches
- Buzzer
- Arduino as a controller

The focus is on digital logic, arithmetic, decoding, and display.

---

# 1. Working Principle

Each parking slot produces a digital occupancy signal.

```text
1 = Occupied
0 = Empty
```

If the selected IR sensor module is active-low when an object is detected, a 74LS04 inverter is used so the internal logic follows the convention above.

The eight occupancy bits are added in stages.

### Pairwise stage

```text
U1 = S1 + S2
U2 = S3 + S4
U3 = S5 + S6
U4 = S7 + S8
```

### Intermediate stage

```text
U5 = U1 + U2
U6 = U3 + U4
```

### Final occupied count

```text
U7 = U5 + U6
```

Therefore:

```text
U7 = total occupied slots
```

The result ranges from:

```text
0000 = 0
...
1000 = 8
```

---

# 2. Available-Slot Calculation

The total capacity is fixed at 8.

```text
Available = 8 − Occupied
```

Two's-complement subtraction is used:

```text
8 − Occupied
= 8 + (~Occupied) + 1
```

Therefore the final subtraction stage:

- Uses a 7404 to invert the occupied-count bits.
- Uses another 74LS283 for the addition.
- Uses `C0 = 1` to complete the two's-complement operation.

The available count also ranges from:

```text
0 to 8
```

---

# 3. Adder Allocation

| IC | Function |
|---|---|
| U1 | Slot 1 + Slot 2 |
| U2 | Slot 3 + Slot 4 |
| U3 | Slot 5 + Slot 6 |
| U4 | Slot 7 + Slot 8 |
| U5 | U1 + U2 |
| U6 | U3 + U4 |
| U7 | Total occupied count |
| U8 | 7404 inverter section |
| U9 | 8 − occupied |
| U10 | Occupied display driver |
| U11 | Available display driver |

---

# 4. Simulation Version — Proteus

The simulation uses switches or `LOGICSTATE` inputs in place of physical IR sensors.

```text
SW1 → Slot 1
SW2 → Slot 2
SW3 → Slot 3
SW4 → Slot 4
SW5 → Slot 5
SW6 → Slot 6
SW7 → Slot 7
SW8 → Slot 8
```

Recommended simulation convention:

```text
Switch ON  = Slot occupied
Switch OFF = Slot empty
```

### Test cases

| Occupied | Binary | Available | Binary |
|---:|:---:|---:|:---:|
| 0 | `0000` | 8 | `1000` |
| 1 | `0001` | 7 | `0111` |
| 2 | `0010` | 6 | `0110` |
| 3 | `0011` | 5 | `0101` |
| 4 | `0100` | 4 | `0100` |
| 5 | `0101` | 3 | `0011` |
| 6 | `0110` | 2 | `0010` |
| 7 | `0111` | 1 | `0001` |
| 8 | `1000` | 0 | `0000` |

### Example

If:

```text
SW1 = 1
SW2 = 1
SW3 = 1
SW4 = 0
SW5 = 1
SW6 = 1
SW7 = 1
SW8 = 0
```

then:

```text
Occupied = 6
Available = 2
```

Expected displays:

```text
OCCUPIED      AVAILABLE

    6              2
```

---

# 5. Hardware Version

The physical version replaces the simulated switches with eight digital IR sensor modules.

```text
Slot 1 → IR1
Slot 2 → IR2
...
Slot 8 → IR8
```

Each sensor is positioned so that a vehicle in the slot changes the sensor output.

The final hardware flow is:

```text
IR Sensors
   ↓
74LS04 / signal conditioning
   ↓
74LS283 adder tree
   ↓
Occupied count
   ↓
8 − Occupied
   ↓
74LS47 drivers
   ↓
7-segment displays
```

---

# 6. Hardware Components

| Component | Quantity | Purpose |
|---|---:|---|
| 5 V digital IR sensor module | 8 | Detect vehicle occupancy |
| 74LS283 4-bit binary adder | 8 | Counting and subtraction arithmetic |
| 74LS04 Hex inverter | 1 | Invert occupied-count bits |
| 74LS47 BCD-to-7-segment driver | 2 | Drive displays |
| Common-anode 7-segment display | 2 | Occupied and available counts |
| 330 Ω resistors | 14 minimum | Segment current limiting |
| 10 kΩ resistors | As required | Pull-up / pull-down where needed |
| 1 kΩ resistors | As required | Test/status LEDs |
| 0.1 µF capacitors | ~11–12 | IC decoupling |
| 5 V regulated supply | 1 | Power |
| Breadboards | 3–4 | Hardware assembly |
| Jumper wires | 1 set | Interconnection |
| DIP-16 sockets | 10 | 74LS283 + 74LS47 |
| DIP-14 socket | 1 | 74LS04 |

---

# 7. Display Section

The display chain is:

```text
U7 → 74LS47 → Occupied display
U9 → 74LS47 → Available display
```

Use **common-anode** seven-segment displays with the 74LS47.

For each display:

```text
QA → 330 Ω → segment a
QB → 330 Ω → segment b
QC → 330 Ω → segment c
QD → 330 Ω → segment d
QE → 330 Ω → segment e
QF → 330 Ω → segment f
QG → 330 Ω → segment g

Common anode → +5 V
```

Use one resistor per segment.

---

# 8. Slot Indicators

Eight LEDs can show the status of individual slots:

```text
IR1 → LED1
IR2 → LED2
...
IR8 → LED8
```

Suggested indication:

```text
LED ON  = Occupied
LED OFF = Empty
```

This allows the user to see both the total count and the exact occupied slots.

---

# 9. Recommended Proteus Build Order

Build and test the simulation in stages:

1. Test each switch/sensor input.
2. Add the 7404 inversion stage.
3. Build U1–U4 pair adders.
4. Build U5 and U6 intermediate adders.
5. Build U7 occupied counter.
6. Verify U7 gives `0–8`.
7. Build the 7404 + U9 subtraction stage.
8. Verify U9 gives `8–0`.
9. Add U10 + occupied display.
10. Add U11 + available display.
11. Add the individual slot indicators.
12. Run all test cases.

Do not debug the complete schematic at once; validate each stage before moving to the next.

---

# 10. Expected Output

The fundamental relationship is:

```text
Occupied + Available = 8
```

Examples:

```text
0 occupied → 8 available
1 occupied → 7 available
2 occupied → 6 available
3 occupied → 5 available
4 occupied → 4 available
5 occupied → 3 available
6 occupied → 2 available
7 occupied → 1 available
8 occupied → 0 available
```

When:

```text
Occupied = 8
Available = 0
```

the parking area is **FULL**.

---

# 11. Practical Hardware Notes

- Calibrate each IR sensor individually.
- Keep all sensor modules at consistent height and alignment.
- Use a regulated 5 V supply for the TTL logic and suitable sensor modules.
- Use a common ground for the complete system.
- Place a 0.1 µF decoupling capacitor close to each logic IC.
- Test the circuit with switches before replacing them with all eight physical sensors.
- Keep the high-current loads away from the logic wiring.

IR sensors can be sensitive to alignment and object distance; this was also noted as a practical limitation in the published reference project.

---

# 12. Repository Structure

```text
8-slot-smart-parking-system/
│
├── README.md
│
├── Proteus/
│   ├── Smart_Parking.pdsprj
│   ├── Smart_Parking.pdsbak
│   └── screenshots/
│       ├── empty.png
│       ├── partial.png
│       └── full.png
│
├── Hardware/
│   ├── circuit-diagram/
│   ├── wiring/
│   ├── photos/
│   └── bill-of-materials/
│
├── Documentation/
│   ├── project-report.pdf
│   ├── block-diagram.png
│   └── presentation.pptx
│
└── LICENSE
```

---

# 13. Limitations

- IR sensors require calibration and alignment.
- The system is limited to eight parking slots in this implementation.
- It determines occupancy, not vehicle identity.
- No automated gate is included in the simplified version.
- The design is intended primarily as an educational digital-logic prototype.

---

# 14. Future Scope

Possible extensions include:

- Automated entry and exit gates
- FULL indicator and buzzer
- RFID-based vehicle identification
- Larger parking capacity
- Multi-level parking
- Remote monitoring
- Parking-time and billing functions
- FPGA implementation

---

# 15. Reference Project

**Sad Abdullah Sami, Rayhan Siddiq, Muzakkir Ahmed et al.**

*Smart Car Parking System based on Digital Logic Design*

Digital Electronics Laboratory Final Project Report  
Bangladesh University of Engineering and Technology  
September 2023

Reference:
https://www.researchgate.net/publication/391049243_Smart_Car_Parking_System_based_on_Digital_Logic_Design

The published project describes eight parking slots, IR sensing, a multi-stage 74283 adder network, two's-complement subtraction for empty-slot calculation, seven-segment displays, and individual slot LEDs.

---

# 16. Modification From the Reference

The published reference includes an additional entrance sensor and hardware for gate control, including motors, limit switches and a buzzer.

This implementation intentionally uses **only the eight parking-slot IR sensors** and focuses on:

```text
8 Sensors
   ↓
Digital Logic
   ↓
Occupied Count
   ↓
Available Count
   ↓
Displays
```

This keeps the project focused on **Logic Circuit Design** and makes it suitable for an S3 ECE mini-project.

---

# Learning Outcomes

This project demonstrates:

- Digital sensor interfacing
- Binary addition
- Multi-stage adder design
- Two's-complement subtraction
- Logic inversion
- BCD-to-7-segment decoding
- Combinational digital logic
- Proteus simulation
- Breadboard hardware implementation

---

---

## License

Add the project's chosen license before publishing the repository.
