# M32 Microcontroller System-on-Chip (SkyWater 130nm)

![HDL](https://img.shields.io/badge/HDL-Verilog-blue)
![PDK](https://img.shields.io/badge/PDK-SkyWater%20130nm%20(sky130A)-green)
![Flow](https://img.shields.io/badge/Flow-OpenLane%20%7C%20OpenROAD-purple)
![Tapeout](https://img.shields.io/badge/ASIC-RTL--to--GDSII%20Signoff-red)
![Clock](https://img.shields.io/badge/Target%20Clock-100%20MHz%20(10ns)-orange)
![Complexity](https://img.shields.io/badge/Cells-10%2C476%20Standard%20Cells-brightgreen)

A tape-out-ready 32-bit Microcontroller System-on-Chip (SoC) combining a pipelined CPU core with an on-chip memory-mapped peripheral subsystem (UART, I2C, Timers, GPIO), hardened through the open-source **OpenLane / OpenROAD ASIC flow** down to GDSII on the **SkyWater 130nm CMOS process (`sky130A`)**.

---

## 📸 Physical Implementation & Chip Layout

| Chip Die Layout (SkyWater 130nm) | Detailed Routing & Cell Placement |
| :---: | :---: |
| ![Chip Layout](screenshots/Screenshot%20from%202026-06-12%2014-01-58.png) | ![Routing Detail](screenshots/Screenshot%20from%202026-06-12%2014-02-19.png) |

---

## 🗺️ SoC Memory Map & Interconnect

The processor interfaces with on-chip RAM and peripheral hardware controllers through a unified 32-bit memory-mapped address space decoded by `addr_decoder.v`:

| Address Range | Subsystem / Peripheral | Description | Control Signals |
| :--- | :--- | :--- | :--- |
| `0x0000_0000 – 0x0000_3FFF` | **Data RAM (16 KB)** | Synchronous internal scratchpad memory | `dm = 1` |
| `0x0000_0010 – 0x0000_001F` | **GPIO Subsystem** | 32-bit bidirectional I/O, direction masks, input/output registers | `ga, gina, gena, drina` |
| `0x0000_0020 – 0x0000_002F` | **Prescaled Timer** | 32-bit down-counter, prescaler divider, overflow interrupts | `tc, tp, t_cnf` |
| `0x0000_0040 – 0x0000_004F` | **UART Transceiver** | Full-duplex TX/RX buffers, programmable baud-rate division | `ut, ur, ue` |
| `0x0000_0060 – 0x0000_006F` | **I2C Master** | Standard two-wire serial master address/data controllers | `i2c_addr, i2c_data` |

---

## 🏗️ Top-Level SoC Block Diagram

![SoC Architecture Block Diagram]

```mermaid
graph TD
    subgraph Core ["32-bit Processor Core"]
        CPU["Pipelined CPU Datapath"]
        CTRL["Hazard & Stall Control"]
        BHT["2-Bit Branch History Table"]
    end

    subgraph Interconnect ["Address Decoder & Bus Crossbar"]
        DEC["Memory Address Decoder (addr_decoder.v)"]
    end

    subgraph Peripherals ["Memory-Mapped Peripheral Subsystems"]
        RAM["16 KB Data Memory (data_mem.v)"]
        GPIO["32-Line Bi-Directional GPIO (gpio_controller.v)"]
        TIMER["32-bit Prescaled Timer (prescale_timer.v)"]
        UART["Full-Duplex UART Transceiver (uart_tx.v / uart_rx.v)"]
        I2C["I2C Serial Master Controller (I2C_addr.v / I2C_data.v)"]
    end

    CPU <--> DEC
    CTRL <--> DEC
    BHT <--> CPU
    
    DEC <--> RAM
    DEC <--> GPIO
    DEC <--> TIMER
    DEC <--> UART
    DEC <--> I2C
```

---

## 📦 Integrated Peripheral Subsystems

1. **GPIO Controller**: 32 software-configurable bidirectional I/O lines with direction masks (`tri_sig`), input registers (`pin`), and output registers (`pon`).
2. **UART Transceiver**: Asynchronous serial transmitter/receiver supporting programmable baud-rate division, framing, and TX/RX buffer status registers.
3. **Timer/Counter Subsystem**: 32-bit down-counter with programmable clock prescaler divider and periodic interrupt overflow generation.
4. **I2C Master Core**: Standard two-wire serial interface providing start/stop condition generation, byte acknowledge detection, and slave peripheral addressing.

---

## ⚙️ Hardware & Physical Design Specifications

| Parameter | Value / Specification | Source |
| :--- | :--- | :--- |
| **Architecture** | 32-bit Pipelined Datapath with Dynamic Branch Prediction | RTL |
| **Target PDK** | SkyWater 130nm Standard Cells (`sky130_fd_sc_hd`) | `config.json` |
| **Standard Cell Count** | **10,476 cells** | `reports/synthesis/1-synthesis.AREA_0.stat.rpt` |
| **Total Routing Nets** | **10,447 wires** (10,478 bits) | `reports/synthesis/1-synthesis.AREA_0.stat.rpt` |
| **Target Clock Period** | **10.0 ns (100 MHz)** | `constraints/clock.sdc` |
| **Data Memory** | 16 KB On-Chip Synchronous RAM | `rtl/data_mem.v` |
| **GPIO Count** | 32 Bi-directional Pins with Tri-state control | `rtl/gpio_controller.v` |
| **Physical Flow** | Automated OpenLane / OpenROAD Flow | `config.json` |

---

## 📁 Repository Structure & ASIC Deliverables

```
M32_MCU_V1/
├── config.json                     # OpenLane ASIC flow configuration
├── constraints/
│   └── clock.sdc                   # SDC timing constraints (100 MHz target clock)
├── rtl/                            # Synthesizable Verilog RTL source (41 modules)
│   ├── top.v                       # Top-level SoC integration
│   ├── addr_decoder.v              # Interconnect bus address decoder
│   ├── bht.v                       # 2-bit Dynamic Branch History Table
│   ├── fwd_unit.v / stall_unit.v   # Pipeline hazard forwarding & load-use stall logic
│   ├── gpio_controller.v           # 32-channel bidirectional GPIO
│   ├── prescale_timer.v            # 32-bit programmable timer
│   ├── uart_tx.v / uart_rx.v       # Full-duplex UART transceiver
│   └── I2C_addr.v / I2C_data.v     # I2C serial protocol controller
├── reports/                        # Implementation logs & verification sign-off
│   ├── synthesis/                  # Cell count statistics, gate-level STA reports
│   └── cts/                        # Clock tree synthesis buffer insertion reports
├── results/final/                  # Final Tape-out Sign-off Artifacts
│   ├── gds/                        # Tape-out ready GDSII stream file
│   ├── def/                        # Placed & routed Design Exchange Format
│   ├── lef/ & maglef/              # Abstract layout models for hierarchical reuse
│   ├── spef/                       # Standard Parasitic Extraction (RC parasitics)
│   ├── sdf/                        # Standard Delay Format for gate-level timing sim
│   ├── spi/lvs/                    # Extracted SPICE netlist for LVS verification
│   └── verilog/gl/                 # Post-layout gate-level netlist
├── screenshots/                    # Die layout captures and architectural schematics
└── synth_yosys/                    # Standalone Yosys synthesis flow & gate-level netlist
```

---

## 🛠️ Verification & ASIC Synthesis Flow

### 1. Functional Simulation (Icarus Verilog):
```bash
# Compile SoC core and peripheral testbench
iverilog -o sim/mcu_sim.vvp rtl/*.v tb/tb_top.v

# Execute simulation
vvp sim/mcu_sim.vvp

# Open waveforms in GTKWave
gtkwave sim/waveform.vcd
```

### 2. Standalone Logic Synthesis (Yosys):
```bash
cd synth_yosys
yosys synth.ys
```

### 3. OpenLane ASIC Implementation (RTL-to-GDSII):
```bash
# Run automated OpenLane flow targeting SkyWater 130nm
openlane config.json
```
