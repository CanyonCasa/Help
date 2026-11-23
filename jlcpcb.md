# JLCPCB Notes

## Schematic

Recommend not changing grid or pin spacings in the schematic, This will lead to potential issues in layout.

## PCB Layout

KiCAD defaults work for basic 2-layer boards. Always run design rule checks before generating Gerbers.

Suggest making board outline with mm grid and place the grid origin in the center of the board or alternately the lower left corner.

When plaing parts best to place all parts using the same grid. Suggest placing parts with a 25 mil grid and routing with a 5 mil grid works for most parts.

Economy double-sided 2-layer boards quote (at this time) $2 for 5 boards 100x100 mm; $5 for 10 boards. Economy board typically take 2-3 days to fab (bare boards, not assembly). Economy shipping runs around $12 and takes 2-4 weeks, depending largely on US customs.

## Assembly

JLCPCB offers low cost PCB assembly (PCBA) with an extensive parts inventory. Parts fall into two classes: Basic and Extended, where Extended parts carry an additional setup fee.

A number of helpful videos ([Designing a PCB in KiCad for JLCPCB Assembly](https://www.youtube.com/watch?v=0RzE770XXYA&t=371s)), toolkits, scripts, and such support JLCPCB assembly in KiCAD. In particular the **'Fabrication Toolkit'** plugin henerates all production files in formats appropriate for JLCPCB assembly, including gerbers, BOM, and pick-n-place placement data.

See [Kicad-klcpcb-example](https://github.com/jvanderberg/Kicad-jlcpcb-example) for more help.

Note, I did not have success using the impartGUI tool with KiCAD 10. Follow the *Adding Custom Parts to KiCAD Libs* instructions below instead.

### Adding Custom Parts to KiCAD Libs

These instructions use a Python script to generate the symbol, footprint, and 3D graphics for KiCAD for use with JLCPCB assembly. 
The script from Nicolas Tousst is available on Github at [TousstNicolas/JLC2KiCad_lib](https://github.com/TousstNicolas/JLC2KiCad_lib). Follow his instructions for installing the script. You may need to add the location to your path.

New parts can be found at [LCSC Electronics](https://lcsc.com) or at [JLCPCB}(https://jlcpcb.com/parts). Note, at JLCPCB parts can be differentiated at Basic or Extended.

1. Open a terminal to your personal KiCAD libs folder (e.g. C:\data\KiCAD\libs)

2. Locate the desired part(s) from one of the repositories listed above. For each part you will need the JLCPCB or LCSC part number, always a alhpanumeric value starting with C (e.g. C2286 for KT-0603R).

3. Run the script given one or more part numbers as parameters, for example:

``` bash
        JLC2KICADLib C8963
```
Note, the script has many other parameters. Run without parameters for help.

This will create files under the folder JLC2KiCad_lib.

4. Open the KiCAD Symbol Editor and select your personal global library such as MyKiCADlib or a project specific library. Goto File Import and locate the symbol under the symbol folder under JLC2KiCad_lib (e.g. C:\data\KiCAD\libs\JLC2KiCad_lib)\symbol). Umport it into the library and save.

    Note, the symbol defaults the footprint property to footprint:... which must be edited to point to your specific library. For example, change footprint:MSOP-10_L3.0-W3.0-P0.50-LS5.0-BL to MyJLCPCB:MSOP-10_L3.0-W3.0-P0.50-LS5.0-BL. Note, some symbols will point to packages in the JLCPCB parts lib such as PCM_JLCPCB:D_0603 and don't need to be changged.

5. For parts referencing your footprint library, similarly open the Footprint Editor and use File Import to add the footprint from the footprint folder and save your library.

    Note: imported footprints default all pins to "Unspecified", which should be changed to the appropriate input, output, power, ...; otherwise, running a board design rule check (DRC) will generate a number of pin warnings.

6. When generating the BOM using JLCPCB tools, first unselect parts in the GUI that you do not want JLCPCB to install. 

#### Assembly Case Study

The [Musketeer.pdf](Musketeer.pdf) contains details of an assembly example. The costs included a number of unexpected fees, but represent reasonable costs. I chose this project acknowledging that I can not physicallly assemble the SMT.

    Caveats... I only had JLCPCB assemble the SMT parts. I specifically chose to not install any throough-hole parts mainly because I already have them and I can do that work. Consequentially, I had the manually selected all parts not placed and remove them from the BOM (i.e. toggle their inclusion in the output). 
    
    Using the toolkit above I had a few problems with the placements. All of the SOT-23 packages were rotated 180d and had to be manually rotated into place.

The order i placed incldues fabrication of 10 boards, 60mmx90mm, with assembly. Here is a rough cost breakdown:

| Item | Breakdown | Cost | Total |
| ----------------------- | ------------------- | ---------: | ---------: |
| 10 PCBs | | | **$5.00** |
| Assembly |
| | Setup Fee | $25.75 |
| | Stencil | $8.27 |
| | Setup Fee | $25.75 |
| | Components | $25.09 |
| | Feeder Loader Fee | $34.10 |
| | SMT Assembly Fee | $3.41 |
| | Packaging Fee | $0.52 |
| | | | **$97.14** |
| Build Time 2-3 days | | | **$49.63** |
|||||
| Merchandise Total | | | **$102.61** |
| Shipping (UPS) | | | **$60.73** |
| Customs duties and taxes | | | **$35.91** |
| Duty and Tax Distribution Fee | | | **$4.53**|
| Sales Tax | | | **$10.11** |
| ORDER TOTAL | | | **$213.89** |

I think there is a small PayPal fee bringing the cost per board to about $21.50. Note, about 1/4 of the cost is taxes and slightly more than a quarter is shipping. The higher value forced a higher shipping cost, estimated at ~$31 be charged as $61.