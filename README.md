# I2C Multi Bus Controller Verification (SystemVerilog)

## Overview
This project builds a SystemVerilog verification environment for the OpenCores I2C Multiple Bus Controller (IICMB). The DUT is a master-only I2C controller that can select and operate on one of up to 16 I2C buses, and is exposed as an 8-bit slave on a Wishbone bus. :contentReference[oaicite:0]{index=0}

The verification goal is to validate correct Wishbone-driven programming of the controller registers and correct I2C bus behavior for start, stop, read, write, and bus selection operations.

## DUT Summary (From Spec)
Key features of the IICMB core include:
- I2C master mode operation with support for up to 16 distinct I2C buses :contentReference[oaicite:1]{index=1}
- Wishbone top-level integration (8-bit Wishbone slave) :contentReference[oaicite:2]{index=2}
- Byte-level and bit-level FSMs that generate I2C waveforms (start, repeated start, stop, read, write) :contentReference[oaicite:3]{index=3}

The controller is driven using byte-level commands (Start, Stop, Read with Ack/Nak, Write, Set Bus, Wait) and returns corresponding responses (Done, No Ack, Arbitration Lost, Byte, Error). :contentReference[oaicite:4]{index=4}

## Register Model (Wishbone Access)
Wishbone and Avalon-MM versions provide a register block with these registers:
- CSR (Control/Status) at offset 0x00
- DPR (Data/Parameter) at offset 0x01
- CMDR (Command) at offset 0x02
- FSMR (FSM States) at offset 0x03 :contentReference[oaicite:5]{index=5}

Programming behavior:
- After reset, CSR.E must be set to 1 to enable the core (otherwise writes to other registers are blocked). :contentReference[oaicite:6]{index=6}
- Write DPR first to provide data/parameters, then write CMDR.CMD to start execution of a command. :contentReference[oaicite:7]{index=7}
- Command completion is indicated by CMDR status bits (DON, NAK, AL, ERR). :contentReference[oaicite:8]{index=8}

## Verification Architecture
The verification environment is organized to separate bus-level stimulus from I2C protocol observation.

Typical components:
- Wishbone driver: performs register reads/writes to CSR, DPR, and CMDR to initiate I2C operations
- I2C monitor/slave model: observes SDA/SCL activity and checks protocol-level behavior
- Scoreboarding/checks: validates that I2C transactions match the intended Wishbone programming sequence
- Assertions: optional protocol and sequencing assertions to catch violations early

The testbench treats the DUT as a black box and verifies it only through its published interfaces (Wishbone + I2C lines).

## Supported I2C Operations
The verification focuses on correctness of:
- Start and repeated-start generation
- Stop generation
- Byte write and acknowledgment handling
- Byte read with Ack/Nak behavior
- Bus selection via Set Bus

These operations correspond directly to the byte-level command set and responses defined in the IICMB specification. :contentReference[oaicite:9]{index=9}

## Test Scenarios
The test suite is built around Wishbone-driven programming sequences that mirror the official programming examples in the spec, such as:
- Enable core by setting CSR.E
- Select a bus using DPR + Set Bus command
- Issue Start, Write address, Write data, Stop
- Perform reads using repeated Start and Read With Nak, then read DPR for the received byte :contentReference[oaicite:10]{index=10}

These scenarios validate both:
- Correct register programming order and command completion behavior
- Correct I2C bus waveform sequencing and transaction content

## How To Run
This repository may include a Makefile-based simulation flow.

Typical usage:
1. Go to the project bench directory (example: project_benches/proj_1/sim)
2. Run the simulation target (example):
   make debug

If your environment uses a simulator command line flow (Questa/Xcelium), compile and run the top-level testbench and review:
- transcript logs for Wishbone transactions and I2C activity
- waveforms for SDA/SCL timing and command sequencing

## Project Structure (Typical)
- project_benches/
  - Testbench top and simulation scripts
- verification_ip/
  - Reusable verification components (interfaces, monitors, drivers)

(Directory names may vary by course infrastructure, but the intent is to keep reusable VIP separate from per-project benches.)

## Key Skills Demonstrated
- SystemVerilog verification environment development
- Protocol verification for I2C transactions (start/stop/read/write)
- Bus-functional stimulus through Wishbone register programming
- Functional checking aligned to published IP specification
- Debugging using logs and waveform analysis

## References
- OpenCores I2C Multiple Bus Controller IP Core Specification https://opencores.org/projects/iicmb
