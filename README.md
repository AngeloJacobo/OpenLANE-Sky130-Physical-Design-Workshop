# OpenLANE-Sky130-Physical-Design-Workshop


## Day 1:

Chip is also called package. QFN-48 (Quad Flat No-leads). As the name suggests, it is a flat package, has no external metal leads, and has 12 pins in each 4 sides(so the name "quad") for a total of 48 pins: QFN-48. The chip is much smaller than the actual package and is connected to the external pad usually via wire bonds.

The core of the chip will contain two type of blocks:
 - Foundry IP Blocks (e.g. ADC, DAC, PLL, and SRAM) = blocks which requires some amount of intelligent techniques to build which can only be designed by foundries.
 - Macro blocks (e.g. RISC-V SOC and SPI) = pure digital logic blocks compared to IP's which might require some analog parts. 
 
 ![image](https://user-images.githubusercontent.com/87559347/182751377-2810d388-21b0-4df1-b1d4-c72176d80d28.png)

Open Source Digital ASIC Design requires theree components:
RTL Designs
EDA Tools = OpenROAD, OpenLANE,QFlow
PDK

PDK (Process Design Kit) = A set of data files and documents which serves as the interface between the designer and the fab. Cell libraries, IO libraries, design rules (DRC, LVS, etc.)
