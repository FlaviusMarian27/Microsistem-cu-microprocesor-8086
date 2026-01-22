# 8086 Microsystem (EasyEDA)

## Overview
This project is an EasyEDA schematic of a small microsystem built around the **Intel 8086**. The system includes ROM/RAM memory, a serial and a parallel I/O interface, a 3x4 keypad, 6 status LEDs, and a multiplexed 7-segment display driven through address decoding and an 8255 PPI. :contentReference[oaicite:0]{index=0}

## Main features
- **CPU:** Intel 8086 (minimal mode), clocked from a ~1 MHz oscillator  
- **Address demultiplexing:** 8086 AD0–AD15 are latched using **two 74LS373** controlled by **ALE**
- **ROM:** **128 KiB EPROM** using 27C2048, mapped at **00000h–1FFFFh**
- **RAM:** **64 KiB SRAM** using 62512, mapped at **40000h–4FFFFh**
- **Serial I/O:** **8251 USART**, address window selectable via **S1**
- **Parallel I/O:** **8255 PPI**, address window selectable via **S2**
- **Inputs/Outputs:**
  - **Keypad:** 3x4 matrix (12 keys) with pull-ups
  - **LEDs:** 6 status LEDs via 8255 ports
  - **7-segment display:** multiplexed digits with segment resistors and digit select via 74LS138

## Memory map (1 MB address space, decoded with 74LS138)
The upper address bits **A19..A17** are decoded with a **74LS138** to split the 1 MB space into **8 blocks of 128 KiB**. In this design:
- **EPROM:** `00000h – 1FFFFh` (128 KiB)
- **SRAM:**  `40000h – 4FFFFh` (64 KiB inside the 128 KiB block)

## I/O map (peripherals moved by DIP switches)
### 8251 (Serial) – selected by switch **S1**
- **S1 = A:** `0AF0h – 0AF2h`
- **S1 = B:** `0BF0h – 0BF2h`

Typical register meaning:
- `base + 0` → Data register  
- `base + 1/2` → Command/Status (depending on C/D and wiring)

### 8255 (Parallel) – selected by switch **S2**
- **S2 = A:** `0C70h – 0C76h`
- **S2 = B:** `0D70h – 0D76h`

8255 internal register select (via A1:A0):
- `00` → Port A
- `01` → Port B
- `10` → Port C
- `11` → Control register

## Hardware structure (what’s implemented)
### CPU core (8086)
- Uses multiplexed address/data bus: **AD0–AD15**
- **ALE** drives the 74LS373 latches to hold **A0–A15**
- **RD# / WR#** control ROM/RAM and I/O reads/writes
- **MN/MX#** tied for **minimum mode**
- **READY** tied high (no wait states)
- **RESET** available for system initialization

### Serial interface (8251)
- Connected on D0–D7 data bus
- Has RXD/TXD routed to a serial connector (for USB-TTL usage)
- Clock input provided from the system clock (derived)

### Parallel interface (8255)
One 8255 is used to drive multiple peripherals:
- **7-segment segments** (via Port A bits)
- **Digit select** through a **74LS138** (3 select lines from 8255)
- **Keypad scan** (rows driven, columns read)
- **6 LEDs** (pins on Port B / Port C)

### Keypad (3x4)
- 4 row lines + 3 column lines
- Columns are pulled up with **10 kΩ** resistors; a pressed key pulls a column low when the active row is driven low

### LEDs
- 6 LEDs controlled through 8255 pins with **470 Ω** series resistors  
- (Depending on the final wiring, the control logic can be active-high or active-low; the project includes dedicated routines for LED ON/OFF.)

### 7-segment display
- Multiplexed 7-segment digits (current schematic uses 3 displays)
- Segment lines are shared; each segment has a **470 Ω** resistor
- Digit enable is generated using **74LS138** (one output per digit)

## Software (assembly subroutines provided)
All programs are written as **subroutines** intended to be called from a main program. Provided routines include:
- **8251 init** (8 data bits, no parity, x16 factor, 9600 bps setup)
- **Serial TX/RX**
  - TX waits for TxRDY then outputs the byte
  - RX waits for RxRDY then reads the byte
- **8255 init**
  - Mode 0, with ports configured for display/LED output and keypad input (per design)
- **Parallel transmit**
  - Writes a byte directly to the 8255 output port (no handshake)
- **Keypad scan (3x4)**
  - Activates rows one by one and tests columns, returning the detected key (‘0’..‘9’, ‘*’, ‘#’)
- **LED control**
  - Routines to turn on a specific LED and to turn all LEDs off
- **Hex to 7-segment**
  - Converts a nibble (0..F) to a 7-segment code and outputs it

> Note: Port base addresses in the assembly should be defined as constants to match the selected address window (S1/S2) used in the hardware.

## How to use / test (practical)
1. Open the schematic in **EasyEDA** and inspect the blocks: CPU + latches, memory + decoder, 8251, 8255, keypad/LED/display.
2. For serial tests, connect a **USB-TTL adapter** to the serial header (TXD/RXD/GND).
3. Use firmware routines to:
   - initialize 8251/8255
   - send/receive bytes over serial
   - scan the keypad and display typed values on 7-segment
   - toggle LEDs for status/debug

## Author / Course
Project made for “Microprocessor Design” (UPT, AC). :contentReference[oaicite:1]{index=1}
