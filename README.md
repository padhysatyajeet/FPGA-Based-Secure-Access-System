# FPGA-Based Secure Access System

## Overview

This project implements a secure access system on a Xilinx Spartan-7 FPGA using Verilog HDL. The system uses a 4×4 matrix keypad for user input and generates a 6-character One-Time Password (OTP) using a ring-oscillator-based True Random Number Generator (TRNG).

The generated OTP is transmitted through UART to an external BLE module, allowing the OTP to be communicated wirelessly. The system verifies the OTP entered through the keypad and provides access only when the complete OTP is correct.

## Features

- Implemented on a Xilinx Spartan-7 FPGA using Verilog HDL.
- 4×4 matrix keypad interface for OTP input.
- Ring-oscillator-based TRNG for hardware entropy generation.
- Uses three ring oscillators with XOR-based entropy combining.
- Two-stage synchronization for asynchronous ring-oscillator outputs.
- Von Neumann correction to reduce bias in the generated random bits.
- Generates a 6-character hexadecimal OTP.
- Rejects hexadecimal values "E" and "F" to maintain uniform selection among "0–D".
- UART transmitter configured for 115200 baud with a 100 MHz FPGA clock.
- Sends the generated OTP to an external BLE module through UART.
- OTP verification through the 4×4 keypad.
- Allows a maximum of three incorrect attempts before locking the system.
- Dedicated F-key soft reset for clearing the lockout state.
- LED indicators for reset, successful access, failed verification, and lockout status.

## System Architecture

The overall system consists of the following modules:

                  ┌──────────────────┐
                  │ Ring Oscillators    │
                  └────────┬─────────┘
                           ↓
                    ┌─────────────┐
                    │   TRNG Core   │
                    └──────┬──────┘
                           ↓
                    Random Entropy
                           ↓
                 ┌──────────────────┐
                 │ OTP Controller      │
                 └───────┬──────────┘
                         ↓
                  6-Character OTP
                         ↓
                 ┌──────────────────┐
                 │ OTP UART Sender     │
                 └───────┬──────────┘
                         ↓
                    ┌─────────┐
                    │ UART TX  │
                    └────┬────┘
                         ↓
                    BLE Module
                         ↓
                  Wireless Transfer


4×4 Keypad ──→ Keypad Scanner ──→ OTP Controller
                                      │
                                      ↓
                               OTP Verification
                                      │
                                      ↓
                                Access / Lockout
                                      │
                                      ↓
                                    LEDs

## TRNG-Based OTP Generation

The TRNG uses three ring oscillators as physical entropy sources. Their outputs are synchronized to the FPGA clock and combined using XOR logic.

The sampled bits are then processed using a Von Neumann correction scheme:

01 → 0
10 → 1
00 → Discard
11 → Discard

The resulting random bits are collected into 4-bit hexadecimal values. Values from "0" to "D" are accepted, while "E" and "F" are discarded. Six accepted hexadecimal values form the 6-character OTP.

The total number of possible OTPs is:

[
14^6 = 7,529,536
]

## Keypad Interface

The system uses a 4×4 matrix keypad.

The keypad scanner drives one column at a time and reads the row inputs to determine which key has been pressed.

The keys are mapped to hexadecimal values:

1  2  3  A
4  5  6  B
7  8  9  C
0  F  E  D

The "F" key is reserved for the soft-reset function.

## UART Communication

The generated OTP is sent from the FPGA to an external BLE module using UART.

UART configuration:

Clock Frequency : 100 MHz
Baud Rate       : 115200
CLKS_PER_BIT    : 868
Data            : 8 bits
Start Bit       : 1
Stop Bit        : 1

The UART transmitter uses an FSM with four states:

IDLE → START → DATA → STOP

The OTP sender transmits:

OTP: XXXXXX

followed by carriage return and line feed.

The BLE module handles the wireless Bluetooth communication; the FPGA communicates with the module using UART.

#$OTP Verification and Lockout

After receiving the OTP, the user enters the OTP through the keypad.

The controller compares each entered hexadecimal character with the corresponding OTP character.

Correct OTP
     ↓
Access Granted

If an incorrect OTP is entered:

Incorrect Attempt
       ↓
Attempt Counter
       ↓
3 Failed Attempts
       ↓
System Locked

The "F" key can be used to activate the soft reset and clear the lockout state.

## LED Status Indication

The FPGA LEDs provide visual feedback for different system states:

- Reset – system initialization/reset
- Success – correct OTP entered
- Failure – incorrect OTP
- Locked – maximum failed attempts reached

## Implementation

The design is written in Verilog HDL and implemented using Xilinx Vivado.

Major RTL modules include:

- "keypad_scan.v" – 4×4 keypad scanning and key detection
- "trng_core.v" – TRNG and entropy processing
- "otp_lock_controller.v" – OTP generation, verification and lockout control
- "otp_uart_sender.v" – formats and sends the OTP message
- "uart_tx.v" – UART transmission
- "keypad_reset.v" – F-key based soft reset
- "keypad_ble_top.v" – top-level system integration

## Verification

The individual RTL modules were simulated and verified using Verilog testbenches and waveform analysis. The complete design was implemented on the Spartan-7 FPGA using Vivado.

Important signals such as keypad input, entropy generation, OTP generation, UART transmission, OTP verification, and lockout status can be observed during simulation and hardware testing.

## Hardware Platform

FPGA: Xilinx Spartan-7 XC7S50
HDL: Verilog
Development Tool: Xilinx Vivado
Input: 4×4 Matrix Keypad
Wireless Interface: External BLE UART Module
Clock: 100 MHz
Communication: UART, 115200 baud
