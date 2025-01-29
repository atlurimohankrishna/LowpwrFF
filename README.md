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
### DUT
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
### Testbench
```systemverilog
// Testbench of a Power-Gated Flip-Flop Design Using SystemVerilog 
 module tb_power_gated_ff;

    // Testbench signals
    reg clk, D, enable, sleep;
    wire Q;

    // Instantiate DUT
    power_gated_flip_flop uut (
        .clk(clk),
        .D(D),
        .enable(enable),
        .sleep(sleep),
        .Q(Q)
    );

    // Clock generation: 10ns period
    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

    // Test stimulus
    initial begin
        // Initialize signals
        D = 0; enable = 1; sleep = 0;

        // Test case 1: Normal operation
        #10 D = 1; #10 D = 0; #10 D = 1;

        // Test case 2: Enter sleep mode
        #10 sleep = 1; enable = 0; #20;

        // Test case 3: Exit sleep mode
        #10 sleep = 0; enable = 1; #20;

        // Test case 4: Randomized inputs
        repeat (5) begin
            #10 D = $random; sleep = $random % 2; enable = $random % 2;
        end

        $finish;
    end

    // Monitor DUT output
    initial begin
        $monitor("Time: %0t | D: %b | enable: %b | sleep: %b | Q: %b", 
                 $time, D, enable, sleep, Q);
    end

endmodule

```
## UVM Environment
### 1. UVM Driver
```systemverilog
     class power_gated_ff_driver extends uvm_driver #(bit);
    virtual power_gated_ff_if vif;  // Virtual interface

    function new(string name, uvm_component parent);
        super.new(name, parent);
    endfunction

    task run_phase(uvm_phase phase);
        forever begin
            vif.D <= $random;  // Random data
            vif.sleep <= $random % 2;  // Random sleep
            vif.enable <= $random % 2;  // Random enable
            @(posedge vif.clk);
        end
    endtask
endclass

```
### 2. UVM Monitor
```systemverilog
    class power_gated_ff_monitor extends uvm_monitor;
    virtual power_gated_ff_if vif;
    uvm_analysis_port #(bit) ap;

    function new(string name, uvm_component parent);
        super.new(name, parent);
        ap = new("ap", this);
    endfunction

    task run_phase(uvm_phase phase);
        forever begin
            @(posedge vif.clk);
            ap.write(vif.Q);  // Send output to scoreboard
        end
    endtask
endclass

```
### 3. UVM Scoreboard
```systemverilog
   class power_gated_ff_scoreboard extends uvm_scoreboard;
    uvm_analysis_export #(bit) ap;

    function new(string name, uvm_component parent);
        super.new(name, parent);
    endfunction

    task run_phase(uvm_phase phase);
        bit expected, observed;

        forever begin
            expected = // Model-based expected behavior
            ap.get(observed);
            if (expected !== observed)
                `uvm_error("Mismatch", $sformatf("Expected: %b, Observed: %b", expected, observed));
        end
    endtask
endclass

```
### 4. UVM Test
```systemverilog
 class power_gated_ff_test extends uvm_test;
    power_gated_ff_env env;

    function new(string name, uvm_component parent);
        super.new(name, parent);
    endfunction

    task run_phase(uvm_phase phase);
        phase.raise_objection(this);
        env = power_gated_ff_env::type_id::create("env");
        env.build_phase(phase);
        env.run_phase(phase);
        phase.drop_objection(this);
    endtask
endclass


```
