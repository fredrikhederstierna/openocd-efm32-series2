As of June 2021,
support for Silicon Laboratories EFM32 **series 2** MCUs appears not to be present in OpenOCD.
Series 0 and 1 MCUs are supported by the _efm32_ driver, though.

To be able to flash and debug MCUs like [EFM32PG22],
this repository [contains a copy][efm32s2] of [OpenOCD's driver flash/nor/efm32.c][efm32c] for series 0 and 1,
adapted to series 2, as far as necessary for basic functionality.  
There may be bugs,
since parts intended to be used with series 0 and 1 MCUs
might not apply to series 2.
Page locking may not work, this hasn't been tested yet.
Also, it is not possible yet to erase or write to the _userdata_ page.

[efm32s2]: ./efm32s2/efm32s2.c
[efm32c]: https://github.com/ntfreak/openocd/blob/42a0bf3c360c1eae418223f0ab535b4d7accae83/src/flash/nor/efm32.c

A number of details are different in series 2, as compared to series 0 and 1:

-	Device Info registers for page, flash and ram sizes are located at different offsets.
-	MSC `WRITECMD` and `ADDRB` registers are located at different offsets,
	which made an adjustment of the flash write assembly code neccessary.
-	To be able to use the MSC peripheral, its clock must be enabled in the CMU.
-	The RAM starts at a different address, 0x2... instead of 0x1....

Since the work area address (i.e. the start of ram) gets configured in tcl/target/efm32.cfg,
it was decided to create a new `efm32s2` target in tcl/target/efm32s2.cfg,
and to derive a flash driver `efm32s2.c` from `efm32.c`.
It is not clear yet, if it would make sense to integrate series 2 support into the original efm32 driver, instead of keeping it separate.

[EFM32PG22]: https://www.silabs.com/mcu/32-bit/efm32pg22-series-2

## Setup OpenOCD sources

In this project's working directory, clone OpenOCD's official Github mirror repository:

	git clone https://github.com/ntfreak/openocd

Checkout a revision compatible to this EFM32 series 2 extension:

	git checkout 42a0bf3

... which sets the OpenOCD source directory to a few commits after v0.11.0.

Initialize submodules:

	git submodule init
	git submodule update

Copy files and apply the apply the patch from the _efm32s2_ subdirectory to their positions in the openocd directory just created:

	cd openocd
	cp ../efm32s2/efm32s2.c src/flash/nor/
	cp ../efm32s2/efm32s2.cfg tcl/target/
	patch -p1 < ../efm32s2/register_driver.patch 

The OpenOCD tree is now ready to be built.

## Build on Linux

For a minimal setup, with most drivers disabled except CMSIS-DAP,
run

	sh build.sh

If _configure_ complains about missing dependencies,
or CMSIS-DAP is not marked _yes_ in the _configuration summary_,
install the missing pieces, and run build.sh again.

If the build succeeded,
a minimal installation may be created in the _dist_ subdirectory
by running inside this project's working directory:

	sh mkdist.sh

To test whether it works, run `ident.sh` in the dist subdirectory.
A CMSIS-DAP capable device,
connect to the target MCU,
needs to be attached to a local USB port.
The [Particle Debugger], for instance,
has worked for debugging using GDB
and flashing without any problems.

	cd dist
	sh ident.sh

The output should contain lines like:

	Info : detected part: PG22C200, rev 17
	Info : flash size = 256kbytes
	Info : flash page size = 8192bytes

[Particle Debugger]: https://docs.particle.io/datasheets/accessories/debugger/