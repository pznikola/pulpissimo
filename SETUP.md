# Getting Started

## Install Pulp GCC tool-chain and SDK

First install dependencies (Ubuntu):
```bash
sudo apt-get install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev
```
Clone pulp GCC repository:
```bash
# Location doesn't matter
git clone --recursive https://github.com/pulp-platform/riscv-gnu-toolchain.git
cd riscv-gnu-toolchain
```

### Installation (PULP)
To build the Newlib cross-compiler for all pulp variants, pick an install path. If you choose, say, `/opt/riscv`, then add `/opt/riscv/bin` to your PATH now. In ~/.bashrc you can add:
```bash
# pulpissimo
export PULP_RISCV_GCC_TOOLCHAIN=/opt/riscv
export RISCV=$PULP_RISCV_GCC_TOOLCHAIN
export PATH=$PULP_RISCV_GCC_TOOLCHAIN/bin:$PATH
```
If the `/opt/riscv/` is not the install directory, change `PULP_RISCV_GCC_TOOLCHAIN` accordingly.

Then, simply run the following command (may take a while):
```bash
# You can change installation directory of /opt/riscv to something else
./configure --prefix=/opt/riscv --with-arch=rv32gc_xpulpv3_xcorev --with-abi=ilp32 --enable-multilib
make
```
This will use the multilib support to build the libraries for the various cores (riscy, zeroriscy and so on). The right libraries will be selected depending on which compiler options you use.

### Install build process for the PULP runtime
Set the following environment variable to point to the folder where the pulpissimo was cloned:
```bash
# Change <pulpissimo root folder> to the absolute path of pulpissimo
# You can add this line to the ~/.bashrc
export VSIM_PATH=<pulpissimo root folder>/sim
```
Clone the following repository:
```bash
git clone https://github.com/pulp-platform/pulp-builder.git
cd pulp-builder
git submodule update --init --recursive
```
Install dependencies:
```bash
sudo apt install git python3-pip gawk texinfo libgmp-dev libmpfr-dev libmpc-dev swig3.0 libjpeg-dev lsb-core doxygen sox graphicsmagick-libmagick-dev-compat libsdl2-dev libswitch-perl libftdi1-dev cmake scons libsndfile1-dev
sudo pip3 install twisted prettytable pyelftools openpyxl xlsxwriter pyyaml numpy configparser pyvcd
# If pip2 exists
sudo pip2 install configparser
# Else
sudo pip3 install configparser
```
Now choose the configuration for which you want to compile the runtime, for example:
```bash
# You can check other configurations in folder configs/
source configs/pulpissimo.sh
```
Then execute this script:
```bash
./scripts/clean
./scripts/update-runtime
./scripts/build-runtime
```

#### Simple Runtime example

Go to the pulpissimo root directory.

The simple runtime is here to get you started quickly. Using it can run and write programs that don't need any advanced features. It is expected that you have allready installed PULP GCCC toolchain and set appropriate exports:
```bash
export PULP_RISCV_GCC_TOOLCHAIN=/opt/riscv
export RISCV=$PULP_RISCV_GCC_TOOLCHAIN
export PATH=$PULP_RISCV_GCC_TOOLCHAIN/bin:$PATH
```

The simple runtime supports many different hardware configurations. We want PULPissimo.
```bash
cd sw/pulp-runtime
# If you are doing this first time, then run
git submodule update --init --recursive
```
Then, to use the CV32E40P (formely RI5CY) core, type:
```bash
source configs/pulpissimo_cv32.sh
```
or to use the Ibex (formely zero-riscy) core:
```bash
source configs/pulpissimo_ibex.sh
```

Now we are ready to set up the simulation environment. Normally you would want to simulate the hardware design running your program, so go [THIS](#building-the-rtl-simulation-platform) section of the README.

### Building the RTL simulation platform

Go to the pulpissimo root directory.

Note you need Questasim or Xcelium to do an RTL simulation of PULPissimo
(verilator support planned, but not finished). Intel Modelsim for Intel FPGAs
does *not* work.

To build the RTL simulation platform, start by getting the latest version of the
IPs composing the PULP system:
```bash
make checkout
```

This will download all the required IPs, solve dependencies and generate the
scripts. The dependency management tool is
[Bender](https://github.com/pulp-platform/bender).

After having access to the SDK, you can build the simulation platform by doing
the following:
```bash
make build
```
This command builds a version of the simulation platform with no dependencies on
external models for peripherals. See below (Proprietary verification IPs) for
details on how to plug in some models of real SPI, I2C, I2S peripherals.

After running `make build` you should export generated lines, for example:
```bash
export VSIM_PATH=<your pulpissimo path>/build/questasim
export VSIM="vsim"
```

For more advanced usage have a look at `./utils/bin/bender --help` for bender.


Also check out the output of `make help` for more useful Makefile targets.


### Downloading and running examples
Finally, you can download and run examples; for that you can checkout the following repositories depending on whether you use the simple runtime or the full sdk.

Simple Runtime: https://github.com/pulp-platform/pulp-runtime-examples
```bash
git clone --recursive https://github.com/pulp-platform/pulp-runtime-examples sw/pulp-runtime-examples
```

SDK: https://github.com/pulp-platform/pulp-rt-examples
```bash
git clone --recursive https://github.com/pulp-platform/pulp-rt-examples sw/pulp-rt-examples
```

Now you can change directory to your favourite test e.g.: for an hello world
test for the SDK, run
```bash
cd sw/pulp-rt-examples/hello
make clean all run
```
or for the Simple Runtime:

```bash
cd sw/pulp-runtime-examples/hello
make clean all run
```

If you have error in file `sw/pulp-runtime/bin/slm_hyper.py`, just change the `rU` to `r` on line `24`.


If you want to change the compiler flags, as for example 
if you are using CV32E40P with the XPULP extensions but you want to compile 
using only the RV32IMC instructions to compare performance,
you can modify the Makefile inside the pulp-runtime-examples/hello folder adding:

```
PULP_ARCH_CFLAGS    =  -march=rv32imc -DRV_ISA_RV32
PULP_ARCH_LDFLAGS   =  -march=rv32imc
PULP_ARCH_OBJDFLAGS = -Mmarch=rv32imc
```
The open-source simulation platform relies on JTAG to emulate preloading of the
PULP L2 memory. If you want to simulate a more realistic scenario (e.g.
accessing an external SPI Flash), look at the sections below.

In case you want to see the Modelsim GUI, just type
```bash
make run gui=1
```
before starting the simulation.

If you want to save a (compressed) VCD for further examination, type
```bash
make run vsim/script=export_run.tcl
```
before starting the simulation. You will find the VCD in
`build/<SRC_FILE_NAME>/pulpissimo/export.vcd.gz` where
`<SRC_FILE_NAME>` is the name of the C source of the test.