
# SRAM Array 4x8

## Introduction

The 4x8 SRAM Array is a static memory matrix with 4 rows (wordlines) and 8 columns (bitlines). This structure is built from the basic memory cells **bitcell_6t** and **bitcell_6t_full** previously designed. Each memory cell stores 1 bit of data. The 4x8 SRAM array allows sequential write and read operations on each data column.

<img width="1790" height="596" alt="image" src="https://github.com/user-attachments/assets/93723299-e9d6-44f1-a6ad-5603c5c56191" />


## SRAM Array Structure

The 4x8 SRAM array consists of:

- **4 wordlines (WL1, WL2, WL3, WL4)**: control access to each row of memory cells.
- **8 bitline pairs (BL/BLB)**: transmit read/write data for each column.
- **8 Precharge circuits**: precharge bitlines before reading.
- **8 Sense Amplifiers**: amplify differential signals from bitlines.
- **8 Write Drivers**: control writing data onto bitlines.


## Operating Principle of the SRAM Array 4x8

The 4x8 SRAM array operates based on the coordination of the following control signals:

- **Wordlines (WL1..WL4)**: Each wordline selects one row of memory cells. When WL is high, the access transistors in the memory cells of that row are turned on, connecting the bitlines to the storage nodes Q/QB.

- **Precharge**: Before each read cycle, all bitline pairs are precharged to VDD2 (0.9V) during the first 10ns of each cycle. This ensures that the bitlines are ready for the read operation.

- **Sense Amplifier**: After precharge ends, the sense amplifier is activated to amplify the very small voltage difference between the bitline pair (BL and BLB) into a logic 0 or 1 level.

- **Write Driver**: During the write phase (first 640ns), the write driver writes data from Data_in onto the bitlines, thereby writing into the memory cell selected by WL.

- **Read**: After the write phase (from 640ns onward), read is activated to output data from the bitlines to Data_out through the sense amplifier.

- **Data_in (Input Data)**: This is a digital signal (0V or 1.8V) that provides the data to be written into the SRAM array. Each column has its own Data_in signal (Data_in1..Data_in8). Data is only written into the memory cell when the corresponding Write signal is high. 

The entire operation is synchronized with a cycle time of `T_cycle = 20ns`. Each column operates independently with its own set of Pre_Charge, Sense, Write, and Read signals.


## Design Flow (Sky130)

### 1. Schematic (xschem)

The user draws the schematic `array_4x8.sch` in the xschem software, using the two symbols `bitcell_6t.sym` and `bitcell_6t_full.sym` created in the previous exercise. Refer to the `array_4x8.pdf` file for connection details between blocks.

<img width="1790" height="596" alt="image" src="https://github.com/user-attachments/assets/3d03efd5-8b2c-473e-8987-8c1161f462a2" />



### Creating the Testbench Symbol (code_show.sym)

Create the `code_show.sym` symbol with testbench content including:

- sky130 library declaration.
- Pulse-type stimulus sources for WL, Pre_Charge, Sense, Write, Read, Data_in.
- Time parameters: `T_cycle = 20n`, `T_row = 160n`, `T_data = 20n`.
- Initial conditions for Q nodes (initial logic 0).
- Simulation configuration: `.tran 0.1n 900n`.


<img width="814" height="747" alt="image" src="https://github.com/user-attachments/assets/928c3120-52da-4164-951c-96c888a01de1" />


### Schematic Simulation Waveforms

The simulation waveforms include the following signals:

- Write1 Write2 Write3 Write4 Write5 Write6 Write7 Write8
- Read1 Read2 Read3 Read4 Read5 Read6 Read7 Read8
- WL1 WL2 WL3 WL4
- Data_out1 Data_out2 Data_out3 Data_out4 Data_out5 Data_out6 Data_out7 Data_out8

Results show that the SRAM array functions correctly:

- During the first 640ns (write phase): Data from Data_in is written to the corresponding memory cells.
- After 640ns (read phase): Data is read out on Data_out.


<img width="975" height="471" alt="image" src="https://github.com/user-attachments/assets/eaa09996-59ef-45c2-8c6a-6d5c96fc836e" />


### 2. Layout and DRC (magic)

Use the `getcell` command in magic to reuse the two previously designed layout files:

- `bitcell_6t.mag`
- `bitcell_6t_full.mag`

Arrange the memory cells into a 4x8 array. Connect the WL, BL, BLB, VDD, GND, VDD2 lines and the control signals Pre_Charge, Sense, Write, Read.

<img width="975" height="836" alt="image" src="https://github.com/user-attachments/assets/d57c31a2-6483-4338-bd63-d342f2d3d05b" />


<img width="975" height="861" alt="image" src="https://github.com/user-attachments/assets/639d9ddb-cf91-43cc-848b-d20f5d9d92d7" />


### List of Layout Ports

A total of 87 ports, including:

- Power supplies: VDD, GND, VDD2
- Wordlines: WL1 WL2 WL3 WL4
- Pre_Charge: 8 signals
- Sense: 8 signals
- Write: 8 signals
- Read: 8 signals
- Data_in: 8 signals
- Data_out: 8 signals
- Q nodes of each memory cell (Q1_1..Q4_8)

<img width="986" height="810" alt="image" src="https://github.com/user-attachments/assets/1378a23d-8caa-4cd8-8dbe-96b16b449387" />


## Extracting Netlist from Layout (magic)

Execute the following commands in magic:

```tcl
extract do local
extract all
ext2spice lvs
ext2spice
```

The result creates the file `Array_4x8_non_hiera.spice` containing all layout connection information.

## Testbench for Layout

Create the file `tb_Array_4x8_non_hiera.spice` with the following content:

- `.lib` sky130
- `.include array_4x8.spice`
- VDD1 = 1.8V, VDD2 = 0.9V sources
- Stimulus sources B_wl1..B_wl4, B_charge1..B_charge8, B_sen1..B_sen8, B_write1..B_write8, B_read1..B_read8, B_data1..B_data8 (same as schematic testbench)
- Initial conditions for Q nodes
- Simulation: `.tran 0.1n 900n`

The layout simulation results match the schematic simulation results, confirming correct layout operation.

<img width="975" height="471" alt="image" src="https://github.com/user-attachments/assets/37437291-61ca-4330-b836-32c0fa599575" />


### 3. LVS (netgen) – Layout vs Schematic

#### Preparation

Create the `LVS array_4x8` directory. Copy 2 files into the directory:

- `Array_4x8_non_hiera_magic.spice` (from magic)
- `Array_4x8_non_hiera_xschem.spice` (from xschem)

#### Run LVS

```bash
netgen -batch lvs \
  "Array_4x8_non_hiera_magic.spice Array_4x8_non_hiera" \
  "Array_4x8_non_hiera_xschem.spice Array_4x8_non_hiera" \
  sky130A_setup.tcl \
  lvs_Array_4x8_non_hiera.log
```

#### Result

<img width="1846" height="848" alt="image" src="https://github.com/user-attachments/assets/4f74dc44-cf9e-4f97-9acc-bf616a3942e1" />

### ✅ Expected result: Circuits match uniquely.

<img width="803" height="230" alt="image" src="https://github.com/user-attachments/assets/6389533e-6419-40ef-b8bb-4fc591c3dc60" />

### This result confirms that the layout and schematic are completely equivalent in terms of connectivity.

### 4. Parasitic Extraction (magic)
Extract parasitics for post-layout simulation:

```bash
extract all
ext2spice hierarchy on
ext2spice scale off
ext2spice cthresh 0
ext2spice rthresh 0
ext2spice -d -o postlayout_Array_4x8_non_hiera.spice -f ngspice
```

### 5. Post-Layout Simulation (ngspice)
Run testbench with:

.lib sky130.lib.spice tt

.include postlayout_Array_4x8_non_hiera.spice
Compare waveforms with pre-layout to evaluate parasitic effects.

<img width="975" height="471" alt="image" src="https://github.com/user-attachments/assets/5908decb-7f3b-49ef-8f97-68e8f42f7c28" />


### 6. Export GDS (magic)
After all checks pass: gds write bitcell_full.gds

<img width="589" height="467" alt="image" src="https://github.com/user-attachments/assets/9fe287b2-7084-4ea3-9576-7ca0d1283ec8" />


---

### 7. Digital Synthesis with Yosys

After completing and verifying that the analog SRAM macro (Array_4x8) operates correctly, the next step is to build the digital control logic (Row Decoder, Controller) and connect them to the macro as a complete `top_sram_tile` block.

The complete RTL includes:
- `row_decoder.v`: Decodes 2-bit address into 4 wordlines.
- `sram_ctrl.v`: FSM generating Precharge, Write, Read, Sense control pulses.
- `sram_macro.v`: Blackbox declaring the ports of the analog macro.
- `top_sram_tile.v`: Wrapper connecting all the above blocks.

Synthesis is performed with **Yosys** using the script `synth_top_sram_4x8.ys`. This tool converts RTL into a gate-level netlist using standard cells from the Sky130 library.

The output is `top_sram_tile_synth.v` – a netlist mapped to standard logic gates, ready for the P&R step.

<img width="975" height="1098" alt="image" src="https://github.com/user-attachments/assets/415b8b39-843a-4d67-8ff6-f21c35fe3df8" />

<img width="975" height="787" alt="image" src="https://github.com/user-attachments/assets/7504d25a-1f56-4dd1-8967-e0800e8e923f" />


### 8. P&R and Routing with OpenROAD

The key difference from the previous version (31.5) is that the **OpenROAD control script now includes the complete routing flow** (detailed wire routing), not just placement and clock tree.

The script `run_sram_4x8.tcl` is enhanced to perform sequentially:

1.  **Load technology and design**: Read LEF files for standard cells and SRAM macro, read post-synthesis netlist.
2.  **Floorplanning**: Define die size (`65x65`), core area, fixed position for SRAM macro (placed at `8.125 11.25`).
3.  **Placement**: Arrange standard cells around the macro with density `0.45`.
4.  **Clock Tree Synthesis (CTS)**: Build clock distribution tree for the `clk` signal.
5.  **Routing (CRITICAL ADDITION)**: This is the newly added step:
    - Configure metal layers for routing (`set_global_routing_layer_adjustment`).
    - **Global route**: Plan the overall wire paths.
    - **Detailed route**: Perform track-based detailed wiring, outputting a DRC report.
6.  **Export results**: Generate `.def` (layout description), `.odb` (database), and final routed `.gds` files.

After successful execution inside the Docker environment, the file `top_sram_tile_final.def` is created, containing complete information about cell locations and the interconnecting wires.


### 9. Viewing the Routed Layout

To visually inspect the P&R and routing results, open the `.def` file in OpenROAD GUI:

```tcl
openroad -gui
read_lef sky130_fd_sc_hd__nom.tlef
read_lef sky130_fd_sc_hd.lef
read_lef SRAM_4x8.lef
read_liberty sky130_fd_sc_hd__tt_025C_1v80.lib
read_def top_sram_tile_final.def
```

The display shows the complete layout:
- **SRAM Macro** sits at the center (black/white).
- **Standard cells** are distributed around it.
- **Metal wires** are fully routed between cells, carrying signals and power.
- **Clock tree** can be visualized via the *Clock Tree Viewer* window.

<img width="975" height="472" alt="image" src="https://github.com/user-attachments/assets/2ed19dbd-5501-4cbd-a275-42f237f1fd22" />

### 10. Generating the Complete GDS with Magic

The `.def` file describes the layout but is not the actual mask file. To obtain the final GDS, use **Magic** to merge all components:

1.  **Load GDS libraries** for standard cells and the analog SRAM macro.
2.  **Load corresponding LEF files**.
3.  **Read the `.def` file** generated by OpenROAD (`top_sram_tile_final.def`).
4.  **Write a single merged GDS file**.


**Execute these commands in Magic's tkcon window:**

```tcl
gds read sky130_fd_sc_hd.gds
gds read array_4x8_pin.gds
lef read sky130_fd_sc_hd__nom.tlef
lef read sky130_fd_sc_hd.lef
lef read SRAM_4x8_fixed.lef
def read top_sram_tile_final.def
gds write top_sram_tile_full_final.gds
save top_sram_tile_full_final.mag
```


**Results:**
- ### `.gds` file: `top_sram_tile_full_final.gds` – complete mask description, ready for tapeout.

<img width="537" height="512" alt="image" src="https://github.com/user-attachments/assets/7394bbe4-30c8-4e77-b24d-c75da0f1eb12" />

- ### `.mag` file: `top_sram_tile_full_final.mag` – Magic layout database for future viewing or editing.

<img width="1009" height="860" alt="image" src="https://github.com/user-attachments/assets/c3df26c9-2511-455d-b07e-0e5314ddc9d0" />


### 11. 3D GDS Visualization with GDS3D and KLayout

#### With KLayout:
```bash
klayout top_sram_tile_full_final.gds
```
→ Displays 2D layout with all layers, zoomable down to individual cells and wires.

<img width="975" height="469" alt="image" src="https://github.com/user-attachments/assets/f88ed862-1bbf-4e2f-84d7-395ab77788ef" />


#### With GDS3D for intuitive 3D view:
```bash
cd GDS3D
linux/GDS3D -p techfiles/sky130water.txt -i gds/top_sram_tile_full_final.gds
```
→ Renders a 3D visualization of the entire chip, clearly distinguishing metal layers, active regions, gates, and routing.

<img width="1013" height="608" alt="image" src="https://github.com/user-attachments/assets/3ebe082f-6ae4-42b0-9268-af14fdde9c9a" />





### Conclusion

By adding **routing** to the OpenROAD script, the complete AMS design flow from RTL to GDS for the 4x8 SRAM is now fully realized:

- **Analog**: Designed bitcell, precharge, sense amp, write driver → layout extracted, LVS successful.
- **Digital**: Wrote RTL for row decoder and controller → synthesized to gate-level netlist.
- **AMS Integration**: Connected digital logic to analog macro using a blackbox wrapper in RTL.
- **Physical Design**: Placement, clock tree, **routing** of standard cells around the macro.
- **Signoff**: Exported complete GDS, verified with 2D/3D layout viewers.

The final product is a **fully functional 4x8 SRAM block, usable as a hard macro in larger systems**, ready for tapeout on the Sky130 process.
