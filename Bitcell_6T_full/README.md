# SRAM 6T Full Bitcell (bitcell_6t_full)

## Introduction

The **bitcell_6t_full** is a complete SRAM memory cell including not only the traditional 6T bitcell but also all necessary peripheral circuits for read and write operations:

- **6T Bitcell** (data storage)
- **Precharge / Equalize** (precharge bitlines before read)
- **Write Driver** (force data onto bitlines)
- **Sense Amplifier** (amplify small differential voltage during read)

This full bitcell can be used as a building block for larger SRAM arrays.

---

## Structure

The block consists of:

| Block | Function |
|-------|----------|
| 6T Bitcell | Stores 1 bit (Q, QB) using cross-coupled inverters |
| Precharge | Pulls BL and BLB to VDD before read |
| Write Driver | Drives BL/BLB to 0 or VDD during write |
| Sense Amplifier | Detects small voltage difference between BL and BLB and outputs digital Data_out |

All blocks share the same BL, BLB, WL, and control signals.

---

## Connection Diagram
Inputs:

WL (Word Line)

Data_in (write data)

Sense (enable sense amplifier)

write (enable write driver)

read (enable read output)

Pre_charge (enable precharge)

Outputs:

Data_out (read data)

BL, BLB (bitlines, internal)

Q, QB (internal storage nodes)

Power:

VDD (1.8V for periphery)

VDD2 (0.9V for bitcell)

GND

---

## Basic Operation

| Mode | WL | Pre_charge | write | read | Sense | Result |
|------|----|-------------|-------|------|-------|--------|
| Hold | 0 | 0 | 0 | 0 | 0 | Data retained in bitcell |
| Write | 1 | 0 | 1 | 0 | 0 | Write Driver forces BL/BLB → bitcell updated |
| Read | 1 | 0 | 0 | 1 | 1 | Bitlines precharged, then sense amp enabled → Data_out valid |

---

## Design Flow (Sky130)

### 1. Schematic (xschem)

Schematic includes:
- 6T bitcell (M1..M6)
- Precharge NMOS (M10, M11)
- Write Driver NMOS (M19)
- Sense Amplifier (M9, M12, M13, M14, M15, M16, M17)
- Read output buffer (M18, M21, M23, M24)

**Netlist example** (simplified):
XM1 net1 net3 VDD VDD sky130_fd_pr__pfet_01v8 L=0.15 W=2
XM5 BL WL Q GND sky130_fd_pr__nfet_01v8 L=0.15 W=0.5
XM10 VDD2 pre_charge BL GND sky130_fd_pr__nfet_01v8 L=0.15 W=0.5
XM19 data_in write BL GND sky130_fd_pr__nfet_01v8 L=0.15 W=0.5
...


📌 *Full netlist is provided in the original document.*

### 2. Testbench Schematic (ngspice)

Create `tb_bitcell_6t_full.sch` with:

- Pulse sources for WL, pre_charge, sense, write, read, data_in
- .ic V(BL)=0 V(BLB)=0
- .tran 0.1n 200n

Run simulation and plot all internal nodes.

### 3. Create Symbol (xschem)

From schematic → Insert Symbol → `bitcell_6t_full.sym`

Pins:
WL, Data_in, Sense, BL, Q, QB, VDD, GND, VDD2, Pre_charge, Data_out, BLB, write, read

Re-run testbench using symbol to verify functionality.

### 4. Layout and DRC (magic)

Layout is built from sub-blocks:

- `Pre_Charge.mag`
- `Sense_Amplifier.mag`
- `Sense_Data_out.mag`
- `Bitcell_6t_full.mag` (top cell)

DRC must pass with zero violations.

### 5. Layout Simulation (ngspice)

extract do local
extract all
ext2spice lvs
ext2spice


Run testbench with `.include bitcell_6t_full_magic.spice`
Compare with schematic simulation.

### 6. LVS (netgen) – Layout vs Schematic

Prepare two files:

- `bitcell_6t_full_magic.spice` (from magic)
- `bitcell_6t_full_xschem.spice` (from xschem, edited to match pin order and add VDD/GND globals)

Run:

```bash
netgen -batch lvs \
  "bitcell_6t_full_magic.spice bitcell_6t_full" \
  "bitcell_6t_full_xschem.spice bitcell_6t_full" \
  sky130A_setup.tcl \
  lvs_buffer.log
✅ Expected result:
Circuits match uniquely.

7. Parasitic Extraction (magic)
Extract parasitics for post-layout simulation:

text
extract all
ext2spice hierarchy on
ext2spice scale off
ext2spice cthresh 0
ext2spice rthresh 0
ext2spice -d -o postlayout_bitcell_6t_full.spice -f ngspice
8. Post-Layout Simulation (ngspice)
Run testbench with:

.lib sky130.lib.spice tt
.include postlayout_bitcell_6t_full.spice
Compare waveforms with pre-layout to evaluate parasitic effects.

Simulation Waveforms (Expected)
WL pulses when active

Pre_charge high before read

Sense high during read

Write high during write

Data_in follows write pattern

Data_out valid after sense amp enable

BL/BLB show differential development during read


9. Export GDS (magic)
After all checks pass:

text
gds write bitcell_6t_full.gds

Conclusion
The bitcell_6t_full integrates a full SRAM storage cell with all necessary read/write peripherals. It is fully designed, simulated, and verified in Sky130 technology, ready for use in larger memory arrays.
