# OpenLANE-Sky130-Physical-Design-Workshop

# MY NOTES
## Day 1:

Chip is also called package. QFN-48 (Quad Flat No-leads). As the name suggests, it is a flat package, has no external metal leads, and has 12 pins in each 4 sides(so the name "quad") for a total of 48 pins: QFN-48. The chip is much smaller than the actual package and is connected to the external pad usually via wire bonds.

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
