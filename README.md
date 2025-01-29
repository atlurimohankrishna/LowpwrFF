# Verification of a Power-Gated Flip-Flop Design Using SystemVerilog and UVM

## Objective
The goal of this project is to verify the functionality of a power-gated flip-flop, ensuring that it reduces leakage power effectively during idle states without compromising data integrity during active states.

## Overview of Power Gating
Power gating is a design technique used to reduce leakage power in VLSI circuits by shutting off power to inactive blocks or components. It involves adding power switches (usually MOSFETs) to disconnect a block from the supply voltage during idle periods.

### Key Components of Power Gating:
- **Sleep Transistors**: High-Vt transistors used to cut off the power supply during idle states.
- **Retention Elements**: Components like flip-flops or registers that retain state information when power is gated off.
- **Control Logic**: Logic to manage when to activate or deactivate power gating.

## Project Workflow
### 1. Understanding the Design
A power-gated flip-flop consists of:
- A flip-flop to store data.
- A power-gating circuit that includes a sleep transistor to manage power supply.
- Retention circuitry to hold the state during power-down.

### 2. Design Inputs
- **Input Signals**: D (data), clk (clock), enable (power gating control), sleep (control signal for power-down).
- **Outputs**: Q (output data).

### 3. Verification Plan
#### **Verification Goals**
- Ensure the flip-flop stores and outputs the correct data in active mode.
- Verify the leakage current is reduced during the power-down state.
- Check data retention during the sleep state if retention is enabled.

#### **Test Scenarios**
- **Normal Operation**: Verify the flip-flop functions correctly when power gating is not enabled.
- **Power Gating Activation**: Simulate and ensure the power gating works during idle states, and leakage power is reduced.
- **Retention Test**: Verify the retention circuitry holds data correctly during power-off.
- **Wake-up Test**: Confirm the flip-flop resumes normal operation when power is restored.

### 4. Implementation of Testbench
Use **SystemVerilog** and **UVM (Universal Verification Methodology)** for the testbench.

#### **Key Testbench Components**
- **Design Under Test (DUT)**: The power-gated flip-flop.
- **Environment**:
  - **Driver**: Generates input stimulus (data, clock, sleep signals).
  - **Monitor**: Captures and logs output signals for analysis.
  - **Checker**: Verifies that the outputs match expected results.
  - **Scoreboard**: Tracks and compares DUT behavior against a reference model.

## Code Implementation

```verilog
// Verification of a Power-Gated Flip-Flop Design Using SystemVerilog and UVM
`timescale 1ns/1ps

module power_gated_flip_flop (
    input wire D,      // Data input
    input wire clk,    // Clock signal
    input wire enable, // Power gating control
    input wire sleep,  // Sleep mode signal
    output reg Q       // Data output
);
    reg retained_state;

    always @(posedge clk or posedge sleep) begin
        if (sleep) begin
            retained_state <= Q; // Retain state during sleep
        end else if (enable) begin
            Q <= D; // Normal operation
        end
    end
endmodule
```

```systemverilog
// UVM-Based Testbench
import uvm_pkg::*;
`include "uvm_macros.svh"

class flip_flop_test extends uvm_test;
    `uvm_component_utils(flip_flop_test)
    
    function new(string name = "flip_flop_test", uvm_component parent);
        super.new(name, parent);
    endfunction

    virtual function void build_phase(uvm_phase phase);
        super.build_phase(phase);
        // Instantiate UVM environment
    endfunction

    virtual task run_phase(uvm_phase phase);
        phase.raise_objection(this);
        // Implement test scenarios: normal operation, power gating, retention, wake-up
        phase.drop_objection(this);
    endtask
endclass
```

```verilog
// Top-level module for simulation
module testbench;
    reg D, clk, enable, sleep;
    wire Q;
    
    // Instantiate DUT
    power_gated_flip_flop dut (.D(D), .clk(clk), .enable(enable), .sleep(sleep), .Q(Q));
    
    initial begin
        clk = 0;
        forever #5 clk = ~clk; // 10ns clock period
    end
    
    initial begin
        D = 0; enable = 1; sleep = 0;
        #10 D = 1;
        #10 D = 0;
        #10 sleep = 1; // Enter sleep mode
        #20 sleep = 0; // Wake-up
        #10 $finish;
    end
    
    initial begin
        $monitor("Time=%0t | D=%b | clk=%b | enable=%b | sleep=%b | Q=%b", $time, D, clk, enable, sleep, Q);
    end
endmodule
```
