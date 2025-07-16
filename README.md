# EP5101 SinCos $11µA_{PP}$ and $1V_{PP}$ encoder/glass scale to TTL (RS422 compliance) converter (WIP! - UNTESTED!)
This interface board is designed to convert a encoder signal with $11µA_{PP}$ (peak-peak) or $1V_{PP}$ signal to TTL with a [iC-Haus iC-NV](https://www.ichaus.de/product/ic-nv/) be used with a [Beckhoff EP5101](https://www.beckhoff.com/de-de/produkte/i-o/ethercat-box/epxxxx-industriegehaeuse/ep5xxx-winkel-wegmessung/ep5101-0011.html?) for example in LinuxCNC.

Assembly concept:
![assembly concept](pics/concept_DIN-rail/0_Beckhoff_EP5101_v5_1.png "Concept assembly")
![assembly concept](pics/concept_DIN-rail/0_Gesamt_DinRail02_v1_1.png "Concept assembly")

PCB:\
comming soon...


## ToDo
- 🔲 finished prototype PCB (v0.?)
- 🔲 working conversion
- 🔲 tested intensively with [LS403](docs/Heidenhain-LS-403-LS-403C.pdf) and [EP5101](https://www.beckhoff.com/de-de/produkte/i-o/ethercat-box/epxxxx-industriegehaeuse/ep5xxx-winkel-wegmessung/ep5101-0011.html?)


## Powering, connectors and signals
The board needs no seperate supply voltage and is fully powered by the 5V supply of the EP5101. The green led of the RJ45 connector indicates a working 5V rail.
Supported input signals are $11µA_{PP}$ (standard) and $1V_{PP}$. The output is a TTL signal. With component ```U2``` ([AM26LS31](https://www.ti.com/product/de-de/AM26LS31) - Differential Line Driver) placed a placed 


## Mounting adaptor
There is a DIN-rail mounting adaptor avaliable, which is used to mount a M23 male connector and a EP5101 onto a 35 mm standard rail. You can find the [files to 3D print](https://than.gs/m/1345234) it yourself over on my Thangs-account.


## Config switches
Pretty much everything of the [iC-NV](https://www.ichaus.de/product/ic-nv/) is configurable through config switches and a potentiometer. \
A detailed list of all configurable options will be added as soon as the first PCB layout is finished. 


## Terminal blocks, connectors and pinouts
The shielded RJ45 port has the following pinout (pin-matched with the Heidenhain spec for their 9 pin connectors):

| pin RJ45 | usage/signal |
| :---  | :---      |
| 1     | In_SIN+   |
| 2     | In_SIN-   |
| 3     | 5 V       |
| 4     | 0 V       |
| 5     | In_COS+   |
| 6     | In_COS-   |
| 7     | In_ZERO+  |
| 8     | In_ZERO-  |

The 8 pin male-header J2 has the following pinout:

```
                        odd pins  |  even pins
╔════╦════╗             ----------|------------
║  1 ║  2 ║════════      In_SIN+  |  In_SIN-
╠════╬════╣                       |
║  3 ║  4 ║════════          5 V  |  GND
╠════╬════╣                       |
║  5 ║  6 ║════════      In_COS+  |  In_COS-
╠════╬════╣                       |
║  7 ║  8 ║════════     In_ZERO+  |  In_ZERO-
╠════╬════╣                       |
║  9 ║ 10 ║════════       shield  |  shield
╚════╩════╝
```

## Analog Frontend / Signal Conditioning Stage
After reaching out to iC-Haus' support for some details about the needed analog frontend for $11 µA_{PP}$ sensors, i got some details on how to implement it. A really nice solution which works with $1 V_{PP}$ as well as $11 µA_{PP}$ signals was documented in the [iC-NQC datasheet (page 28)](docs/iC-NQC_datasheet_E2en.pdf). This exact signal conditioning stage was implemented for all three stages. By shorting each ```CHx_JP1``` you can connect the $120 \Omega$ termination resistor to the input side of the input stage and by that set the device to its $1 V_{PP}$ operating mode.

<img src="docs/schematic_frontend_suggestion-3.jpg" width="500">


## Usage/Software
This interface board is meant to be used in combination with the [Beckhoff EP5101 module](hhttps://www.beckhoff.com/de-de/produkte/i-o/ethercat-box/epxxxx-industriegehaeuse/ep5xxx-winkel-wegmessung/ep5101-0011.html?). The EL5021 is then used by the [LinuxCNC software](https://linuxcnc.org/) to read e.g. Heidenhain LS403 glass scales on a Maho MH400E.

Also keep in mind that you need a sufficiently sized counter. For most retrofits a 16 bit wide counter will not cut it. 
Quick example:
```math 
n_{min} = \log_2 \left(\frac{range\_of\_motion}{axis\_resolution}\right) = \log_2 \left(\frac{400 mm}{1 \mu m}\right) = 18.61 bit \approx \textbf{19 bit > 16 bit}
```
In addition to the resolution you you also need to check the cutoff frequency (maximum input frequency) of the encoder input. The ```EP5101-0011``` for example is limited to $4.0 MHz$ (that's really high, especally in fourfold evaluation mode). For a resolution of $1 µm$ the maximum axis speed is:
```math
v_{max\_EC-Input} = \frac{resolution\_of\_sensor}{interpolation\_factor \cdot quadratur\_encoder\_input} \cdot cutoff\_frequency = \\~\\
= {overall\_resolution \cdot cutoff\_frequency} = \\~\\
= \frac{20µm}{5 \cdot 4} \cdot 4.0 MHz = 1µm \cdot 4.0 MHz = 4.0 \frac{m}{s}
```
The same calculation needs to be done with the input side of the IC ```iC-NV```. The cutoff frequency depends on the used gain. For a $gain = 4$ the manual states a $fin_{MAX}=200 kHz$ (for RCLK = VCC).
```math
v_{max\_IC-Input} = resolution\_of\_sensor \cdot cutoff\_frequency_{IC} = \\~\\
= 20µm \cdot 200 kHz = 4.0 \frac{m}{s}
```
The lower value of the two calculated velocities, $v_{max\_EC-Input} = 4.0 \frac{m}{s}$ and $v_{max\_IC-Input} = 4.0 \frac{m}{s}$, is $4.0 \frac{m}{s}$. This represents the maximum permissible speed for the corresponding axis and must not be exceeded!

## Documentation
All docs can be found in the [docs folder](docs/).


## Online Preview / BOM
[Detailed schematics preview](https://kicanvas.org/?github=https%3A%2F%2Fgithub.com%2FPedPEx%2FSinCosEnc-Conv_EP5101) (KiCanvas)

[Online BOM](https://htmlpreview.github.io/?https://raw.githubusercontent.com/PedPEx/SinCosEnc-Conv_EP5101/master/bom/webviewer-BOM.html)


## *Note*
The pictures were rendered with the help of Blender and the [pcb2blender](https://github.com/30350n/pcb2blender) plugin and the HTML BOM was created with [InteractiveHtmlBom](https://github.com/openscopeproject/InteractiveHtmlBom).