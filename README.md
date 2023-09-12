# PD_OPENLANE
RTL to GDS flow using openlane
<details>
<summary>openlane and sky130 PDK</summary>
# DAY 1: Inception of Open-source EDA, OpenLane and Sky130 PDK

The core of the chip will contain two types of blocks:
 - **Foundry IP Blocks** (e.g. ADC, DAC, PLL, and SRAM) = blocks which requires some amount of intelligent techniques to build which can only be designed by foundries.
 - **Macro blocks** (e.g. RISC-V SOC and SPI) = pure digital logic blocks compared to IP's which might require some analog parts. 
 
 ![image](https://user-images.githubusercontent.com/87559347/182751377-2810d388-21b0-4df1-b1d4-c72176d80d28.png)

Open Source Digital ASIC Design requires three open-source components:  
- **RTL Designs** = github.com, librecores.org, opencores.org
- **EDA Tools** = OpenROAD, OpenLANE,QFlow  
- **PDK** = Google + Skywater 130nm Production PDK

**PDK (Process Design Kit)** = A set of data files and documents which serves as the interface between the designer and the fab. This includes cell libraries, IO libraries, design rules (DRC, LVS, etc.)

### OPENLANE RTL to GDSII Flow:
## Synthesis
1. The RTL Level Design is then synthesized using a Logic Synthesizer we use yosys for the same here.
2. The RTL Netlist is then converted into a synthesised netlist where there are details about the standard cells and its implementations.
3.  Yosys takes the RTL design and timing .libs and verilog models of standard cells and converts into a RTL Netlist.
4.  abc does the tehnology mapping to the required skywater-pdk variants

1.1 Synthesis Strategies
Different strategies can be used to synthesize for the either the least area or the best timing. To analyse this, synthesis exploration utility generates a report showing the effect on delays/timing/area et.,

1.2 Deign Exploration Utility
This is used to suit the design configuration and generate reports with different metrics to select the best. This is also used for regression testing

1.3 Design For Test - DFT Insertion
1. in a real chip once fabricated we  need to check for manufacturing defects
2. we cannot test individual blocks in an soc thus we need to design the chip such that it can be tested out later
3. DFT is done using scan chains we do this by modifying the flip flop such that it has extra inputs and we can pass test signals to check the functionality of the flip flop

## Floor Planning 
1. This is done by OpenROAD flow The macros and IPs are placed in the core before proceding further This is called as pre-placement. 
2. Floor planning is done separately for the macros and it is called macro floor planning where in the macros are placed in such a way that they are closer to the inputs/outputs/other macros where more connections are present.
3. To prevent the loading effects de-coupling capacitors are placed so that the logic states are well within the noise margin.
   
## power planning

1. When several blocks tap power from a single source, there is a problem of Voltage Drop at the Vdd and Ground source at the Vss which can again push the logic out of the required noise margin into the undefined state.
2. To mitigate this Vdd and Vss are placed as horizontal and vertical strips in the chip so that the blocks can tap power from the nearest source.
3. in power grid creation
   a) first rings are created
   b) then stripes are created
   c) then rails are created
### rings:
vdd and vss rings are formed across core and macro
### stripes:
they carry vdd and vss across the chip
### rails:
connect vdd and vss to the standard cell

## Placement
There are two types of placement. The other required logic is placed optimally. Placement is of two steps

1. Global Placement- finds the optimal position for each cells. These positions are not necessarly correct, cells may overlap
2. Detialed Placement - After Global placement is done minimal alterations are done to correct the issues
4. Clock Tree Synthesis
To ensure minimum skew the Clock is routed optimally through the circuit using different algorithms. This is done in the OpenROAD flow by TritonCTS.

5. Fake Antenna and diode swapping
Long wires acts as antennas and cause accumulation of charges during the fabrication process damaging the transistor. To avoid this bridging is used to pass the wire through different layers or an antenna diode cell is added to leak away the charges

OpenLane approach - Insert Fake Diode to every cell input during placement. This matches the footprint of the library of the antenna diode. The Antenna Checker is run to check for violations, if there are violations then the fake diode is swapped with a real one.
OpenROAD approach - In the global route step, the antenna violation is addressed automatically by inserting an antenan diode OpenLane allows the user to chose either of the above approaches

## Routing
This step is used to implement the interconnect using the different metal layers specified in the PDK. There are two steps

1. Global Routing - This is done inside the OpenROAD flow (FastRoute)
2. Detailed Routing - This is performed using TritonRoute outside the OpenROAD flow after the global routing.
3. Before performing this step the Logic Equivalence Check is performed by Yosys, since OpenROAD does some optimisations the circuit.

## RC Extraction
From the .def file, the parasitic extraction is done to generate the .spef file (Standard Prasitic Exchange Format) which produces an accurate analog model of the circuit by including the parasitic effects due to wires, parasitic capacitances, etc.,

## STA
Static timing analysis (STA) is a method of validating the timing performance of a design by checking all possible paths for timing violations. STA breaks a design down into timing paths, calculates the signal propagation delay along each path, and checks for violations of timing constraints inside the design and at the input/output interface.

## Sign-off Steps
Design Rule Check (DRC) is performed by Magic
Layout Versus Schematic (LVS) is performed by Netgen

## GDSII Extraction
The routed .def file is used my Magic to generate the GDSII file



### OpenLane Directory Hierarchy:

``` 
├── OOpenLane             -> directory where the tool can be invoked (run docker first)
│   ├── designs          -> All designs must be extracted from this folder
│   │   │   ├── picorv32a -> Design used as case study for this workshop
│   |   |   ├── ...
|   |   ├── ...
├── pdks                 -> contains pdk related files 
│   ├── skywater-pdk     -> all Skywater 130nm PDKs
│   ├── open-pdks        -> contains scripts that makes the commerical PDK (which is normally just compatible to commercial tools) to also be compatible with the open-source EDA tool
│   ├── sky130A          -> pdk variant made especially compatible for open-source tools
│   │   │  ├── libs.ref  -> files specific to node process (timing lib, cell lef, tech lef) for example is `sky130_fd_sc_hd` (Sky130nm Foundry Standard Cell High Density)  
│   │   │  ├── libs.tech -> files specific for the tool (klayout,netgen,magic...) 
```

Inside a specific design folder contains a `config.tcl` which overrides the default settings on OpenLANE. These configurations are specific to a design (e.g. clock period, clock port, verilog files...). The priority order for the OpenLANE settings:
1. sky130_xxxxx_config.tcl in `OpenLane/designs/[design]/`
2. config.tcl in `OpenLane/designs/[design]/`
3. Default values in `OpenLane/configuration/`
 

</details>
