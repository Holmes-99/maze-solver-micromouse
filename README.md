# Toots - Autonomous Micromouse Robot

An autonomous maze-solving robot built on an ESP32, using a floodfill algorithm, PID-corrected motor control, and time-of-flight wall sensing to navigate an 8×8 maze without human input.

**Team:** Shatha Abualrub, Razan Shalabi, Lara Daifallah, Ghada Swalha

**Course:** [COURSE CODE AND NAME]

**Instructor:** Wasel Ghanem

**Institution:** Birzeit University

<img src="media/speedrun.gif" width="360">

*Toots completing a speed-run pass after the exploration phase.*

<!-- Optional: add a still photo of the robot -->
<!-- <img src="media/toots.jpg" width="360"> -->

[Full demo video](LINK_TO_VIDEO)

## Overview

Toots explores an 8×8 maze, builds a map of its walls, and computes the shortest path to the center. A run happens in three phases:

1. **Exploration:** the robot drives from the start cell toward the center, mapping walls as it goes.
2. **Return home:** it drives back to the start using only cells it has already visited, so the route is guaranteed to be known and safe.
3. **Speed run:** it follows the precomputed shortest path from start to center at higher speed.

Core capabilities:

- Real-time wall detection using three VL53L0X time-of-flight sensors
- Wheel odometry via interrupt-driven magnetic encoders
- Shortest-path planning with a floodfill (BFS) solver
- PID-corrected differential drive to maintain heading between walls
- A web dashboard, served directly from the ESP32, for live monitoring and parameter tuning

## Results

| Metric | Value |
|---|---|
| Maze size | 8×8 |
| Exploration time | [XX] s |
| Speed-run time | [XX] s |
| Successful runs | [X] of [Y] attempts |
| Straight-line drift per cell | [X] mm |

<!-- Remove any row you did not measure rather than guessing a number. -->

## Team Contributions

| Member | Responsibility |
|---|---|
| Shatha Abualrub | Motor driver subsystem: DRV8833 H-bridge direction control, PWM speed regulation, encoder-based PID motor synchronization |
| Razan Shalabi | [CONTRIBUTION] |
| Lara Daifallah | [CONTRIBUTION] |
| Ghada Swalha | [CONTRIBUTION] |

## Documentation

| Resource | Description |
|---|---|
| [`hardware/README.md`](hardware/README.md) | Full parts list, pin map, and per-component wiring notes |
| [`hardware/3d-models/`](hardware/3d-models/README.md) | Chassis STL file and Fusion 360 source |
| [`software/README.md`](software/README.md) | Firmware architecture, run modes, dashboard API, tunable parameters |
| [Trello board](PUBLIC_BOARD_LINK) | Full project build log (read-only) |

## Hardware Summary

| Component | Part | Role |
|---|---|---|
| MCU | ESP32 Dev Kit | Runs firmware and web dashboard |
| Motor driver | DRV8833 | Drives both motors |
| Motors | N20 DC with magnetic encoder, 6V, 530 RPM | Differential drive with speed feedback |
| Distance sensors | VL53L0X ×3 | Front, left, right wall detection |
| Power | 2× 18650 cells + UPS board | Regulated 5V supply |
| Chassis | Custom 3D-printed | Sized to maze cell constraints |

<!-- Optional: add a wiring diagram image -->
<!-- <img src="media/wiring.png" width="600"> -->

Full specifications and wiring: [`hardware/README.md`](hardware/README.md)

## How It Works

**Wall sensing.** Three ToF sensors feed a correction function that keeps the robot centered between walls, or at a fixed offset from a single wall, while driving forward. All three VL53L0X sensors share one I2C bus and power up with the same default address. At boot, their XSHUT pins hold all but one in reset, and each sensor is woken one at a time and assigned a unique address.

**Motor control.** The DRV8833 dual H-bridge sets each motor's direction, and PWM duty cycle sets its speed. Interrupt-driven encoder ticks on both wheels are compared through a PID loop that adjusts PWM to keep the two motors synchronized, so the robot holds a straight line.

**Path planning.** A floodfill (BFS) solver runs outward from the goal cells and gives every cell its distance to the center. At each step the robot moves to the neighbor with the lowest value. When a new wall is discovered, the map is reflooded and the values update automatically. Because of this, dead ends need no special handling: once a dead end is mapped, its cells get higher values and the robot backs out on its own. When two neighbors tie, the robot prefers the unvisited one, which speeds up exploration.

**Connectivity.** WiFi credentials are never hardcoded. On first boot, the ESP32 opens a setup access point (`TOOTs-Setup`); the operator selects a network and enters credentials once, which are then stored to flash.

**Monitoring.** A dashboard served at `http://toots.local` shows live sensor data, current position, and run mode, with controls to tune PID, turn timing, and wall-following parameters without re-flashing.

Full technical detail: [`software/README.md`](software/README.md)

## Getting Started

**Requirements:**
- Arduino IDE with the ESP32 board package
- Libraries: `WiFiManager` (tzapu), `VL53L0X`, `ESPmDNS` (bundled with the ESP32 core)

**Steps:**
1. Open `software/toots.ino` in the Arduino IDE
2. Select the correct ESP32 board and port
3. Upload
4. On first boot, connect to the `TOOTs-Setup` network and enter your WiFi credentials
5. Open `http://toots.local` (or the IP shown in Serial output) to access the dashboard

## Repository Structure

```
maze-solver-micromouse/
├── README.md
├── LICENSE
├── media/
│   └── speedrun.gif            # Speed-run demo
├── software/
│   ├── README.md               # Firmware documentation
│   └── toots.ino               # Main firmware sketch
└── hardware/
    ├── README.md               # Parts list, pin map, wiring
    ├── components/             # [DESCRIBE: e.g. datasheets / component photos]
    └── 3d-models/              # Chassis STL and CAD source
```

## Background

The project began with component selection and procurement, followed by chassis design in Fusion 360, built to fit within maze cell constraints while keeping weight low enough not to strain the motors. The chassis was 3D-printed and assembled alongside firmware development.

The floodfill solving logic was validated in simulation before deployment to hardware, to confirm correctness of the algorithm independent of sensor noise or mechanical variance.

A full build log, including design decisions and iteration history, is maintained on the [Trello board](PUBLIC_BOARD_LINK).

## Known Limitations and Future Work

- [e.g. Turns are timed rather than closed-loop, so accuracy depends on battery level]
- [e.g. No diagonal movement during the speed run]
- [e.g. Tested on 8×8 only; scaling to a standard 16×16 maze would need more memory planning]

<!-- Replace these with the real limitations your team observed. -->

## License

Released under the [MIT License](LICENSE). Developed as academic coursework at Birzeit University.
