---
layout: default
title: Using SRAM with OpenMPW
---
# Using SRAM with OpenMPW

If you don't have time to read it, we have prepared a repository that will serve as an example, so please refer to it as appropriate and do your best.
[https://github.com/Cra2yPierr0t/sky130_sram_test](https://github.com/Cra2yPierr0t/sky130_sram_test)

---

I had a **really bad time** when I used self-created memory in OpenMPW. Apparently, you can be happy if you use OpenRAM-generated memory, so here are the results of my various trial and error attempts.

OpenRAM is an OSS that can generate SRAM of any configuration.
[https://openram.org](https://openram.org)

It is a nice OSS, but I am not going to use it directly this time. Or rather, I don't know how to use it. Actually, 1kB and 2kB SRAMs built with OpenRAM are installed together with the installation of SKY130 PDK using Caravel, so we will use them this time.

Verilog and GDSII of SRAM are prepared in the following directory.

```
$PDK_ROOT/sky130B/libs.ref/sky130_sram_macros/
```

This directory provides 32x256, 8x1024 1kB memory and 32x512 2kB memory.

The order in which this memory is used is as follows

1. Place memory in `user_project_wrapper.v`
2. Edit `config.tcl`
3. Specify location in `macro.cfg`
4. Build

The following is an example of how to use 1kB of 32x256 memory. Basically, other memory can be used in the same way, but there are some points to keep in mind for 8x1024 1kB memory, which will be explained later.

## 1. Place memory in `user_project_wrapper.v`

First, set up the memory in `user_project_wrapper.v` as follows. As with other modules, you can name the instances as you like, and you can instantiate multiple instances. You should also write the part enclosed by `ifdef~endif`.

```verilog
    // write    : when web0 = 0, csb0 = 0
    // read     : when web0 = 1, csb0 = 0, maybe 3 clock delay...?
    // read     : when csb1 = 0, maybe 3 clock delay...?
    sky130_sram_1kbyte_1rw1r_32x256_8 rx_mem(
    `ifdef USE_POWER_PINS
        .vccd1  (vccd1          ),
        .vssd1  (vssd1          ),
    `endif
        // RW
        .clk0   (clk0),   // clock
        .csb0   (csb0),   // active low chip select
        .web0   (web0),   // active low write control
        .wmask0 (wmask0), // write mask (4 bit)
        .addr0  (addr0),  // addr (8 bit)
        .din0   (din0),   // data in (32 bit)
        .dout0  (dout0),  // data out (32 bit)
        // R
        .clk1   (clk1),   // clock
        .csb1   (csb1),   // active low chip select
        .addr1  (addr1),  // addr (8 bit)
        .dout1  (dout1)   // data out (32 bit)
    );
```

The role of each IO in the SRAM is described below. The numbers behind are omitted.

| Name | I/O | Width | Role |
| ------- | --- | --- | ------------------------------------------------ |
| `clk` | I | 1bit | Clock input, write and read on falling edge.       |
| `csb` | I | 1bit | Chip select signal bar, 0 for select, 1 for deselect |
| `web` | I | 1bit | Write enable bar; 0 to write, 1 to read |
| `wmask` | I | 4bit (by SRAM) | Byte enable; 8bit unit |
| `addr` | I | 8-bit (by SRAM) | address |
| `din`| I |32bit (by SRAM) | Data Input |
| `dout` | O | 32bit (by SRAM) | data output |

The data write and read methods are as shown in the waveform below, and there is a delay in the readout. Also, the period of `clk` needs to be 20ns or longer.[^1] 

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/MPWRAM/wave.png?raw=true)

~~As for the delay in data readout, it is better to wait a little because the description that causes the delay is written in the SRAM code.~~

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/MPWRAM/arbi.png?raw=true)

I am not familiar with semiconductor design, so I can only guess, but in OpenMPW, mega cells like SRAM can only be instantiated in user_project_wrapper.v.

## 2. Edit `config.tcl`

Once you have connected your module and SRAM, the next step is to edit the config.tcl of the user_project_wrapper. An example of the edited config.tcl can be found here.

[https://github.com/Cra2yPierr0t/Vthernet-SoC/blob/main/openlane/user_project_wrapper/config.tcl](https://github.com/Cra2yPierr0t/Vthernet-SoC/blob/main/openlane/user_project_wrapper/config.tcl)

The four variables to be edited are `FP_PDN_MACRO_HOOKS`, `VERILOG_FILES_BLACKBOX`, `EXTRA_LEFS`, and `EXTRA_GDS_FILES`.

### `FP_PDN_MACRO_HOOKS`

Set `instance name <vdd_net> <gnd_net> <vdd_pin> <gnd_pin>` for `FP_PDN_MACRO_HOOKS`. Basically, write `instance name vccd1 vssd1 vccd1 vssd1`.

```verilog
set ::env(FP_PDN_MACRO_HOOKS) "\
	rx_mem vccd1 vssd1 vccd1 vssd1"
```

If there are multiple instantiations, separate them with a comma and a space as follows. If a space is not placed after the comma, an error will occur.

```verilog
set ::env(FP_PDN_MACRO_HOOKS) "\
	rx_mem1 vccd1 vssd1 vccd1 vssd1, \
	rx_mem2 vccd1 vssd1 vccd1 vssd1"
```

### `VERILOG_FILES_BLACKBOX`

The `VERILOG_FILES_BLACKBOX` is set to the path of the Verilog file in SRAM. This file is not used for synthesis, but is referenced for simulation.

Below is an example of using 1kB of 32x256 memory.

```bash
$::env(PDK_ROOT)/sky130B/libs.ref/sky130_sram_macros/verilog/sky130_sram_1kbyte_1rw1r_32x256_8.v \
```


### `EXTRA_LEFS`

The `EXTRA_LEFS` should be set to the path of the LEF file of the SRAM to be used.

The following is an example of using 1kB of 32x256 memory.

```bash
$::env(PDK_ROOT)/sky130B/libs.ref/sky130_sram_macros/lef/sky130_sram_1kbyte_1rw1r_8x1024_8.lef \
```

### `EXTRA_GDS_FILES`

The `EXTRA_GDS_FILES` should be set to the path of the GDSII file of the SRAM to be used.

The following is an example of using 1kB of 32x256 memory.

```bash
$::env(PDK_ROOT)/sky130B/libs.ref/sky130_sram_macros/gds/sky130_sram_1kbyte_1rw1r_8x1024_8.gds \
```

### DRC error avoidance

Added by suggestion.
Add the following three lines to `config.tcl` to avoid DRC errors caused by magic.

```bash
set ::env(MAGIC_DRC_USE_GDS) 0
set ::env(RUN_MAGIC_DRC) 0
set ::env(QUIT_ON_MAGIC_DRC) 0
````

If you use OpenRAM SRAMs, magic will inevitably generate DRC errors. magic will not do DRC, but will pass the precheck (in most cases).

This completes the minimal editing of `config.tcl`.


## 3. Specify location in `macro.cfg`

Next, use `macro.ctg` to specify the location on the chip where the memory will be placed. The format is `instance name <X coordinate> <Y coordinate> <Orientation>`.
I don't know what Orientation means, but basically it is OK if you set it to N. The unit of the coordinate is μm, and it specifies the position of the lower left corner of the megacell.

Set the path of `macro.cfg` to `MACRO_PLACEMENT_CFG`. This probably already exists in `config.tcl`.

The following is an example of a case where there are multiple instances.

```bash
rx_mem1 100 100 N
rx_mem2 700 100 N
```

The following is the generated layout, with the lower left corner of the memory at (100, 100) as specified.

![](https://raw.githubusercontent.com/Cra2yPierr0t/Cra2yPierr0t.github.io/master/images/MPWRAM/layout.png?raw=true)

The manual says that `MACRO_PLACEMENT_CFG` does not need to be set, but it did not work as far as the author tried.

## 4. Build

Just run the normal Caravel build of `user_project_wrapper`

```bash
make user_project_wrapper
```

If there are no problems, memory should be generated on the layout.

## Notes on using `sky130_sram_1kbyte_1rw1r_8x1024_8`.

At the writing stage (mpw-7a), the value of `NUM_MASKS` must be set to 2, otherwise an error will occur.

The OpenRAM developer said he fixed it, so you may not need it anymore, but try using `sky130_sram_1kbyte_1rw1r_8x1024_8` if the error occurs. If you want to be safe, you should use 32x256.

Here is an example. I made some changes to `NUM_MASKS` and `wmask0`.

```verilog
sky130_sram_1kbyte_1rw1r_8x1024_8 #(.NUM_WMASKS(2)) rx_mem(
    ...
    .web0   (rx_data_vb     ),  // active low write control
    .wmask0 ({wmask0, wmask0}), // write mask (1 bit)
    .addr0  (rx_addr        ),  // addr (10 bit)
    ...
```

## If you want a bigger memory
I want it too.

I think you can generate it using OpenRAM or you can make a module that puts a lot of 1kB SRAM and make it addressable nice.

I want to be able to use OpenRAM properly because I feel that how to create memory in OpenMPW is a matter of life and death.


Please contact @Cra2yPierr0t if you have any comments.

---

[^1]: Thank you Matthew Guthaus-san.
