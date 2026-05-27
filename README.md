# Ferris Goes Vroom — Main Firmware

## Overview

This is the main firmware for the Ferris Goes Vroom autonomous line-following car, written in Rust using the Embassy async framework on an STM32 Nucleo-U545RE-Q microcontroller.

## How it works

The firmware runs a single async loop at 50ms intervals that:

1. **Reads the 5 IR sensors** (TCRT5000) — each sensor returns LOW when it detects a black line, HIGH on white surface
2. **Determines line position** — matches the sensor pattern to one of 9 positions: center, left, right, center-left, center-right, far-left, far-right, all-black, no-line
3. **Controls the motors with PWM** — adjusts speed differentially based on position:
   - Straight ahead: both motors at 100%
   - Gentle turn: outer motor 100%, inner motor 50%
   - Sharp correction: outer motor 100%, inner motor 0%
   - No line: both motors stop
4. **Updates the LEDs** — red when no line, blue when centered, green when moving
5. **Reads MPU6050** — integrates X-axis acceleration to estimate speed
6. **Updates the OLED display** — shows line position, motor direction, and estimated speed

## Pin Mapping

| Component | Signal | STM32 Pin | Arduino Pin |
|-----------|--------|-----------|-------------|
| TCRT5000 | S1 (left) | PA8 | D7 |
| TCRT5000 | S2 | PC7 | D8 |
| TCRT5000 | S3 (center) | PC6 | D9 |
| TCRT5000 | S4 | PC9 | D10 |
| TCRT5000 | S5 (right) | PA7 | D11 |
| L298N | IN1 | PC8 | D2 |
| L298N | IN2 | PB3 | D3 |
| L298N | IN3 | PB5 | D4 |
| L298N | IN4 | PB4 | D5 |
| L298N | ENA (PWM) | PB10 | D6 |
| L298N | ENB (PWM) | PA6 | D12 |
| LED green | — | PA0 | A0 |
| LED blue | — | PA4 | A2 |
| LED red | — | PC1 | A4 |
| OLED SSD1306 | SCL | PB6 | D15 |
| OLED SSD1306 | SDA | PB7 | D14 |
| MPU6050 | SCL | PB6 | D15 |
| MPU6050 | SDA | PB7 | D14 |

## Constants

| Constant | Value | Description |
|----------|-------|-------------|
| `SPEED_FAST` | 100% | Full speed — straight ahead or outer motor on turn |
| `SPEED_SOFT` | 50% | Half speed — inner motor on gentle turn |
| `DT` | 0.05s | Loop interval for speed integration |
| `MPU_ADDR` | 0x68 | I2C address of MPU6050 |
| `MAX_DUTY` | 10000 | PWM max duty ticks (16MHz / 1600Hz) |

## How it communicates

The firmware uses three communication protocols:

- **GPIO Input** — reads the 5 IR sensor channels (S1–S5) as digital HIGH/LOW signals
- **GPIO Output** — controls motor direction (IN1–IN4) and LED state
- **PWM (TIM2_CH3, TIM3_CH1)** — controls motor speed via L298N ENA/ENB pins at 1600Hz
- **I2C** — communicates with both the SSD1306 OLED display (0x3C) and the MPU6050 accelerometer (0x68) on the same shared bus (PB6=SCL, PB7=SDA)
