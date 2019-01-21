RISCV Sail Model
================

This repository contains a model of the RISCV architecture written in
[Sail](https://www.cl.cam.ac.uk/~pes20/sail/). It used to be contained
in the [Sail repository](https://github.com/rems-project/sail).

It currently implements enough of RV64IMAC to boot a conventional OS
with a terminal output device.  Work on a 32-bit model is ongoing.
The model specifies assembly language formats of the instructions, 
the corresponding encoders and decoders, and the instruction semantics.

Directory Structure
-------------------

```
sail-riscv
|
+---- model                   // Sail specification modules
|
+---- generated_definitions   // Files generated by Sail
|     |
|     +---- c
|     +---- ocaml
|     +---- lem
|     +---- isabelle
|     +---- coq
|     +---- hol4
|     +---- latex
|
|---- handwritten_support     // Prover support files
|
+---- c_emulator              // supporting platform files for C emulator
+---- ocaml_emulator          // supporting platform files for OCaml emulator
|
+---- test                    // test files
      |
      +---- riscv-tests       // snapshot of tests from the riscv/riscv-tests github repo
```

An outline of the specification
-------------------------------

The model contains the following Sail modules in the `model` directory:

- `prelude.sail` contains useful Sail library functions

- `riscv_types.sail` contains some basic RISC-V definitions

- `riscv_sys.sail` describes M-mode and S-mode CSRs and exception handling

- `riscv_platform.sail` contains platform-specific functionality
    (e.g. physical memory map and clock and terminal device interfaces)

- `riscv_mem.sail` contains the interface to physical memory

- `riscv_vmem.sail` describes the S-mode address translation

- `riscv.sail` captures the instruction definitions and their assembly language formats

- `riscv_step.sail` implements the top-level fetch and execute loop

- `riscv_analysis.sail` is used in the formal operational RVWMO memory model


Simulators
----------

The files in the OCaml and C simulators implement ELF-loading and the
platform devices, define the physical memory map, and use command-line
options to select implementation-specific ISA choices.

Building the model
------------------

Install Sail via Opam, or build Sail from source and have SAIL_DIR in
your environment pointing to its top-level directory.

```
$ make
```
will build the OCaml simulator in `ocaml_emulator/riscv_ocaml_sim`, the C simulator in
`c_emulator/riscv_sim`, the Isabelle model in `generated_definitions/isabelle/Riscv.thy`, and the Coq
model in `generated_definitions/coq/riscv.v`.

Executing test binaries
-----------------------

The C and OCaml simulators can be used to execute small test RV64 binaries.

```
$ ./ocaml_emulator/riscv_ocaml_sim  <elf-file>
$ ./c_emulator/riscv_sim <elf-file>
```
Some information on additional configuration options for each simulator is available
from `./ocaml_emulator/riscv_ocaml_sim -h` and `./c_emulator/riscv_sim -h`.

Booting Linux with the C backend
--------------------------------

The C model needs an ELF-version of the BBL (Berkeley-Boot-Loader)
that contains the Linux kernel as an embedded payload.  It also needs
a DTB (device-tree blob) file describing the platform (say in the file
`spike.dtb`).  Once those are available (see below for suggestions),
the model should be run as:

```
$ ./c_emulator/riscv_sim -t console.log -b spike.dtb bbl > execution-trace.log 2>&1 &
$ tail -f console.log
```
The `console.log` file contains the console boot messages.

Booting Linux with the OCaml backend
------------------------------------

The OCaml model only needs the ELF-version of the BBL, since it can generate its
own DTB.
```
$ ./ocaml_emulator/riscv_ocaml_sim bbl > execution-trace.log 2> console.log
```

Generating input files for Linux boot
-------------------------------------

One could directly build Linux and the toolchain using
`https://github.com/sifive/freedom-u-sdk`.  The built `bbl`
will be available in `./work/riscv-pk/bbl`.  You will need to configure
a kernel that can be booted on Spike; in particular, it should be
configured to use the HTIF console.

The DTB can be generated using Spike and the DeviceTree compiler
`dtc` as:

```
spike --dump-dts . | dtc > spike.dtb
```

(The '.' above is to work around a minor Spike bug and may not be
needed in future Spike versions.)

Caveats for OS boot
-------------------

- Some OS toolchains generate obsolete LR/SC instructions with now
  illegal combinations of `.aq` and `.rl` flags.  You can work-around
  this by changing `riscv_mem.sail` to accept these flags.

- One needs to manually ensure that the DTB used for the C model
  accurately describes the physical memory map implemented in the C
  platform.  This will not be needed once the C model can generate its
  own DTB.
