# An FPGA-based 7-ENOB 600 MSample/s ADC without any External Components
This repository contains the VHDL code of the proposed ADC from the paper [An FPGA-based 7-ENOB 600 MSample/s ADC without any External Components](https://doi.org/10.1145/3431920.3439287) published at the conference FPGA 2021.

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.4016001.svg)](https://doi.org/10.5281/zenodo.4016001) [![DOI](https://img.shields.io/badge/DOI-10.1145%2F3431920.3439287-blue)](https://doi.org/10.1145/3431920.3439287)

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

## Known limitations
- The delay chain cannot be longer than 1.5 times the period of the clock used to determine the length of the delay chain. In other words, if a delay element has a delay of 4ps and a 600MHz clock is used, the delay chain cannot be longer than 625 delay elements (1250 if the XOR outputs are also used).
- The sum stage of the [transition detector](VHDL/transitionDetector/transitionDetector.vhd) is currently not working for delay chains with less than 512 delay elements. To make it work the range of the integers on [lines 129 to 134](VHDL/transitionDetector/transitionDetector.vhd#L129-134) needs to be increased.
