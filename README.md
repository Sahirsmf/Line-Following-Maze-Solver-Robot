# Maze Solver Robot

A line-following robot that autonomously finds its way out of a maze using infrared sensors and a simple, reliable **left-hand rule** algorithm. Built with an Arduino Uno R3 SMD as part of a CSE350 course project.

## Overview

The robot follows a line on the ground that marks the maze path. Three TCRT5000 infrared sensors read the surface beneath the robot, and an Arduino Uno makes movement decisions based on a left-hand-rule strategy: always try to turn left first, then straight, then right, and stop if none are available. Two DC motors, driven through an L298N motor driver, handle movement.

## Hardware Components

| Component | Purpose |
|---|---|
| Arduino Uno R3 SMD | Main controller |
| 3× TCRT5000 IR sensors (Left, Front, Right) | Line detection |
| L298N motor driver | Motor direction & speed control |
| 2× DC motors | Drive wheels |
| Wooden chassis | Frame |
| 2× 3.7V 2600mAh batteries (ICR18650) | Power supply |

> A single 9V battery could not supply enough current for both motors simultaneously — two 3.7V Li-ion cells solved this.

## How It Works

### Sensing
Each TCRT5000 sensor emits infrared light and reads the reflection: a dark line absorbs more light, a white surface reflects more. This is read as a digital signal per sensor:
- `0` → line detected
- `1` → no line detected

### Left-Hand Rule Algorithm
The robot evaluates its three sensors (Left, Front, Right) against a truth table and picks an action:

| L | F | R | Action |
|:-:|:-:|:-:|---|
| 0 | 0 | 0 | Turn left 90° |
| 0 | 0 | 1 | Turn left 90° |
| 0 | 1 | 0 | Turn left 90° |
| 0 | 1 | 1 | Turn left 90° |
| 1 | 0 | 0 | Turn right 90° |
| 1 | 0 | 1 | Move forward |
| 1 | 1 | 0 | Turn right 90° |
| 1 | 1 | 1 | Stop |

This guarantees the robot always finds an exit, since it systematically favors the leftmost open path at every junction.

### Pin Configuration

**Motors**
- Right motor: IN1 = `8`, IN2 = `9`, Enable (PWM) = `3`
- Left motor: IN1 = `10`, IN2 = `11`, Enable (PWM) = `5`

**Sensors**
- Left = `7`, Front = `6`, Right = `4`

**Tunable parameters**
- `forwardSpeed = 90`, `turnSpeed = 70` (PWM)
- `turn90Delay = 250` ms — timed-delay 90° turns; tune experimentally for your chassis

## Circuit Diagram

*(Add your circuit diagram and build photos here, e.g. `![Circuit Diagram](images/circuit.png)`)*

## Code

The full sketch is in [`maze_solver.ino`](./maze_solver.ino). Core decision loop:

```cpp
int L = digitalRead(sensorLeft);
int F = digitalRead(sensorFront);
int R = digitalRead(sensorRight);

if      (L == 0 && F == 0 && R == 0) turnLeft90();
else if (L == 0 && F == 0 && R == 1) turnLeft90();
else if (L == 0 && F == 1 && R == 0) turnLeft90();
else if (L == 0 && F == 1 && R == 1) turnLeft90();
else if (L == 1 && F == 0 && R == 0) turnRight90();
else if (L == 1 && F == 0 && R == 1) moveForward();
else if (L == 1 && F == 1 && R == 0) turnRight90();
else if (L == 1 && F == 1 && R == 1) stopMotors();
```

## Getting Started

1. Wire the components as shown in the circuit diagram.
2. Open `maze_solver.ino` in the Arduino IDE.
3. Upload it to the Arduino Uno R3.
4. Place the robot on the maze's starting line and power it on (there's a 3-second startup delay before it begins).
5. Tune `forwardSpeed`, `turnSpeed`, and `turn90Delay` for your specific chassis and surface.

## Challenges & Solutions

- **Underpowered motors:** A 9V battery couldn't drive both DC motors at once — replaced with two 3.7V 2600mAh Li-ion cells.
- **Inverted sensor logic:** Reference code assumed `1 = line detected`; our sensors output the opposite, causing constant spinning. Fixed by rewriting the truth table for our sensor convention.
- **Turning overshoot:** Sensors sometimes lost or falsely detected the line mid-turn. Reducing `turn90Delay` from 350ms to 250ms improved turning accuracy.

## Limitations & Future Work

The robot cannot currently distinguish a dead end from the maze's actual exit, so it stops at either. A worthwhile improvement would be a way to tell the two apart and implement a U-turn at genuine dead ends while still stopping at the real exit.

## References

[1] Arduino Project Hub, "Line Following Robot" — https://projecthub.arduino.cc/lightthedreams/line-following-robot-34b1d3#section1
