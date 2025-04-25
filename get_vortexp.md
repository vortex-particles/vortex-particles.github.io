---
layout: default
title: Getting vortex-p
nav_order: 2
last_modified_at: 2024-03-12T12:29:56
---

## Getting vortex-p

This section describes the required operations to download vortex-p, setting the required folder structure, installing the required libraries, compiling the code and running it.

### Source download

The easiest way to download the code is to directly clone [the repository](https://github.com/dvallesp/vortex-p). If you happen to not have git in your system, you can find [some nice tutorials](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) on how to install it, or you could just download the source directly from the GitHub repository page.

To get the code, just run the following command:

```bash
git clone https://github.com/dvallesp/vortex-p
```

This will create an vortex-p folder in your working directory, inside which you fill find the `./src` folder containing the source code of vortex-p. It contains:

- Several `.f` Fortran source files, which contain the different subroutines and modules of the code. 
- A `Makefile` to compile the code. You might need to modify it to match your system's configuration (see below).
- The `vortex_parameters.dat` file, containing the [compilation-time parameters](#compilation-time-parameters) of the code.
- The `vortex.dat` file, containing the [run-time parameters](set_parameters.md) of the code.

### Folder structure

Once within the `src` folder, the code also requires you to create a `./simulation` folder, which can be a symlink to the folder containing your simulation results.

```bash 
ln -s /path/to/your/simulation ./simulation
```
Additionally, you will need to create and `./output_files` folder, where the code will store the outputs of the analysis.

```bash
mkdir ./output_files
```

### Compilation-time parameters

Before compiling, it is required to set some compilation-time parameters, which are related to dimensioning of arrays. These parameters are included in the `vortex_parameters.dat` file, and are explained one by one below. Each time you change the parameters, you need to [recompile the code](#compilation).

```fortran 
! Level 0 grid 
INTEGER NMAX,NMAY,NMAZ
PARAMETER (NMAX=128,NMAY=128,NMAZ=128)
```
- `NMAX`, `NMAY` and `NMAZ` are the (maximum) number of base grid cells in each direction. They are used in order to dimension the density arrays (must be larger or, ideally, equal to the actual grid size used, specified in `vortex.dat`; see [the parameters file page](set_parameters.md)).
  
```fortran 
! Refinements 
INTEGER NPALEV,NLEVELS
PARAMETER (NPALEV=10000,NLEVELS=4) 
```
- `NPALEV` is the maximum number of AMR patches. This quantity may vary significant depending on your specific user case, so it is advisable to run the code with a large value (e.g. `NPALEV=10000`). If it is too small (i.e., when running the code, the maximum number of patches is reached), the code will stop with an error message. If it is too generous, you can consider lowering it to save memory.
- `NLEVELS` is the maximum number of refinement levels. Take into account that your best resolution to identify density peaks will be $L/(N_x \cdot 2^\mathrm{NLEVELS})$, with $L, \, N_x$ the domain length and the number of base grid cells. A typical suggestion is to set `NLEVELS` to match the force resolution of the simulation, since you are not expected to form structures below this scale. $a + b$

```fortran 
! Maximum patch extension 
INTEGER NAMRX,NAMRY,NAMRZ
PARAMETER (NAMRX=32,NAMRY=32,NAMRZ=32)
```
- `NAMRX`, `NAMRY` and `NAMRZ` are the maximum extension of the AMR patches. The smaller the value, the more modular the AMR structure will be, which could involve less memory usage. 32 or 64 are typically reasonable values.


### Required libraries

#### coretran 

The kd-tree space-partitioning algorithm used in vortex-p is provided by the [`coretran` library](https://github.com/leonfoks/coretran), which contains many well-optimised routines for intensive numerical computation.

While `coretran` is well documented, for the sake of completeness, here we summarise the steps to install it locally:

1. First, move to the folder you want to install `coretran` in (in our example, it will be the home directory), and clone the repository:

```bash
cd ~
git clone https://github.com/leonfoks/coretran
cd coretran
```

2. Create a `build` directory, where the library will be compiled, and a `library` directory, where we will install the library. Enter the `build` directory:

```bash
mkdir build
mkdir library
cd build
```

3. Generate the Makefile with `cmake`:

```bash
cmake -DBUILD_SHARED_LIBS=ON -DCMAKE_INSTALL_PREFIX=/home/your_user/coretran/library ../src
```

When doing so, make sure that cmake detects the Fortran compiler you want to use. If it does not, you can specify it with the `-DCMAKE_Fortran_COMPILER` flag. If you are using a cluster, in general you just need to unload all compilers and load the one you want to use, and then cmake will detect it. In our tests, we have used gfortran-11. We recommend you to do the same to avoid any potential issues with untested configurations.

4. Compile and install: 

```bash
make 
make install
```

Now, the library is installed in your `~/coretran/library` folder, which contains the relevant library and include files. In the `~/coretran/bin/` folder you will find two executables to test your installation.

#### FFTW

vortex-p can, optionally, make use of FFTW for fast, parallel computation of the Fourier transforms required for the Helmholtz-Hodge decomposition. FFTW is fairly easy to install with your preferred package manager, and is as a matter of fact installed in most computing clusters (at least, in those abundantly used by astrophysicists!). You can get more information from [the FFTW website](http://www.fftw.org/download.html). In our tests, we have used FFTW 3.3.8.

### Compilation

#### Setting the Makefile

You can now compile the code by using the provided Makefile. This Makefile provides some examples with different paths and configuration in the system. If you have compiled `coretran` as described [above](#coretran), you may use the option `COMP=3`.

Note that:

- If you have installed `coretran` in a different place/way, you will need to modify the `Makefile` accordingly. In particular, these two lines:

```Makefile
 LIBS=-Wl,-rpath,$(HOME)/coretran/lib $(HOME)/coretran/lib/libcoretran.so
 INC=-I$(HOME)/coretran/library/include/coretran/
 ```
 - If you want to use FFTW, you need to link it properly, 
```Makefile
 ifeq ($(FFTW),1)
  LIBS +=-L/usr/local/lib64 -lfftw3f_omp -lfftw3f
 endif
 ```

#### Compiling options

The Makefile provides several options to compile the code. You can obtain help by running:

```bash 
make info
```

The only mandatory option is:

- `COMP`: the compilation options. If you have followed the steps here, you can use `COMP=3`.

Several important optional options (which are not mandatory; check the default values with `make info`) are:

- `READER`: used to toggle between the different included readers. Available options (default is the first one) are:
    - `READER=0`: Gadget unformatted snapshot reader (default).
    - `READER=1`: Arepo HDF5 snapshot reader. 
- `FFTW`: if you want to use FFTW, set `FFTW=1` (default); else`FFTW=0`.
- `FILTER`: if you want to use the multi-scale filter, set `FILTER=1`; else `FILTER=0` (default).
- `WEIGHT`: the weighting scheme: particle-number-weighted (0; default), mass-weighted (1), or volume-weighted (2).
- `KERNEL`: the kernel family (0: cubic spline, M4, default; 1: Wendland C4; 2: Wendland C6; 3: quartic spline, M5; 4: quintic spline, M6).
- `WEIGHT_FILTER`: by default bulk velocities are volume-weighted. If you se this to `WEIGHT_FILTER=1`, the bulk velocities will be mass-weighted. This is only relevant if you are using the filter.
- `OUTPUT_GRID`: if you want to output any gridded results and/or inputs, set `OUTPUT_GRID=1` (default); else `OUTPUT_GRID=0`.
- `OUTPUT_PARTICLES`: if you want to output any particle results, set `OUTPUT_PARTICLES=1` (default); else `OUTPUT_PARTICLES=0`.
- `OUTPUT_FILTER`: if you want to output any information about the filter (turbulent total velocities prior to the HHD, filtering lengths, etc.), set `OUTPUT_FILTER=1` (default); else `OUTPUT_FILTER=0`.

The behaviour of vortex-p with respect to the outputs and the filtering scheme can be further tuned at runtime by modifying the `vortex.dat` file. See [the parameters file page](set_parameters.md) for more information. Therefore, if you have compiled the code with `FILTER=1`, you can still choose to not use the filter at runtime (but not the other way around). The same applies to the `OUTPUT_GRID`, `OUTPUT_PARTICLES` and `OUTPUT_FILTER` options. This is mainly done to generate a more efficient executable.

### Running

For running vortex-p, you may want to use a shell script containing all the necessary environment variables. An example is served below:

```bash
>> cat run.sh
#!/bin/bash

ulimit -s 128000000
ulimit -v 500000000 # very large, even unlimited if your system configuration allows! (this is not real memory)
ulimit -c 0
export OMP_NUM_THREADS=24 # Number of threads to use.
export OMP_STACKSIZE=4000m
export OMP_PROC_BIND=true

# Run the code. You may use unbuffer to get instant output logs in the vortex.out file.
./vortex > vortex.out  &
```
