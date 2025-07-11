# This repository contains the theory & lab Experiments of RTL design and Synthesis Workshop by vlsisystemdesign.com
# Day_1 :
## Design

The `Design` refers to the actual Verilog code (or set of Verilog modules) that implements the intended functionality according to the specified requirements.

---

## Testbench

A `Testbench` is a simulation environment used to apply test vectors (stimulus) to the design in order to verify its functionality.

---

## How the Simulator Works

- The simulator monitors **input signals** for changes.
- When an input **changes**, the simulator evaluates outputs based on the design logic.
- If there is **no input change**, then **no output update** is triggered.
- The simulator reacts only to **input transitions**, not static inputs.

---

## Simulation Flow

Open-source tools used:

- `Icarus Verilog (iverilog)` – Used to compile and simulate Verilog HDL code.
- `GTKWave` – Used to view waveforms from `.vcd` (Value Change Dump) files.

![image](Day1/media/Simulation_flow.png)
###  Tool Installation

```bash
sudo apt install iverilog
sudo apt install gtkwave
```

###  Compile Design and Testbench

```bash
iverilog -o output.out design.v testbench.v
```

- This creates an executable `output.out`.
- Run the simulation:

```bash
./output.out
```

- It generates a `wave.vcd` file containing signal transitions.

### View Waveform

```bash
gtkwave wave.vcd
```

### Example: Simulation Waveform for 2:1 MUX

![waveform](Day1/media/Waveform_2_1_MUX.png)

---

## Synthesis Flow

- A **synthesizer** converts RTL Verilog code into a **gate-level netlist** using cells from a technology library (`.lib`).
- We use `Yosys` as the synthesis tool.
- The `.lib` file (Standard Cell Library) contains functional, timing, area and power information of logic gates like AND, OR, MUX, INV, DFF, etc.
- The synthesizer maps your logic into these standard cells.

### Timing Constraints

- For a digital circuit to work properly, two key timing constraints need to be satisfied :
  - **Setup Time**
  - **Hold Time**
  
![image](Day1/media/yosys_setup.png)
Slower cells are sometimes needed to fix hold time violations.

###  Tool Installation

```bash
sudo apt install yosys
```

---

## Commands to Synthesize the Design using Yosys

```bash
read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ../verilog_files/good_mux.v
synth -top good_mux
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
show
write_verilog -noattr good_mux_netlist.v
```

###  Command Summary

- ``read_liberty -lib``  
  Loads the Sky130 standard cell library.

- ``read_verilog``  
  Loads your RTL Verilog code.

- ``synth -top good_mux``  
  Performs synthesis; `good_mux` is set as the top module.

- ``abc -liberty``  
  Maps synthesized logic to standard cells from the Sky130 library.

- ``show``  
  Displays a schematic of the synthesized circuit (optional).

- ``write_verilog -noattr``  
  Exports the final gate-level netlist without extra attributes.

---

## Synthesized Schematic: 2:1 MUX

![Schematic](Day1/media/good_mux_netlist.png)

---

##  Resources

All design and testbench files are available in the [repository](https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git).

# Day_2 :
## Timing Libraries

The `.lib` timing library describes the functional, timing, power and area information of standard cells used during synthesis.

The timing library used here is `sky130_fd_sc_hd__tt_025C_1v80.lib`.

-  tt stands for Typical Corner.

**Process corners** describe the range of possible variations in the fabrication process — like how fast or slow transistors switch, depending on tiny variations in doping, dimensions or temperature.

The main standard corners are :

TT (Typical-Typical): Nominal transistor speed.

FF (Fast-Fast): Transistors are faster than nominal.

SS (Slow-Slow): Transistors are slower than nominal.

FS / SF: Mixed — NMOS fast, PMOS slow or vice versa.

### Semiconductor chips are designed and tested to function under various PVT conditions to handle real-world situations.
P — Process ,
V — Voltage , 
T — Temperature

- **025C** means it’s characterized for 25°C,

- **1v80** means a supply voltage of 1.8V.

## Types of Synthesis

### Hierarchical Synthesis
It keeps the module hierarchy intact, just like the RTL design . Each module is synthesized separately and sub-modules are retained as blocks.


### Flattened Synthesis

All modules merged into one netlist for better area and power optimization across modules. It is harder to debug.

`flatten` is the command to do flat synthesis.

Hierarchical synthesis is done for large, modular SoCs, IP-based flows or when reusing verified blocks.

Flattened synthesis is done for maximum performance/area optimization, for small but speed-critical blocks.
- Example :
### Hierarchical Synthesis of multiple_modules.v : 
![hiera](Day2/multiple_modules_hierar.png)

### Flattened Synthesis of multiple_modules.v :
![flat](day2/multiple_modules_flatten.png)
### Glitches in combinational circuits :
In combinational logic:

- Outputs depend on propagation delays through gates.
When inputs change, different paths may have different delays.

- This can cause temporary incorrect outputs (glitches or hazards) before the final stable output settles.

- When Flip-flops are used between combinational circuits it only capture stable values at edge.
Glitches that occur while signals are settling don’t matter .
## Flip-Flop Coding Styles

### Asynchronous Reset D Flip-Flop

```verilog
module dff_asyncres (input clk, input async_reset, input d, output reg q);
  always @(posedge clk or posedge async_reset)
    if (async_reset)
      q <= 1'b0;
    else
      q <= d;
endmodule
```
![dff](Day2/dff_asyncro.png)

This is a D Flip-Flop with an asynchronous reset.

always @(posedge clk or posedge async_reset) means the always block triggers on either:

- the rising edge of the clock (posedge clk), or

- the rising edge of the async reset signal (posedge async_reset).

If async_reset is asserted (1), q is immediately reset to 0, regardless of the clock.

If async_reset is not asserted, the flip-flop samples input d on the rising clock edge and stores it in q.
### Synchronous Reset D Flip-Flop

```verilog
module dff_syncres (input clk, input sync_reset, input d, output reg q);
  always @(posedge clk)
    if (sync_reset)
      q <= 1'b0;
    else
      q <= d;
endmodule
```
![dff](day2/dff_sync.png)
This D Flip-Flop has a synchronous reset.

The always block is only sensitive to posedge clk.

- On the rising edge of the clock.

If sync_reset is 1, q is cleared to 0.

- Otherwise, q takes the value of d.
### Synchronous & Asynchronous Reset D Flip-Flop
```verilog
module dff_asyncres_syncres (input clk,input async_reset,input sync_reset,input d,output reg q);
always @(posedge clk or posedge async_reset)
begin
  if (async_reset)
    q <= 1'b0;      
  else if (sync_reset)
    q <= 1'b0;       
  else
    q <= d;         
end
endmodule
```
![dff](Day2/dff_asyncro_sync.png)

If async_reset is high:

- q is immediately reset to 0.

If async_reset is low, but sync_reset is high on a clock edge:

- On the next rising edge of clk, q is reset to 0 synchronously.

If both resets are low:

- On the next rising clock edge, q takes the value of d.

# Day_3 :

## Design Optimization: Combinational & Sequential Circuits

There are verious techniques to optimize combinational and sequential digital circuits to improve **performance**, **area** and **power efficiency**.

## Constant Propagation

- Replaces variables with constant values during synthesis.
- Simplifies logic and reduces circuit size.
- Improves speed and saves resources.

## State Optimization

- Used for FSMs (Finite State Machines).
- Merges equivalent states to reduce total states.
- Optimizes state encoding for minimal logic.
- Applies Boolean logic to minimize equations.

## Cloning

- Duplicates a logic cell or block to improve performance.
- Balances load, reduces wire delays.
- Helps meet timing closure.

## Retiming

- Shifts flip-flops in a circuit without changing its logic.
- Balances path delays.
- Lowers critical path delay to achieve higher clock speed.
- Maintains functional equivalence.
## Synthesis Flow Example

Use `Yosys` to run optimizations:

```
read_liberty -lib sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog opt_check.v
synth -top opt_check
opt_clean -purge -------------------// command to do the optimization 
abc -liberty sky130_fd_sc_hd__tt_025C_1v80.lib
```

## Example Labs
# Combinational circuits
```verilog
module opt_check (input a , input b , output y);
  assign y = a ? b : 0;
endmodule
```
In this code, the ternary operator (a ? b : 0) is initially synthesized as a **multiplexer**. However, after optimization, constant propagation simplifies the logic, and the expression is effectively reduced to an **AND gate** (y = a & b).
Synthesis Result :
![AND](Day3/opt_check.png)

```verilog
module opt_check2 (input a , input b , output y);
  assign y = a ? 1 : b;
endmodule
```
here also, the ternary operator (a ? 1 : b) is initially synthesized as a **multiplexer**. However, after optimization, constant propagation simplifies the logic, and the expression is effectively reduced to an **OR gate** (y = a | b).

![OR](Day3/opt_check2.png)

### However, the OR gate is typically implemented as a NAND gate with inverters at its inputs. This is because, in a CMOS configuration, an OR gate requires two stacked PMOS transistors in series, which increases resistance and slows down switching. By using NAND gates and inverters instead, the design avoids stacked PMOS and achieves better performance.

```verilog 
module sub_module1(input a , input b , output y);
 assign y = a & b;
endmodule

module sub_module2(input a , input b , output y);
 assign y = a^b;
endmodule

module multiple_module_opt(input a , input b , input c , input d , output y);
wire n1,n2,n3;
sub_module1 U1 (.a(a) , .b(1'b1) , .y(n1));
sub_module2 U2 (.a(n1), .b(1'b0) , .y(n2));
sub_module2 U3 (.a(b), .b(d) , .y(n3));
assign y = c | (b & n1); 
endmodule
```
here the design is 1st flatten the synthesized.

![](Day3/multiple_modules_flat_opt.png)

# Sequential circuits

```verilog
module dff_const1(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
    if(reset)
        q <= 1'b0;
    else
        q <= 1'b1;
end
endmodule
```
The command used to synthesize sequential circuits is ```dfflibmap -liberty < .lib file name >```. Sometimes, **different .lib** files are available for flip-flops, so this command specifies which library the synthesizer should use for flip-flop mapping.

![](Day3/dff1.png)
```verilog
module dff_const2(input clk, input reset, output reg q);
always @(posedge clk, posedge reset)
begin
    if(reset)
        q <= 1'b1;
    else
        q <= 1'b1;
end
endmodule
```
Here the Dff output is always 1 irr of the reset or clk . So it is synthesized without any flip flop.

![](Day3/dff2.png)

### During sequential optimization, any unused outputs (outputs that do not contribute to primary outputs or any logic of interest) are typically removed by the synthesis tool. Keeping unused outputs wastes silicon area and power, and can create unnecessary clock loads. Removing them improves the design’s efficiency.

# Day_4 :
## Gate-Level Simulation (GLS)

**Gate-Level Simulation (GLS)** is an essential step in digital design verification. It ensures that the synthesized netlist works correctly after RTL synthesis.

**Why perform GLS?**

-  **Synthesis Validation:** Verifies that synthesis tools correctly translated RTL to netlist.
-  **Timing Verification:** Checks for timing violations using realistic delays (SDF).
-  **Testability:** Ensures scan chains and other test structures work post-synthesis.

### When to perform GLS?

- **After Synthesis:** When RTL is converted into a gate-level netlist.
- **Before Physical Design:** To catch issues early.

### Types of GLS

- **Functional GLS:** Logic-only simulation (zero/unit delays).
- **Timing GLS:** Uses annotated timing data to check real-world behavior.

## Synthesis-Simulation Mismatch

A **synthesis-simulation mismatch** happens when the RTL simulation results do not match the post-synthesis gate-level simulation .

### Common causes:

-  Non-synthesizable constructs (`initial` blocks, delays).
-  Incomplete/ambiguous RTL (missing `else`, incomplete sensitivity lists).
-  Tool interpretation differences.


### Example : Incomplete sensitivity list :
```verilog
module bad_mux (input i0, input i1, input sel, output reg y);
  always @ (sel) begin
    if (sel)
      y <= i1;
    else 
      y <= i0;
  end
endmodule
```
```verilog
`timescale 1ns / 1ps
module tb_bad_mux;
    reg i0,i1,sel;
    wire y;
    bad_mux uut (
        .sel(sel),
        .i0(i0),
        .i1(i1),
        .y(y)
    );
    initial begin
    $dumpfile("tb_bad_mux.vcd");
    $dumpvars(0,tb_bad_mux);
    sel = 1'b0;
    i0 = 1'b0;
    i1 = 1'b0;
    #300 $finish;
    end
always #75 sel = ~sel;
always #10 i0 = ~i0;
always #55 i1 = ~i1;
endmodule
```
### Simulation  Result :
![](Day4/simulation_mux.png)

### Synthesis Result :
![](Day4/bad_mux.png)
### GLS Result :
![](Day4/GLS_mux.png)

- The first simulation result is incorrect because the output is evaluated only when ```sel``` changes, which causes a mismatch between simulation and synthesis .
- Also the non-blocking assignments in the combinational circuit might create implied latches which leads to glitches .


## Blocking vs. Non-Blocking Assignments in Verilog

Verilog uses two assignment types:

### Blocking (=)

- Executes immediately, sequentially.
- Used for combinational logic (`always @(*)`).

```verilog
always @(*) y = a & b;
```

### Non-Blocking (<=)

- Executes concurrently, scheduled at end of timestep.
- Used for sequential logic (`always @(posedge clk)`).

```verilog
always @(posedge clk) q <= d;
```

### The order of blocking assignments in a combinational always block is crucial, because blocking assignments execute sequentially in the order written. If the order is incorrect, it can produce wrong simulation results, logic mismatches after synthesis or even unintentional latch inference due to incomplete assignment coverage. 

### Example
```verilog
module blocking_caveat (input a, input b, input c, output reg d);
  reg x;
  always @ (*) begin
    d = x & c;
    x = a | b;
  end
endmodule
```
Simulation Result :
![wrong](Day4/Wrong_order.png)
here the order of assignments causes d to use the old value of x not the newly computed value .

Corrected order :
```verilog
always @ (*) begin
  x = a | b;
  d = x & c;
end
```
![wrong](Day4/right_order.png)

# Day_5 :
## If-Else Statements in Verilog

If-else statements allow conditional execution inside procedural blocks such as `always` ,  `initial` , tasks or functions.

### Syntax

```verilog
if (condition) begin
    // Code if condition is true
end else begin
    // Code if condition is false
end
```

- `condition` evaluates to true or false.
- `begin ... end` groups multiple statements.
- `else` is optional.

#### Nested If-Else

```verilog
if (condition1) begin
    // Code if condition1 is true
end else if (condition2) begin
    // Code if condition2 is true
end else begin
    // Code if no conditions are true
end
```

## Inferred Latches in Verilog

1. A latch is inferred when a combinational logic block does not assign a variable on all execution paths.

### Example : Incomplete_if

```verilog
module incomplete_assign (
    input wire a, b, sel,
    output reg y
);
    always @(a, b, sel) begin
        if (sel == 1'b1)
            y = a;  
    end
endmodule
```

**Issue:** When `sel` is 0, `y` is not assigned, so a latch is inferred.

### Synthesis Result :
![](Day5/incomplete_assign..png)

**Solution:** Add an else or default assignment.

```verilog
module incomplete_assign_default (
    input wire a, b, sel,
    output reg y
);
    always @(a, b, sel) begin
        if (sel == 1'b1)
            y = a;  
        else
            y = b;   // Default value when sel is 0
    end
endmodule

```
### Synthesis Result :
![](Day5/incomplete_assign_default.png)


## Case Statements in Verilog

`case` statements allow multi-way branching inside procedural blocks such as `always` ,  `initial` , tasks  or functions.

### Syntax :

```verilog
case (expression)
    value1: begin
        // Code if expression == value1
    end
    value2: begin
        // Code if expression == value2
    end
    default: begin
        // Code if no values match
    end
endcase
```

* `expression` is compared to each `value` in order.
* The `default` branch is optional but recommended to avoid latches.

### Example

```verilog
case (sel)
    2'b00: y = a;
    2'b01: y = b;
    2'b10: y = c;
    default: y = d;  // Executes if no other cases match
endcase
```


* Always cover all possible values or include `default` to prevent unintended latches.


### Example 2 : Incomplete_case
```verilog
module incomp_case (input i0 , input i1 , input i2 , input [1:0] sel, output reg y);
always @ (*)
begin
  case(sel)
    2'b00 : y = i0;
    2'b01 : y = i1;
  endcase
end
endmodule
```
![](Day5/incomplete_case.png)
**Solution :** Add an default assignment.

```verilog
module comp_case (input i0 , input i1 , input i2 , input [1:0] sel, output reg y);
always @ (*)
begin
  case(sel)
    2'b00 : y = i0;
    2'b01 : y = i1;
    default : y = i2;
  endcase
end
endmodule
```
![](Day5/comp_case.png)

## For Loops in Verilog

For loops repeat a block of code multiple times inside procedural blocks. The iteration count must be fixed at compile time.

### Syntax :
```verilog
for (initialization; condition; increment) begin
    // Statements
end
```

## Generate Blocks in Verilog

Generate blocks create hardware at compile time. Commonly used for repetitive structures.

### Example : Creating a 4:1 MUX 
```verilog
module mux_generate (input i0 , input i1, input i2 , input i3 , input [1:0] sel  , output reg y);
wire [3:0] i_int;
assign i_int = {i3,i2,i1,i0};
integer k;
always @ (*)
begin
for(k = 0; k < 4; k=k+1) begin
  if(k == sel)
    y = i_int[k];
end
end
endmodule
```
### Synthesis Result :
![](Day5/mux_generate.png)
### Simulation Result :
![](Day5/generate_block_use.png)

#### This verifies the correct functionality of the generated multiplexer.
