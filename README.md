# Sky130 RTL · Simulation · Yosys Synthesis Lab

A five-day, open-source sprint https://www.vlsisystemdesign.com/ that takes Verilog RTL designs from functional simulation through gate-level netlists on the **SkyWater 130 nm** PDK.

---

## 📅 Workshop Road-map

| Day | 🎯 Focus | 🔬 Signature exercise |
|-----|----------|-----------------------|
| **1** | RTL → sim → synth → GLS | 2-to-1 multiplexer |
| **2** | Timing libs · hierarchy vs flatten · D-FF resets | `multiple_modules`, `dff_*` |
| **3** | Optimisation passes · constant / state / retime | `opt_check`, `dff_const*` |
| **4** | Gate-level pitfalls & race hazards | `bad_mux` ordering caveat |
| **5** | Conditional constructs & latch-avoidance | `if/else`, `case`, `generate` |

---

## 🛠 Toolchain

| Tool | Role | Ubuntu install |
|------|------|----------------|
| Icarus Verilog (`iverilog`) | Compile & run test-benches | `sudo apt install iverilog` |
| GTKWave | View `.vcd` waveforms | `sudo apt install gtkwave` |
| Yosys | Synthesize to Sky130 standard cells | `sudo apt install yosys` |
| Sky130 Liberty | Timing/area library | via SkyWater-PDK |

---

## 🚀 Quick-Start (Day 1 flow)

📌 **Try it:** open a terminal at the repo root and run the commands below.  
They perform RTL simulation, synthesis, and a gate-level check for the Day-1 multiplexer.

```bash
cd Day_1

# 1️⃣ Functional simulation
iverilog -o sim.out rtl/good_mux.v rtl/tb_good_mux.v
vvp sim.out            # generates wave.vcd

# 2️⃣ Synthesis
yosys -s gate/synth.tcl   # writes gate/good_mux_netlist.v

# 3️⃣ Gate-level check
iverilog -o gls.out gate/good_mux_netlist.v rtl/tb_good_mux.v
vvp gls.out
