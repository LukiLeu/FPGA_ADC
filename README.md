# An FPGA-based 7-ENOB 600 MSample/s ADC without any External Components
This repository contains the VHDL code of the proposed ADC from the paper "An FPGA-based 7-ENOB 600 MSample/s ADC without any External Components" published at the conference FPGA 2021.

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.4016001.svg)](https://doi.org/10.5281/zenodo.4016001)

## VHDL
The VHDL code of the ADC is provided in [VHDL/adc/adc.vhd](VHDL/adc/adc.vhd). The ADC was implemented on a ZCU104 demo board from Xilinx which deploys the Ultrascale+ FPGA XCZU7EV-2FFVC1156.

## Constraints
The following constraints are necessary for the ADC to work:
```
set_property OUTPUT_IMPEDANCE RDRV_60_60 [get_ports inp]
set_property SLEW SLOW [get_ports inp]
```
This sets the output impedance and the slewrate of the pin which is used to create the reference slope.

```
set_false_path -from [get_pins clk_wiz_0/inst/mmcme4_adv_inst/CLKOUT0] -to [get_pins {adc_0/U0/inst_TDC/gen_TDC[*].*.inst_TDC/inst_carryChain/output_buffer[*].*/D}]
set_false_path -from [get_pins clk_wiz_0/inst/mmcme4_adv_inst/CLKOUT1] -to [get_pins {adc_0/U0/inst_TDC/gen_TDC[*].*.inst_TDC/inst_carryChain/output_buffer[*].*/D}]
...
set_false_path -from [get_pins adc_0/U0/inst_TDC/inst_calLength/*/C] -to [get_pins {adc_0/U0/inst_TDC/gen_TDC[*].*.inst_TDC/inst_carryChain/output_buffer[*].*/D}]
set_false_path -from [get_pins adc_0/U0/inst_TDC/inst_inlCorrection/inst_controlFalling/*/C] -to [get_pins {adc_0/U0/inst_TDC/gen_TDC[*].*.inst_TDC/inst_carryChain/output_buffer[*].*/D}]
set_false_path -from [get_pins adc_0/U0/inst_TDC/inst_inlCorrection/inst_controlRising/*/C] -to [get_pins {adc_0/U0/inst_TDC/gen_TDC[*].*.inst_TDC/inst_carryChain/output_buffer[*].*/D}]
```
To achieve the timing with this ADC, it is necessary to set a false path from all signals which are fed into the carry chain to the carry chain FFs.

## DRC error
During placement a DRC error will occur which states that a single ended output buffer is not allowed to drive a differential input buffer. This error can be downgraded to a warning with the following tcl script:
```
set_property SEVERITY WARNING [get_drc_checks REQP-1581]
```
