---
layout: default
title: Getting vortex-p
nav_order: 2
last_modified_at: 2022-04-12T11:08:56
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
- `NLEVELS` is the maximum number of refinement levels. Take into account that your best resolution to identify density peaks will be <img src="https://render.githubusercontent.com/render/math?math=L/(N_x \cdot 2^\mathrm{NLEVELS})">, with <img src="https://render.githubusercontent.com/render/math?math=L, \, N_x"> the domain length and the number of grid cells. A typical suggestion is to set `NLEVELS` to match the force resolution of the simulation, since you are not expected to form structures below this scale. $a + b$

```fortran 
! Maximum patch extension 
INTEGER NAMRX,NAMRY,NAMRZ
PARAMETER (NAMRX=32,NAMRY=32,NAMRZ=32)
```
- `NAMRX`, `NAMRY` and `NAMRZ` are the maximum extension of the AMR patches. The smaller the value, the more modular the AMR structure will be, which could involve less memory usage. 32 or 64 are typically reasonable values.



### Compilation

ASOHF can be compiled either with the GNU (`gfortran`) or with the Intel (`ifort`) Fortran Compilers. The following compilation options work well for us using gfortran-11 and ifort 2018 on several machines:

#### GNU:
```bash
gfortran -O3 -march=native -fopenmp -mcmodel=medium -funroll-all-loops -fprefetch-loop-arrays -mieee-fp -ftree-vectorize particles.f asohf.f -o asohf.x
```

#### Intel:
```bash
ifort -O3 -mcmodel=medium -qopenmp -shared-intel -fp-model consistent -ipo -xHost particles.f asohf.f -o asohf.x
```

#### macOS and the stack size:
macOS system have a limitation of the maximum stack size that can be granted to a process, regardless of what the user specifies with the `OMP_STACKSIZE` environment variable or the `ulimit` command. ASOHF makes use of the stack, rather than the heap, by passing several large arrays as parameters to subroutines rather than using `COMMON` blocks, since it provides faster access. However, this may cause instantaneous failure of the code with Apple systems running macOS.

To circumvent this, please use the `-Wl,-stack_size,0x80000000` flag in your compilation command to set the stack size to 2GB, which should be enough even for large simulations. 

### Running

For running ASOHF, you may want to use a shell script containing all the necessary environment variables. An example is served below:

```bash
>> cat run.sh
#!/bin/bash

export OMP_NUM_THREADS=24 # number of threads
export OMP_STACKSIZE=3000m # size of the stack
export OMP_PROC_BIND=true # avoid switching threads
export OMP_WAIT_POLICY=active
export KMP_BLOCKTIME=1000000 #very large number

ulimit -c 0 # avoid core dumps

./asohf.x > asohf.out
```
