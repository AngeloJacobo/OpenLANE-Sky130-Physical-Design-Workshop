# OpenLANE-Sky130-Physical-Design-Workshop

# MY NOTES
## Day 1:

The core of the chip will contain two type of blocks:
 - Foundry IP Blocks (e.g. ADC, DAC, PLL, and SRAM) = blocks which requires some amount of intelligent techniques to build which can only be designed by foundries.
 - Macro blocks (e.g. RISC-V SOC and SPI) = pure digital logic blocks compared to IP's which might require some analog parts. 
 
 ![image](https://user-images.githubusercontent.com/87559347/182751377-2810d388-21b0-4df1-b1d4-c72176d80d28.png)

Open Source Digital ASIC Design requires three opensourced components:  
- RTL Designs = github.com, librecores.org, opencores.org
- EDA Tools = OpenROAD, OpenLANE,QFlow  
- PDK = Google + Skywater 130nm Production PDK

PDK (Process Design Kit) = A set of data files and documents which serves as the interface between the designer and the fab. Cell libraries, IO libraries, design rules (DRC, LVS, etc.)

### Simplified RTl to GDSII Flow:
- Sythesis = The RTL is converted into a gate level netlist made up of components of standard cell libary. 
- Floor Planning/ Power Planning = Plan silicon area and create robust power distribution network. The power network usually uses the upper metal layer which are thicker than lower layer and thus lower resistance. This lowers the IR drop problem
 - Placement = There are two steps, first is global placement which is the general optimal positons for cells and might not be legal. Next is detailed placement which is the actual legal placements of the cells.
 - Clock tree synthesis = clock distribution to all flip flops and is usually a tree (H-tree, X-tree ... )
 - Routing = Use horizontal and vertical wires to connect cells together. The router uses PDK information (thickness, pitch, width,vias) for each metal layer to do the routing. The Sky130 defines 6 routing layers. It doe global routing and detailed routing.
 - Verification before sign-off = Involves physical verification like DRC and LVS and timing verification. Design Rule Checking or DRC ensures final layout honors all design rules and Layout versus Schematic or LVS ensures final layout matches the gate level netlist from synthesis phase. Timing verification ensures timing constraints are met.  
 The final layout is in GDSII file format.
 
 [OpenLANE](https://github.com/The-OpenROAD-Project/OpenLane) = Opensourced ASIC developemnt flow rerference. It consists of multiple opensourced tools needed for the whole RTL to GDSII flow. This is tuned epecially for Sky130 PDK. It also works for OSU 130nm. It is recommended to read the OpenLANE documentation before going forward.
 ![image](https://user-images.githubusercontent.com/87559347/182759711-6b9352ec-7652-4589-af31-53a409eb2830.png)

### Notable details are:  
- The input for the whole flow are the rtl files, sdc file, and PDK files. The output is GDSII/LEF file.
- Yosys is used to convert the HDL to gate level netlist using generic components. The [ABC script](http://people.eecs.berkeley.edu/~alanmi/abc/) is then used to map the generic components to the standard cell library of the PDK. These ABC scripts is used to make various synthesis strategies (using the Synthesis Exploration) which can optimize the design either with least area or best timing.  

- The OpenROAD application automates Place and Route  

- The Logic Equivalency CHceking (LEC) is used to compare the resulting netlist after optimization of place and route to the gate level netlist from synthesis phase
Antenna Rules Violation = long wire segments will act as antennna and will accumulate charges, this might damage the connected transistor gates. Solution is to eithre use bridging or antenna diode insertion to leak away the charges  

### Notable Directories:

``` 
├── openlane             -> directory where the tool can be invoked (run docker first)
│   ├── designs          -> All designs must be extracted from this folder
│   │   │   ├── picorv32a -> Design used as case study for this workshop
│   |   |   ├── ...
|   |   ├── ...
├── pdks                 -> contains pdk related files 
│   ├── skywater-pdk     -> all Skywater 130nm PDKs
│   ├── open-pdks        -> contains scripts that makes the PDK (which is normally just compatible to commercial tools) to work with the opensource EDA tool
│   ├── sky130A          -> pdk variant made especially compatible for opensource tools
│   │   │  ├── libs.ref  -> files specific to node process (timing lib, cell lef, tech lef) for example is `sky130_fd_sc_hd` (Sky130nm Foundry Standard Cell High Density)  
│   │   │  ├── libs.tech -> files specific for the tool (klayout,netgenmagic...) 
```

Inside a specfic design folder contains a `config.tcl` which overrides the default settings on OpenLANE. These configurations are specific to a design (e.g. clock period, clock port, verilog files...). The priority order for the OpenLANE settings:
1. sky130_xxxxx_config.tcl
2. config.tcl
3. Default values

### Task for the Day 1 Lab: Find the flipflop ratio percentage for the design `picorv32a`. This is the ratio of the number of flip flops to the total number of cells

## Steps:
1. Run OpenLANE. Below are the commands used in sequence:
 - `$ docker` = Open the docker platform inside the `openlane/`
 - `% flow.tcl -interactive` = run script for automating the whole RTL to GDSII flow but in step by step `-interactive` mode
 - `% package require openlane 0.9` == retrives all dependecies for running v0.9 of OpenLANE  
 
 ![image](https://user-images.githubusercontent.com/87559347/182833010-c5b32449-bfa1-42d0-8433-9edfdefbf1f6.png)

 
2. Design Setup Stage.Command used is:
 - `% prep -design picorv32a` = Setup the filesystem where the OpenLANE tools can dump the outputs. This also creates a `run/` folder inside the specific design directory which contains the command log files, results, and the reports dump by each tools. These folders will be empty for now except for lef files generated by this design setup stage. This the merged the cell lef files `.lef` and technology lef files `.tlef` generating `merged.lef` inside `run/tmp/`
 

![image](https://user-images.githubusercontent.com/87559347/182833339-f117de40-af1f-4607-9c47-81b3ae5f7b7e.png)

3. Run synthesis. Command used is:
 - `% run_synthesis` = Run yosys RTL synthesis, ABC scripts (for technology mapping), and OpenSTA.  
 
![image](https://user-images.githubusercontent.com/87559347/182847715-b0da0f41-b444-4036-9ca3-c5335bfd42cf.png)

### The flipflop ratio is (number of flip flops)/(total number of cells) is 1613/14876 = 0.10843. Or 10.843%

After running synthesis, inside the `runs/[date]/results/synthesis` is `picorv32a_synthesis.v` which is the mapping of the netlist to standard cell library using ABC. The `runs/[date]/reports/synthesis` will contain synthesis statistic reports and static timing analysis reports. The `runs/[date]/synthesis/logs` contains log files for the terminal output dumps for running yosys and OpenSTA.

![image](https://user-images.githubusercontent.com/87559347/182874085-12a7d8b8-d095-4a18-b9ab-101050473046.png)



## Day 2
## Floor Plan General Steps:

#### First step of Physical design flow is to find height and width of core and die.   
Core is where the logic blocks are placed and this seats at the center of the die. The width and height depends on dimensions of each standard cells on the netlist. Utilization factor is (area occupied by netlist)/(total area of the core). In practical scenario, utilization factor is 0.5 to 0.6. This is space occupied by netlist only, the remaining space is for routing and more additional cells. Aspect ratio is (height)/(width) of core, so only aspect ratio of 1 will produce a square core shape.

#### Second step is to define location of Preplaced Cell.   
These are reusable complex logicblocks or modules or IPs or macros that is already implemented (memory, clock-gating cell, mux, comparator...) . The placement on the core is user-defined and must be done before placement and routing (thus preplaced cells). The automated place and route tools will not be able to touch and move these preplaced cells so this must be very well defined

#### Third step is to surround preplaced cells with decoupling capacitors. 
The complex preplaced logicblock requires a high amount of current from the powersource for current switching. But since there is a distance between the main powersource and the logicblock, there will be voltage drop due to the resistance and inductance of the wire. This might cause the voltage at the logicblock to be not within the [noise margin](https://www.electronics-tutorial.net/digital-logic-families/noise-margin/) range anymore (logic is unstable). The solution is to use decoupling capacitors near the logic block, this capacitor will send enough current needed by the logicblock to switch within the noise margin range.

![image](https://user-images.githubusercontent.com/87559347/183011079-5f08eb01-28cf-4617-bcbc-f8f26eacc83d.png)

### Fourth step is Power Planning
Decoupling capactor for sourcing logic blocks with enough current is not feasible to be applied all over the chip but only on the critical elements (preplaced complex logicblocks). Large number of elements switching to logic 0 might cause ground bounce due to large amount of current that needs to be sink at the same time, and switcing to logic 1 might cause voltage droop due to not enough current from the powersource to source needed current of all elements. Ground bounce and voltage droop might cause the voltage to not be within the noise margin range. The solution is to have multiple powersource taps (power mesh) where elements can source current from the nearest VDD and sink current to the nearest VSS tap. This is the reason why most chips have multiple powersource pins.

#### Fifth step is Pin Placement
The input and output ports are placed on the space between the core and the die. The placements of the ports depens on where the cells connected to those ports are placed on the core. The clock port is thicker(least resistance path) than data ports since this clock must be capable to drive the whole chip.

#### Sixt step is Logical Cell Placement Blockage
This makes sure that the automated placement and routing tool does not place any cell on the pin locations of the die.

OPENLANE
Notable details:
Standard cells are not placed on floorplan stage, it is done on Placement stage. Macros or preplaced cells are placed on floorplan stage. Before running floorplan stage, the configuration variables or switches must be configured first. The configuration variables are on `openlane/configuration`:
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
The  `README.md` describes all configuration variables for every stage and the tcl files contain the default OpenLANE settings. All configurations accepted by the current run is on `openlane/designs/picorv32a/runs/config.tcl`. This may come either from (with priorirty order):
1. PDK specific configuration inside the design folder
2. `config.tcl` inside the design folder
3. System default settings inside `openlane/configurations`

Run floorplan on Openlane using `% run floor_plan`. The output of this stage is `runs/[date]/results/floorplan/picorv32a.floorplan.def` which is a design exchange format, containing the die area and positions. 
```
...........
DESIGN picorv32a ;
UNITS DISTANCE MICRONS 1000 ;
DIEAREA ( 0 0 ) ( 660685 671405 ) ;
............
```
The die area here is in database units and 1 micron is equivalent to 1000 database units. Thus (660685/1000)microns\*(671405/1000)microns = 0.443587 micrometer squared. To see actual layout after floorplan, open def file using `magic`:
```
magic -T /home/kunalg123/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def
```
![image](https://user-images.githubusercontent.com/87559347/183226953-8bf8b067-5a70-43a3-9b92-f39caaf02a4a.png)


To center the view, press "s" to select whole die then press "v" to center the view. Point the cursor to a cell then press "s" to select it, zoom into it by pressing 'z". Type "what" in `tkcon` to display information of selected object. These objects might be IO pin, decap cell, or well taps as shown below.
![image](https://user-images.githubusercontent.com/87559347/183100900-b3527702-5375-4a4e-ad87-194fce382128.png)


### Placement and Routing
1. Bind the netlist to a physical cell with real dimensions. The physical cell will come from a library that can provide multiple options for shapes, dimensions, and delay for same cells. 
2. Next is placement of those physical cells to the floorplan. The flip flops must be placed as near as possible to the input and output pins to reduce timing delay

3.Optimize placement to maintain signal integrity. This is where we estimate wirelength and capciatance (C=EA/d) and based on that insert repeaters/buffers. The wirelength will form a resistnace which will cause unnecessary voltage drop and a capacitance which will cause a slew rate that might not be permissible for fast current switching of logic gates. The solution to reduce resistance and capaciatnce is to insert buffers for long routes that will act as intermediary and separate a single long wire to multilple ones.
 Sometime we also do abutment where logic cells are placed very close to each other (almost zero delay) if it has to run at high frequency (2GHz). Crisscrossing of routes is a normal condition for PnR since we can use separate metal layer (using vias) for crisscrossed path.

After placement optimizization, We will setup timing analysis using idle clock (zero delay for wires) considering we have not yet done CTS
![image](https://user-images.githubusercontent.com/87559347/183224947-67a29c54-9a18-45a4-bbd1-9132bcebc304.png)

The goal of placement is not yet on timing but on congestion. 
### LAB
In placement we are less concerned on timing but more on congestion. Placement is done on two stages:
 - Global Placement = placement with no legalizations and goal is to reduce wirelength. It uses Half Perimeter Wirelength (HPWL) reduction model. 
 - Detailed Placement = placement with legalization where the standard cells are placed on stadard rows, abutted, and must have no overlaps  
 
In OpenLANE, un placement using `% run_placement`. This commmand is a wrapper which does global placement (performed by RePlace tool), Optimization (by Resier tool), and detailed placement (by OpenDP tool). It displays hundreds of iterations displaying HPWL and OVFL. The algorithm is said to be converging if the overflow is decreasing. It also checks the legality. 

The output of this stage is `runs/[date]/results/floorplan/picorv32a.floorplan.def.` To see actual layout after floorplan, open def file using `magic`:
```
magic -T /home/kunalg123/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def
```
![image](https://user-images.githubusercontent.com/87559347/183227636-cc5b24b3-5b05-469f-9af1-de89b3c7ed1e.png)  

Power Distibution Network (PDN) is normally created on floorplan stage but on OpenLANE, PDN is done on post-CTS (before routing).  


### Library CHarcterization
Of all RTL-to-GDSII stages, one common thing that the EDA tool always need is data from the library of gates which keeps all standards cells (and, or, buffer gates,...), macros, IPs, decaps, etc. Same cells might have different flavors inside the library (different sizes, delays, threshold voltage). Bigger cell sizes means bigger drive strength to drive longer and thicker wires. Bigger threshold voltage (due to bigger size) will take more time to switch(slower clock) than those with smaller threshold voltage.  

A single cell needs to go through cell design flow. The inputs to make a single cell comes from the foundry Process Design Kits:
 - DRC & LVS Rules = tech files and poly subtrate paramters (CUSTOME LAYOUT COURSE)
 - SPICE Models  = Threshold, linear regions, saturation region equations with added foundry parameters. Including NMOS and PMOS parameteres (Ciruit Deisgn and Spice simulation Course)
 - User defined Spec = Cell height (separation between power and ground rail), Cell width (depends on drive strength), supply voltage, metal layer requirement (which metal layer the cell needs to work)

The library cell developer must adhere to the rules given on the inputs so that when the cell is used on a real design, there will be no errors. Next is design the library cell:
1. Design the circuit function (Output: circuit design language (CDL))
2. Model the pmos and nmos that meets input library requirement
3. Layout the design using Euler's path and sticky diagram to produce best area. This can be done on `magic` layout tool.The outputs are:
   - GDSII (layout file)
   - LEF (defines the width and height of cell)
   - extract spice netlist .cir (parasitics of each element of cell: resistance, capacitance)
 Afte deisgn is characterization using GUNA software, where the outputs are timing, noise, and power characterization.
 .
 ### Timing Characterization (timing .lib)
 
Timing variables for slew. This is two inverters in series, red is output of first inverter and blue is output of second inverter:
![image](https://user-images.githubusercontent.com/87559347/183231913-a9b3826b-5139-4bdc-b12b-3495b87cd8b9.png)

Timing variables for propragation delay. The red is input waveform and blue is output waveform of the buffer. The left side is rise delay and right side is fall delay.  

![image](https://user-images.githubusercontent.com/87559347/183232515-fe3cef76-8a2f-475d-9a64-392fc2fda111.png)

Negative propagation delay is unexpected. That means the output comes before the input so designer needs to choose correct threshold point to produce positive delay. Delay threshdol is susually 50% and slew rate threshold is 20%.

## Day 3

Configurations on OpenLANE canbe changed on the flight. For example, to change IO_mode to be not equidistant, use `% set ::env(FP_IO_MODE) 2;` on OpenLANE. The IO pins will not be equidistant on mode 2 (default of 1). Run floorplan again via `% run_floorplan` and view the def layout on magic. However, changing the configuration on the fly will not change the `runs/config.tcl`, the configuration will only be available on the current session.

### We will make a sample cell (inverter) and insert it to openlane 

Steps to design a CMOS inverter cell.
1. SPICE deck = component connectivity (basically a netlist) of the CMOS inverter.

2. SPICE deck values = W/L (0.375u/0.25u means width is 375nm and length is 250nm). PMOS should be wider in width (2x or 3x) than NMOS ideally. The gate and supply voltages are normally a multiple of length (in the example, gate voltage can be 2.5V)
3. Add nodes to surround each component and name it. This will be used in SPICE to identify a componenet

![image](https://user-images.githubusercontent.com/87559347/183238662-ac9803cc-3246-430e-9172-a1ea681884a1.png)
