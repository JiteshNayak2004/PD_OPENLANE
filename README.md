# PD_OPENLANE
RTL to GDS flow using openlane
<details>
<summary>openlane and sky130 PDK</summary>

### fundametals of chip structure

![image](https://github.com/JiteshNayak2004/PD_OPENLANE/assets/117510555/8ba7b58c-0580-43ec-aaed-bc877bb9ff21)

![image](https://github.com/JiteshNayak2004/PD_OPENLANE/assets/117510555/c2274d58-299e-413e-afb6-d7325d4fc7a7)

![image](https://github.com/JiteshNayak2004/PD_OPENLANE/assets/117510555/eb5c48b1-91c2-4f2f-9df3-e929a357cc5e)


we call the above as a package and not a chip

1. PADS - the ways signal comes inside or goes outside
2. CORE - all the digital logic recides
3. DIE - size of the chip


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

#### Synthesis Strategies
Different strategies can be used to synthesize for the either the least area or the best timing. To analyse this, synthesis exploration utility generates a report showing the effect on delays/timing/area et.,

#### Deign Exploration Utility
This is used to suit the design configuration and generate reports with different metrics to select the best. This is also used for regression testing

#### Design For Test - DFT Insertion
1. in a real chip once fabricated we  need to check for manufacturing defects
2. we cannot test individual blocks in an soc thus we need to design the chip such that it can be tested out later
3. DFT is done using scan chains we do this by modifying the flip flop such that it has extra inputs and we can pass test signals to check the functionality of the flip flop
4. we do this using an open source tool called fault

## Floor Planning 
1. This is done by OpenROAD flow The macros and IPs are placed in the core before proceding further This is called as pre-placement. 
2. Floor planning is done separately for the macros and it is called macro floor planning where in the macros are placed in such a way that they are closer to the inputs/outputs/other macros where more connections are present.
3. To prevent the loading effects de-coupling capacitors are placed so that the logic states are well within the noise margin.
   
## power planning

1. When several blocks tap power from a single source, there is a problem of Voltage Drop at the Vdd and Ground source at the Vss which can again push the logic out of the required noise margin into the undefined state.
2. To mitigate this Vdd and Vss are placed as horizontal and vertical strips in the chip so that the blocks can tap power from the nearest source.
3. in power grid creation
   1. **rings**:vdd and vss rings are formed across core and macro
   2. **stripes**: they carry vdd and vss across the chip
   3. **rails** :connect vdd and vss to the standard cell

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
It involves a series of checks and simulations to confirm that the design is ready for fabrication 
and meets the desired functionality, performance, power, and reliability targets

Design Rule Check (DRC) is performed by Magic
Layout Versus Schematic (LVS) is performed by Netgen

## GDSII Extraction
1. Once the layout is verified and passes all checks, the final step is to generate the GDSII file format which represents the complete physical layout of the chip.
2. The GDSII file contains the geometric information necessary for fabrication, including the shapes, layers, masks, and other relevant details.
3. The routed .def file is used my Magic to generate the GDSII file

## intro to openlane
1. OpenLane is an automated RTL to GDSII flow based on several components including OpenROAD, Yosys, Magic, Netgen, CVC, SPEF-Extractor, KLayout and a number of custom scripts for design exploration and optimization It also provides a number of custom scripts for design exploration and optimization
2. Currently, it supports both A and B variants of the sky130 PDK, and the C variant of the gf180mcu PDK, and instructions to add support for other (including proprietary) PDKs are documented
3. OpenLane abstracts the underlying open source utilities, and allows users to configure all their behavior with just a single configuration file

OpenLane integrated several key open source tools over the execution stages:

1. RTL Synthesis, Technology Mapping, and Formal Verification : yosys + abc
2. Static Timing Analysis: OpenSTA
3. Floor Planning: init_fp, ioPlacer, pdn and tapcell
4. Placement: RePLace (Global), Resizer and OpenPhySyn (formerly), and OpenDP (Detailed)
5. Clock Tree Synthesis: TritonCTS
6. Fill Insertion: OpenDP/filler_placement
7. Routing: FastRoute or CU-GR (formerly) and TritonRoute (Detailed) or DR-CU
8. SPEF Extraction: OpenRCX or SPEF-Extractor (formerly)
9. GDSII Streaming out: Magic and KLayout
10. DRC Checks: Magic and KLayout
11. LVS check: Netgen
12. Antenna Checks: Magic
13. Circuit Validity Checker: CVC
    
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
 
## labwork 

tool files

![p1](https://github.com/JiteshNayak2004/PD_OPENLANE/assets/117510555/84de87c9-bc7b-487d-9cb0-bc21f8e52575)

process files

![Screenshot from 2023-09-14 18-47-36](https://github.com/JiteshNayak2004/PD_OPENLANE/assets/117510555/f2c8e77b-80a9-4e35-b8bf-0313fc2bd8d8)

running openlane
**1. Run OpenLANE:**
 - `$ make mount` = Open the docker platform inside the `openlane/`
 - `% flow.tcl -interactive` = run script for automating the whole RTL to GDSII flow but in step by step `-interactive` mode
 - `% package require openlane 0.9` == retrives all dependencies for running v0.9 of OpenLANE  
 
![Screenshot from 2023-09-14 18-52-02](https://github.com/JiteshNayak2004/PD_OPENLANE/assets/117510555/3a4e6512-b358-4dae-9a8c-d072a004d91c)

design setup stage
- `% prep -design picorv32a` = Setup the filesystem where the OpenLANE tools can dump the outputs. This also creates a `run/` folder inside the specific design directory which contains the command log files, results, and the reports dump by each tools. These folders will be empty for now except for lef files generated by this design setup stage. This merged the [cell LEF files](https://teamvlsi.com/2020/05/lef-lef-file-in-asic-design.html) `.lef` and [technology LEF files](https://teamvlsi.com/2020/05/lef-lef-file-in-asic-design.html) `.tlef` generating `merged.nom.lef` inside `run/tmp/`

![Screenshot from 2023-09-14 18-53-49](https://github.com/JiteshNayak2004/PD_OPENLANE/assets/117510555/94693ab5-7019-4639-a652-2079dfd768b7)

we see runs being executed

![Screenshot from 2023-09-14 18-58-09](https://github.com/JiteshNayak2004/PD_OPENLANE/assets/117510555/d9ef08b5-1b59-4b7a-b787-6e42693d5012)

synthesizing picorv32 on openlane
- `% run_synthesis` = Run yosys RTL synthesis, ABC scripts (for technology mapping), and OpenSTA.  


```
Synthesis Report
=================


   Number of wires:               9824
   Number of wire bits:          10206
   Number of public wires:        1512
   Number of public wire bits:    1894
   Number of memories:               0
   Number of memory bits:            0
   Number of processes:              0
   Number of cells:              10104
     sky130_fd_sc_hd__a2111o_2       2
     sky130_fd_sc_hd__a211o_2      101
     sky130_fd_sc_hd__a211oi_2       4
     sky130_fd_sc_hd__a21bo_2       19
     sky130_fd_sc_hd__a21boi_2       7
     sky130_fd_sc_hd__a21o_2       414
     sky130_fd_sc_hd__a21oi_2      127
     sky130_fd_sc_hd__a221o_2       65
     sky130_fd_sc_hd__a221oi_2       1
     sky130_fd_sc_hd__a22o_2       197
     sky130_fd_sc_hd__a22oi_2        2
     sky130_fd_sc_hd__a2bb2o_2      16
     sky130_fd_sc_hd__a311o_2       38
     sky130_fd_sc_hd__a31o_2        90
     sky130_fd_sc_hd__a31oi_2       10
     sky130_fd_sc_hd__a32o_2        89
     sky130_fd_sc_hd__a41o_2         2
     sky130_fd_sc_hd__and2_2       283
     sky130_fd_sc_hd__and2b_2       32
     sky130_fd_sc_hd__and3_2        77
     sky130_fd_sc_hd__and3b_2       76
     sky130_fd_sc_hd__and4_2        46
     sky130_fd_sc_hd__and4b_2        6
     sky130_fd_sc_hd__and4bb_2       3
     sky130_fd_sc_hd__buf_1       2735
     sky130_fd_sc_hd__buf_2         16
     sky130_fd_sc_hd__conb_1       106
     sky130_fd_sc_hd__dfxtp_2     1596
     sky130_fd_sc_hd__inv_2         83
     sky130_fd_sc_hd__mux2_2      1817
     sky130_fd_sc_hd__mux4_2       323
     sky130_fd_sc_hd__nand2_2      250
     sky130_fd_sc_hd__nand2b_2       2
     sky130_fd_sc_hd__nand3_2       18
     sky130_fd_sc_hd__nand3b_2       3
     sky130_fd_sc_hd__nand4_2        2
     sky130_fd_sc_hd__nor2_2       185
     sky130_fd_sc_hd__nor3_2        11
     sky130_fd_sc_hd__nor3b_2        3
     sky130_fd_sc_hd__nor4_2         4
     sky130_fd_sc_hd__nor4b_2        3
     sky130_fd_sc_hd__o2111a_2       1
     sky130_fd_sc_hd__o211a_2      224
     sky130_fd_sc_hd__o211ai_2       6
     sky130_fd_sc_hd__o21a_2       154
     sky130_fd_sc_hd__o21ai_2       94
     sky130_fd_sc_hd__o21ba_2       15
     sky130_fd_sc_hd__o21bai_2       3
     sky130_fd_sc_hd__o221a_2       19
     sky130_fd_sc_hd__o221ai_2       1
     sky130_fd_sc_hd__o22a_2        26
     sky130_fd_sc_hd__o22ai_2        1
     sky130_fd_sc_hd__o2bb2a_2       7
     sky130_fd_sc_hd__o311a_2       31
     sky130_fd_sc_hd__o311ai_2       2
     sky130_fd_sc_hd__o31a_2        21
     sky130_fd_sc_hd__o31ai_2        2
     sky130_fd_sc_hd__o32a_2        14
     sky130_fd_sc_hd__o41a_2         1
     sky130_fd_sc_hd__or2_2        337
     sky130_fd_sc_hd__or2b_2        20
     sky130_fd_sc_hd__or3_2        102
     sky130_fd_sc_hd__or3b_2        17
     sky130_fd_sc_hd__or4_2         29
     sky130_fd_sc_hd__or4b_2         6
     sky130_fd_sc_hd__xnor2_2       78
     sky130_fd_sc_hd__xor2_2        29

   Chip area for module '\picorv32': 102957.494400

```

```
Flop ratio = Number of D Flip flops = 1596  = 0.1579
             ______________________   _____
             Total Number of cells    10104
```


After running synthesis, inside the `runs/[date]/results/synthesis` is `picorv32a_synthesis.v` which is the mapping of the netlist to standard cell library using ABC. The `runs/[date]/reports/synthesis` will contain synthesis statistic reports and static timing analysis reports. The `runs/[date]/synthesis/logs` contains log files for the terminal output dumps for running yosys and OpenSTA.
</details>






<details>
 <summary>Good Floorplan vs Bad Floorplan and intro to library cells </summary>
 
## floor plan stage

### Find height and width of core and die.
Core is where the logic blocks are placed and this seats at the center of the die. The width and height depends on dimensions of each standard cells on the netlist. Utilization factor is (area occupied by netlist)/(total area of the core). In practical scenario, utilization factor is 0.5 to 0.6. This is space occupied by netlist only, the remaining space is for routing and more additional cells. Aspect ratio is (height)/(width) of core, so only aspect ratio of 1 will produce a square core shape.

### Define location of Preplaced Cell.

1. These are reusable complex logicblocks or modules or IPs or macros that is already implemented (memory, clock-gating cell, mux, comparator...) . The placement on the core is user-defined and must be done before placement and routing (thus preplaced cells). The automated place and route tools will not be able to touch and move these preplaced cells so this must be very well defined

2. Surround preplaced cells with decoupling capacitors.The complex preplaced logicblock requires a high amount of current from the powersource for current switching. But since there is a distance between the main powersource and the logicblock, there will be voltage drop due to the resistance and inductance of the wire. This might cause the voltage at the logicblock to be not within the noise margin range anymore (logic is unstable). The solution is to use decoupling capacitors near the logic block, this capacitor will send enough current needed by the logicblock to switch within the noise margin range.

#### decoupling capacitors
1. The purpose of the decoupling capacitor is to charge the circuit. When a switching activity occurs, the decoupling capacitor transfers some of its charge to the circuit.
2. Decoupling capacitors are large capacitors that store electrical charge. They have a voltage across them similar to that of the power supply
3. When a circuit switches, the decoupling capacitor acts as a power source for the circuit, effectively isolating it from the main power supply. During switching events, the decoupling capacitor supplies the necessary current to the circuit
4. to minimize voltage drops, these capacitors are positioned in close proximity to the circuit. They ensure that the circuit receives the required current during switching operations. 
5. During periods of no switching activity, the decoupling capacitor replenishes its charge from the power supply
6. decoupling caps also help in  mitigating noise and maintaining consistent voltage for delicate components.
7. As electronic apparatuses operate at elevated frequencies, abrupt shifts in current demands can incite voltage fluctuations and unwanted noise,
   thereby resulting in performance dilemmas and signal deterioration.
9. Decoupling capacitors, akin to a safeguard, establish a local storehouse of electrical charge that can swiftly respond to these fluctuations.
   Essentially, they act as reservoirs, storing and disbursing electrical energy as required, effectively sieving out undesirable noise and voltage oscillations.
   
### power planning
1. Power planning in integrated circuit (IC) design involves the careful consideration and distribution of power and ground connections to ensure proper functionality and performance of the chip.
2. One important aspect of power planning is the placement of multiple ground (GND) and supply voltage (VDD) points throughout the IC layout.
3. The need for multiple GND and VDD points arises due to several reasons by providing multiple GND and VDD points, the power can be distributed more evenly throughout the chip, reducing the chances of voltage drops and improving overall power delivery efficiency.
4. Ground bounce occurs when there are variations in the voltage levels of different GND points due to transient currents. This current flow to the ground creates an inductive effect,
   which causes the ground voltage to rise or fall momentarily which can lead to logic's in ckts to change for a moment
6. Similarly, power supply noise refers to fluctuations in the VDD levels caused by switching events.
7. By strategically placing multiple GND and VDD points, the impact of ground bounce and power supply noise can be minimized, improving circuit performance and reducing the risk of functional failures.


### pin placement
1. Pin placement in physical design is all about how and where we put the input/output pins on a chip or circuit board. 
2. It's important because it affects how well signals move around, how little they get messed up, and how easy it is to build and test the device.
3. We have to think about things like keeping the signals strong, spreading out power evenly, managing heat, and making sure it fits with standard connectors and packaging.
4. When we do this pin placement right, it makes the electronic system more reliable, easier to build, and more user-friendly.

### Logical Cell Placement Blockage
This makes sure that the automated placement and routing tool does not place any cell on the pin locations of the die.

Below are all 6  steps for floor planning:  

![image](https://user-images.githubusercontent.com/87559347/183446309-a0714ec5-0619-4327-bdfe-890c19cc97e0.png)


### Placement Stage:
1. Bind the netlist to a physical cell with real dimensions. The physical cell will come from a library that can provide multiple options for shapes, dimensions, and delay for same cells. 
2. Next is placement of those physical cells to the floorplan. The flip flops must be placed as near as possible to the input and output pins to reduce timing delay. 
3. Optimize placement to maintain signal integrity. This is where we estimate wirelength and capacitance (C=EA/d) and based on that insert repeaters/buffers. The wirelength will form a resistanace which will cause unnecessary voltage drop and a capacitance which will cause a slew rate that might not be permissible for fast current switching of logic gates. The solution to reduce resistance and capacitance is to insert buffers for long routes that will act as intermediary and separate a single long wire to multilple ones. Sometime we also do abutment where logic cells are placed very close to each other (almost zero delay) if it has to run at high frequency (2GHz). Crisscrossing of routes is a normal condition for PnR since we can use separate metal layer (using vias) for crisscrossed path.
4. After placement optimization, We will setup timing analysis using idle clock (zero delay for wires and has no clock buffer related delays) considering we have not yet done CTS.   

The goal of placement is not yet on timing but on congestion. Also, standard cells are not placed on floorplan stage, it is done on Placement stage. Macros or preplaced cells are the ones placed on floorplan stage.Macros or preplaced cells are placed on floorplan stage.

![image](https://user-images.githubusercontent.com/87559347/183224947-67a29c54-9a18-45a4-bbd1-9132bcebc304.png)  

Placement is done on two stages:
 - Global Placement = placement with no legalizations and goal is to reduce wirelength. It uses Half Perimeter Wirelength (HPWL) reduction model. 
 - Detailed Placement = placement with legalization where the standard cells are placed on stadard rows, abutted, and must have no overlaps    
 


### Lab [Day 2] - Determine Die Area: 

**1. Set configuration variables.** Before running floorplan stage, the configuration variables or switches must be configured first. The configuration variables are on `openlane/configuration`:  

```
.
├── README.md      
├── checkers.tcl
├── cts.tcl
├── floorplan.tcl  
├── general.tcl
├── lvs.tcl
├── placement.tcl
├── routing.tcl
└── synthesis.tcl 

```  

The  `README.md` describes all configuration variables for every stage and the tcl files contain the default OpenLANE settings. All configurations accepted by the current run is on `openlane/designs/picorv32a/runs/config.tcl`. This may come either from (with priority order):
 - PDK specific configuration inside the design folder
 - `config.tcl` inside the design folder
 - System default settings inside `openlane/configurations`

**2. Run floorplan on OpenLane:** `% run floor_plan`

 
**3. Check the results.** The output of this stage is `runs/[date]/results/floorplan/picorv32a.floorplan.def` which is a [design exchange format](https://teamvlsi.com/2020/08/def-file-in-vlsi-design-exchange.html), containing the die area and positions. 
```
...........
DESIGN picorv32a ;
UNITS DISTANCE MICRONS 1000 ;
DIEAREA ( 0 0 ) ( 660685 671405 ) ;
............
```
The die area here is in database units and 1 micron is equivalent to 1000 database units. **Thus area of the die is (660685/1000)microns\*(671405/1000)microns = 443587 microns squared.** 

**4. View the layout on magic**. Open def file using `magic`:  

```
magic -T /home/kunalg123/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def
```  
![1](https://github.com/JiteshNayak2004/PD_OPENLANE/assets/117510555/d4f96b63-14f5-4e37-b31a-4df1eab6809c)

To center the view, press "s" to select whole die then press "v" to center the view. Point the cursor to a cell then press "s" to select it, zoom into it by pressing 'z". Type "what" in `tkcon` to display information of selected object. These objects might be IO pin, decap cell, or well taps as shown below.  

![image](https://user-images.githubusercontent.com/87559347/183100900-b3527702-5375-4a4e-ad87-194fce382128.png)
 Useful Magic commands are listed on the [Magic Commands section](https://github.com/AngeloJacobo/OpenLANE-Sky130-Physical-Design-Workshop#magic-commands).

**5 Run placement:** `% run_placement`. This commmand is a wrapper which does global placement (performed by RePlace tool), Optimization (by Resier tool), and detailed placement (by OpenDP tool). It displays hundreds of iterations displaying HPWL and OVFL. The algorithm is said to be converging if the overflow is decreasing. It also checks the legality. 

**6. View the output of this stage**. The output of this stage is `runs/[date]/results/placement/picorv32a.placement.def.` To see actual layout after placement, open def file using `magic`:  

```
magic -T /home/kunalg123/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def
```  

![3](https://github.com/JiteshNayak2004/PD_OPENLANE/assets/117510555/b1c696b7-f87f-4515-af2c-d4da71a1569c)

![4](https://github.com/JiteshNayak2004/PD_OPENLANE/assets/117510555/7c1be3d4-6d90-4200-b79b-721b532d4dd3)

![5](https://github.com/JiteshNayak2004/PD_OPENLANE/assets/117510555/e91690d9-ab46-40d7-bb90-919513648956)


## Libraary Binding and Placement

1. Netlist binding is the process of mapping the logical representation of a digital design (typically described in a HDL onto a library of standard cells.
2. Every component is mapped to a given shape and the working of these are defined in the library.
3. Then all these shapes from each stage of the netlist are placed onto the floorplan in a efficient way so that delay is minimal and the blocks are closer to their  i/p or o/p pins.

![image](https://github.com/JiteshNayak2004/PD_OPENLANE/assets/117510555/ec7863b1-e009-4bb4-8434-52ea8bd4d2f2)

4. from din2 to ff1 we use buffers because in longer distances the resistance and capacitance of the interconnect causes distortion in signal aka signal integrity is lost thus buffers are used that take the signal and replicate it enhanching signal integrity
5. 


### CELL DESIGN AND CHARACETRIZATION FLOWS

Library is a place where we get information about every cell. It has differents cells with different size, functionality,threshold voltages. There is a typical cell design flow steps.

1. Inputs : PDKS(process design kit) : DRC & LVS, SPICE Models, library & user-defined specs.
2. Design Steps :Circuit design, Layout design (Art of layout Euler's path and stick diagram), Extraction of parasitics, Characterization (timing, noise, power).
3. Outputs: CDL (circuit description language), LEF, GDSII, extracted SPICE netlist (.cir), timing, noise and power .lib files

### Standard Cell Characterization Flow
1. characterization provides the essential data needed to accurately model and simulate the behavior of these components
   
A typical standard cell characterization flow that is followed in the industry includes the following steps:

1. Read in the models and tech files
2. Read extracted spice Netlist
3. Recognise behavior of the cells
4. Read the subcircuits
5. Attach power sources
6. Apply stimulus to characterization setup
7. Provide neccesary output capacitance loads
8. Provide neccesary simulation commands

Now all these 8 steps are fed in together as a configuration file to a characterization software called GUNA. This software generates timing, noise, power models.
These .libs are classified as Timing characterization, power characterization and noise characterization.


### TIMING CHARACTERIZATION

In standard cell characterisation, One of the classification of libs is timing characterisation.

#### Timing threshold definitions 
Timing defintion |	Value
-------------- | --------------
slew_low_rise_thr	| 20% value
slew_high_rise_thr | 80% value
slew_low_fall_thr |	20% value
slew_high_fall_thr |	80% value
in_rise_thr	| 50% value
in_fall_thr |	50% value
out_rise_thr |	50% value
out_fall_thr | 50% value

#### Propagation Delay and Transition Time 

**Propagation Delay** 
The time difference between when the transitional input reaches 50% of its final value and when the output reaches 50% of its final value. Poor choice of threshold values lead to negative delay values. Even thought you have taken good threshold values, sometimes depending upon how good or bad the slew, the dealy might be still +ve or -ve.

```
Propagation delay = time(out_thr) - time(in_thr)
```
**Transition Time**

1. The time it takes the signal to move between states is the transition time ,
2. where the time is measured between 10% and 90% or 20% to 80% of the signal levels.

```
Rise transition time = time(slew_high_rise_thr) - time (slew_low_rise_thr)

Low transition time = time(slew_high_fall_thr) - time (slew_low_fall_thr)
```

</details>

<details>
 <summary>Design library cell using Magic Layout and ngspice </summary>

### customizing openlane configuration

1. Configurations on OpenLANE can be changed on the flight.
2. openlane/config has a readme witch description on all switches go to the floorplan section
3. FP_IO_MODE is the mode to set i/p o/p pin placement strategy
4. For example, to change IO_mode to be not equidistant use `% set ::env(FP_IO_MODE) 2;` on OpenLANE.
5. The IO pins will not be equidistant on mode 2 (default of 1).
6. Run floorplan again via `% run_floorplan` and view the def layout on magic.
7. go to `designs/picorv32a/runs/date/result/floorplan` and use the following command
  `magic -T tech file info lef read picorv32a.floorplan.def & `
9. However, changing the configuration on the fly will not change the `runs/config.tcl`, the configuration will only be available on the current session. To echo current value of variable: `echo $::env(FP_IO_MODE)`


### Designing a Library Cell:
1. SPICE deck = component connectivity (basically a netlist) of the CMOS inverter.
2. SPICE deck values = value for W/L (0.375u/0.25u means width is 375nm and lengthis 250nm). PMOS should be wider in width(2x or 3x) than NMOS. The gate and supply voltages are normally a multiple of length (in the example, gate voltage can be 2.5V)  
3. Add nodes to surround each component and name it. This will be used in SPICE to identify a component.    

**Notes:**
 - Width is the length of source and drain. Length is the distance between source and drain
 - PMOS' hole carrier is slower than NMOS' electron carrier mobility, so to match the rise and fall time PMOS must be thicker (less resistance thus higher mobility) than NMOS  
 - A good refresher on MOSFETS and CMOS [is this video](https://www.youtube.com/watch?v=oSrUsM0hoPs) and [this site.](http://courseware.ee.calpoly.edu/~dbraun/courses/ee307/F02/02_Shelley/Section2_BasilShelley.htm)

### SPICE Deck Netlist Description:  

![image](https://user-images.githubusercontent.com/87559347/183240195-608727e5-2d04-4e44-ab4a-2df545cd13ea.png)

**Notes:**
 - Syntax for the PMOS and NMOS descriptiom:
     - `[component name] [drain] [gate] [source] [substrate] [transistor type] W=[width] L=[length]`
 - All components are described based on nodes and its values
 -  cload out 0 10pf means cload is connected b/w node out and 0 and has value 10pf
 - `.op` is the start of SPICE simulation operation where Vin will be sweep from 0 to 2.5 with 0.5 steps
 - `tsmc_025um_model.mod` is the model file containing the technological parameters for the 0.25um NMOS and PMOS
   
The steps to simulate in SPICE:
```
source [filename].cir
run
setplot 
dc1 
plot out vs in 
```  

## SPICE Analysis for Switching Threshold and Propagation Delay:
CMOS is a robust device for logic application as the vtc curve is high for low i/p 
and low for high i/p the robustness of cmos inverter depends on:  

## Switching threshold
1. Vin is equal to Vout. This the point where both PMOS and NMOS is in saturation or kind of turned on, and leakage current is high.
2. If PMOS is thicker than NMOS, the CMOS will have higher switching threshold (1.2V vs 1V) while threshold will be lower when NMOS becomes thicker.
3. It's the point at which this inverter switches between sending out a "0" or a "1" in a computer chip
4. there are two types of tests to analyse cmos inverter behaviour
##### static test
we see how the inverter behaves in a stable state that is
1. how much power it uses
2. how fast it can send a signal
3. how safe it is against errors
#### dynamic test
we see how inverter behaves when it is switching on and off that is
1. how fast can it switch
2. how strong the signals are
3. to catch issues like sudden changes and stuck states

## Propagation delay (rise or fall delay)

DC transfer analysis is used for finding switching threshold. SPICE DC analysis below uses DC input of 2.5V. Simulation operation is DC sweep from 0V to 2.5V by 0.05V steps:
```
Vin in 0 2.5
*** Simulation Command ***
.op
.dc Vin 0 2.5 0.05
```  
Below is the result of SPICE simulation for DC analysis, the line intersection is the switching threshold:  

![image](https://user-images.githubusercontent.com/87559347/187056328-d6f6d5f5-4ce1-4454-9a5a-26be83a84734.png)




Meanwhile, transient analysis is used for finding propagation delay. SPICE transient analysis uses pulse input: 
1. starts at 0V
2. ends at 2.5V
3. starts at time 0
4. rise time of 10ps
5. fall time of 10ps
6. pulse-width of 1ns
7. period of 2ns  

![image](https://user-images.githubusercontent.com/87559347/187055752-dd66feae-f1e7-4b5b-a037-d1a148b01833.png)  

The simulation operation has 10ps step and ends at 4ns:  

```
Vin in 0 0 pulse 0 2.5 0 10p 10p 1n 2n 
*** Simulation Command ***
.op
.tran 10p 4n
```  
Below is the result of SPICE simulation for transient analysis:

![image](https://user-images.githubusercontent.com/87559347/187056370-18949899-a158-4307-96d9-d5c06bbeed66.png)
 
 ### CMOS Fabrication Process (16-Mask CMOS Process):  
 **1. Selecting a substrate** = Layer where the IC is fabricated. Most commonly used is P-type substrate  
 **2. Creating active region for transistor** = Separate the transistor regions using SiO2 as isolation
  - Mask 1 = Covers the photoresist layer that must not be etched away (protects the two transistor active regions)
  - Photoresist layer = Can be etched away via UV light  
  - Si3N4 layer = Protection layer to prevent SiO2 layer to grow during oxidation (oxidation furnace)  
  - SiO2 layer = Grows during oxidation (LOCOS = Local Oxidation of Silicon) and will act as isolation regions between transistors or active regions  
  
![image](https://user-images.githubusercontent.com/87559347/187062659-9e18e9a5-eff4-4d01-804d-cc1e10597486.png)  

 **3. N-Well and P-Well Fabrication** = Fabricate the substrate needed by PMOS (N-Well) and NMOS (P-Well)  
  - Phosporus (5 valence electron) is used to form N-well  
  - Boron (3 valence electron) is used to form P-Well.  
  - Mask 2 protects the N-Well (PMOS side) while P-Well (NMOS side) is being fabricated then Mask 3 while N-Well (PMOS side) is being fabricated
   
![image](https://user-images.githubusercontent.com/87559347/187099587-4a837f08-b6d3-4cb9-afe6-75ee8d88cfff.png) 

 **4. Formation of Gate** = Gate fabrication affects threshold voltage. Factors affecting threshold voltage includes:    
 
![image](https://user-images.githubusercontent.com/87559347/187111068-874f408a-d41b-4b16-a5f0-49edfced8926.png)

Main parameters are:
  - Doping Concentration = Controlled by ion implantation (Mask 4 for Boron implantation in NMOS P-Well and Mask 5 for Arsenic implantation in PMOS N-Well)
  - Oxide capacitance = Controlled by oxide thickness  (SiO2 layer is removed then rebuilt to the desire thickness)  
  
 Mask 6 is for gate formation using polysilicon layer.
 
![image](https://user-images.githubusercontent.com/87559347/187116601-0ac34212-3622-4719-9309-fca887ad995a.png)
**5. Lightly Doped Drain formation** = Before forming the source and drain layer, lightly doped impurity is added: 
 - Mask 7 for N- implantation (lightly doped N-type) for NMOS 
 - Mask 8 for P- implantation (lightly doped P-type) for PMOS.  
Heavily doped impurity (N+ for NMOS and P+ for PMOS) is for the actual source and drain but the lightly doped impurity will help maintain spacing between the source and drain and prevent hot electron effect and short channel effect. 

![image](https://user-images.githubusercontent.com/87559347/187121868-94dfade0-2c63-4c9c-afef-942ef9662d5a.png)
**6. Source and Drain Formation** = Mask 9 is for N+ implantation and Mask 10 for P+ implantation  
 - Channeling is when implantations dig too deep into substrate so add screen oxide before implantation
 - The side-wall spacers maintains the N-/P- while implanting the N+/P+    
 
![image](https://user-images.githubusercontent.com/87559347/187128442-76d48790-53a0-4ad2-9856-924f3efd33eb.png)

**7. Form Contacts and Interconnects** =  TiN is for local interconnections and also for bringing contacts to the top. TiS2 is for the contact to the actual Drain-Gate-Source. Mask 11 is for etching off the TiN interconnect for the first layer contact. 

![image](https://user-images.githubusercontent.com/87559347/187141267-b043152d-0a76-4101-90ec-82c9adcc64e2.png)

**8. Higher Level Metal Formation** = We need to planarize first the layer via CMP before adding a metal interconnect. Aluminum contact is used to connect the lower contact to higher metal layer. Process is repeated until the contact reached the outermost layer.
 - Mask 12 is for first contact hole
 - Mask 13 is for first Aluminum contact layer
 - Mask 14 is for second contact hole
 - Mask 15 is for second Aluminum contact layer. Mask 16 is for making contact to topmost layer. 
 
![image](https://user-images.githubusercontent.com/87559347/187158161-4d230654-5102-4225-8e58-d6d8ed950990.png)

### Layout and Metal Layers:

When polysilicon crosses N-diffusion/P-diffusion (diffusion is also called implantation), then an NMOS/PMOS is created. [Explained here](https://electronics.stackexchange.com/questions/223973/why-diffusions-in-cmos-cad-tool-magic-is-continuous) is the reason why the diffusion layer of source and drain "seems" to be connected under the polysilicon (diffusion layer for source and drain supposedly be separated).


The first layer is local-interconnect layer or local-i then metal 1 to 5. [Here is the process stack diagram](https://skywater-pdk.readthedocs.io/en/main/rules/assumptions.html) of sky130nm PDK. Metal 1 is for Power and Ground lines. `Nsubstratecontact` connects the N-well to locali. `licon` connects the locali to metal1.Locali is for local connections of cells. 

The layer hierarchy for NMOS is: Psubstrate -> Psubstrate Diffusion (psd) -> Psubstrate Contact (psc) -> Local-interconnect (li) -> Mcon -> Metal1. For poly: Poly -> Polycontact -> Locali. P-substrate diffusion an N-substrate diffusion is also referred to as P-tap and N-tap. 

The output of the layout is the LEF file. [LEF (Library Exchange Format)](https://teamvlsi.com/2020/05/lef-lef-file-in-asic-design.html) is used by the router tool in PnR design to get the location of standard cells pins to route them properly. So it is basically the abstract form of layout of a standard cell. `picorv32a/runs/[DATE]/tmp` contains the merged lef files (cell LEF and tech LEF). Notice how metal layer directon (horizontal or vertical) is alternating. Also, metal layer width and thickness is increasing. 

### Magic Commands:  
[Here is a great video guide](https://www.youtube.com/watch?v=RPppaGdjbj0) on layout using Magic. And [here is the Magic website](http://opencircuitdesign.com/magic/) with tutorials.
- Left click = lower-left corner of box  
- Right click = upper-right corner of box  
- "z" = zoom in, "Z" = zoom out, "ctrl + z" = zoom into the box 
- Middle click on empty area will turn the box into empty (similar to erasing it)
- "s" three times will select all geometries electrically connected to each other  
- `:box` = display parameters of selected box  
- `:grid` 0.5um 0.5um = turn on/off and set grid   
- `:snap user` = snap based on current grid  
- `:help snap` = display help for command  
- `:drc style drc(full)` = use all DRC when doing DRC checking
- `:paint poly` = paint "poly" to current box
- `:drc why` = show drc violation inside selected area (white dots are DRC violations )
- `:erase poly` = delete poly inside the box
- `:select area` = select all geometries inside the box
- `:copy n 30` = copy selected geometries to North by 30 grid steps
- `:move n 1` = move selected geometries to North by 1 step ("." to move more, "u" to undo)  
- `: select cell _08555_` = select a particular cell instance (e.g. cell \_08555_ which can be searched in the DEF file)
- `:cellname allcells` = list all cells in the layout
- `:cellname exists sky130_fd_sc_hd__xor3_4` = check if a cell exists 
- `:drc why` = show DRC violation and also the DRC name which can be referenced from [Sky130 PDK Periphery Rules](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#rules-periphery--page-root).

![image](https://user-images.githubusercontent.com/87559347/187588800-f083e5a5-2f22-4670-8a69-93d222794d27.png)


### Lab Part 1 [Day 3] - Slew Rate and Propagation Delay Characterization:

The task is to characterize a sample inverter cell by its slew rate and propagation delay.  

1. Clone [vsdstdcelldesign](https://github.com/nickson-jose/vsdstdcelldesign). Copy the techfile `sky130A.tech` from `pdks/sky130A/libs.tech/magic/` to directory of the cloned repo. Below are the contents of `vsdstdcelldesign/libs/`:


2. View the mag file using magic `magic -T sky130A.tech sky130_inv.mag &`:  the below image is the layout of the inverter
![image](https://github.com/JiteshNayak2004/PD_OPENLANE/assets/117510555/f10fb291-02eb-4896-814e-9638d512d26e)

4. tWe can get to know the details of the inverter by hovering the mouse cursor over it and pressing 's' on the keyboard.
Then we can type **what** in the tkcon terminal
 ![image](https://github.com/JiteshNayak2004/PD_OPENLANE/assets/117510555/ca9955a6-5aae-48a2-8e60-03cf13517b2a)

6. Pressing 's' three times will show what parts are connected to the selected part
7. LEF represents abstract component data in a machine-readable format for IC libraries, while layout is the physical geometric arrangement of these components on a semiconductor chip.
8. DRC errors in magic will be highlighted with white dotted lines:
   ![image](https://github.com/JiteshNayak2004/PD_OPENLANE/assets/117510555/ddf4ee1c-9a2f-43f9-b750-887eb6fedfe0)

 ### extracting spice netlist
 
1. Make an extract file `.ext` by typing `extract all` in the tkon terminal. 
2. Extract the `.spice` file from this ext file by typing `ext2spice cthresh 0 rthresh 0` 
3. cthresh 0 rthresh 0 -> this is done to copy the parasitic capacitances
4. then `ext2spice` in the tkcon terminal.
![image](https://github.com/JiteshNayak2004/PD_OPENLANE/assets/117510555/c893924b-7085-4d70-adce-4079b0f5a4fa)

5. we can now see that sky130_inv.spice file has been created

## sky130 tech file labs

1. let us first find the dimensions of a box in the layout window
2. We can use 'g' on the keyboard to activate the grid and after selecting a grid by right clicking on the mouse, we type box in tkcon window to check the minimum value of the layout window.
   
3. We then modify the spice file to be able to plot a transient response:

```
* SPICE3 file created from sky130_inv.ext - technology: sky130A

.option scale=0.01u
.include ./libs/pshort.lib
.include ./libs/nshort.lib

* .subckt sky130_inv A Y VPWR VGND
M0 Y A VGND VGND nshort_model.0 ad=1435 pd=152 as=1365 ps=148 w=35 l=23
M1 Y A VPWR VPWR pshort_model.0 ad=1443 pd=152 as=1517 ps=156 w=37 l=23
C0 A VPWR 0.08fF
C1 Y VPWR 0.08fF
C2 A Y 0.02fF
C3 Y VGND 0.18fF
C4 VPWR VGND 0.74fF
* .ends

* Power supply 
VDD VPWR 0 3.3V 
VSS VGND 0 0V 

* Input Signal
Va A VGND PULSE(0V 3.3V 0 0.1ns 0.1ns 2ns 4ns)

* Simulation Control
.tran 1n 20n
.control
run
.endc
.end
```  

4. Open the spice file by typing `ngspice sky130A_inv.spice`.
5. Generate a graph using `plot y vs time a` :  

![image](https://github.com/JiteshNayak2004/PD_OPENLANE/assets/117510555/502051a8-3f59-4d7d-bc12-6decc8dde6fe)


Using this transient response, we will now characterize the cell's slew rate and propagation delay:  
- Rise Transition [output transition time from 20%(0.66V) to 80%(2.64V)]:
    - **Tr_r = 2.19981ns - 2.15739ns = 0.04242 ns**  
![image](https://user-images.githubusercontent.com/87559347/188260029-84633ed7-446e-4d1b-b723-12397dcfc71a.png)  


- Fall Transition [output transition time from 80%(2.64V) to 20%(0.66V)]:
   - **Tr_f = 4.0672ns - 4.04007ns = 0.02713ns**   
![image](https://user-images.githubusercontent.com/87559347/188260236-4cc5d4c7-654a-4600-a277-9f6c1df63b11.png)


- Rise Delay [delay between 50%(1.65V) of input to 50%(1.65V) of output]:
   - **D_r = 2.18197ns - 2.15003ns = 0.03194ns**   
![image](https://user-images.githubusercontent.com/87559347/188261194-395c7cfd-caea-4efa-a670-310cb30ff6a2.png)


- Fall Delay [delay between 50%(1.65V) of input to 50%(1.65V) of output]:
   - **D_f = 4.05364ns - 4.05001ns =0.00363ns**  
![image](https://user-images.githubusercontent.com/87559347/188261518-792d3e99-6a5a-423d-9309-62287c608ec0.png)


### Lab Part 2 [Day 3] - Fix Tech File DRC via Magic:
 
Read through [this site about tech file](http://opencircuitdesign.com/magic/techref/maint2.html). All technology-specific information comes from a technology file. This file includes such information as layer types used, electrical connectivity between types, design rules, rules for mask generation, and rules for extracting netlists for circuit simulation. 
Read through also [this site on the DRC rules for SKY130nm PDK](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#rules-periphery--page-root)

1. Download the [lab contents from this site](opencircuitdesign.com/open_pdks/archive/drc_tests.tgz). Extract the tarball. Inside the `drc_tests/` are the `.mag` layout files and the `sky130A.tech`.
2. 
Commands to open magic 
```
magic -d XR
```

Then we open the met3.mag file


![image](https://github.com/Anirudh-Ravi123/pes_pd/assets/142154804/e746a4b8-cd4b-442e-8916-2bc8fdce4c9e)



To check which DRC rule is being violated select area and type drc why in tkcon 


![image](https://github.com/Anirudh-Ravi123/pes_pd/assets/142154804/be24dcd6-9893-4c5a-b2d1-5b2061b9370c)



to add contact cuts 
add met3 contact by selecting area and clicking on m3contact using middle mouse button.
then type ``` cif see VIA2``` in tkcon prompt

![image](https://github.com/Anirudh-Ravi123/pes_pd/assets/142154804/075377c5-4aa4-4583-9656-d0e87d5ab95a)

## fixing drc errors

3. Open magic with `poly.mag` as input: `magic poly.mag`. Focus on `Incorrect poly.9` layout. As described on the poly.9 [design rule of SKY130 PDK](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html#poly), the spacing between polyresistor with poly or diff/tap must at least be 0.480um. Using `:box`, we can see that the distance is 0.250um YET there is no DRC violations shown. Our goal is to fix the tech file to include that DRC.  
![image](https://user-images.githubusercontent.com/87559347/188370620-7e802ce0-cd15-4385-9b73-d8f5ee5fe8ae.png)

4. Open `sky130A.tech`. The included rules for poly.9 are only for the spacing between the n-poly resistor with n-diffusion and the spacing between the p-poly resistor with diffusion. We will now add new rules for the spacing between the **poly resistor with poly non-resistor**, highlighted green below are the two added rules. On the left is the rule for spacing between n-poly resistor with poly non-resistor and on the right is the rule for the spacing between the p-poly resistor with poly non-resistor. The `allpolynonres` is a macro under `alias` section of techfile. 
![image](https://user-images.githubusercontent.com/87559347/188374444-4999b439-40ab-42ae-91cd-91017c217f3e.png)

5. Run `tech load sky130A.tech` then `drc check` in tkcon to reload the tech file. The new DRC rules will now take effect.Notice the white dots on the poly indicating the design rule violations. Command `drc find` to iterate in each violations.  
![image](https://user-images.githubusercontent.com/87559347/188373919-e9d1bd08-7c50-400a-9a17-65fa4296c82e.png)

6. Next, notice below that there are violations between N-substrate diffusion with the polyresistors (from left: npolyres, ppolyres, xpolyres) which is good. But between npolyres with P-substrate diffusion, there is no violation shown. 
![image](https://user-images.githubusercontent.com/87559347/188421029-0f94f6c8-8fc7-4aab-b895-05de5de40f7c.png)

7. To fix that, just modify the tech file to include not only the spacing between npolyres with N-substrate diffusion in poly.9 but between **npolyres and all types of diffusion**. `alldif` is also a macro under `alias` section. Load the tech file again, the new DRC will now take effect.  
![image](https://user-images.githubusercontent.com/87559347/188384339-225f2a84-8aca-44c6-b742-272448051fc9.png)  
![image](https://user-images.githubusercontent.com/87559347/188421488-3d84c048-06b3-46ac-9816-513dd7c721f2.png)

</details>



