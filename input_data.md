---
layout: default
title: Input data
nav_order: 3
last_modified_at: 2024-03-12T15:04:11
---

## Input format

In this section, we describe the different ways of loading your particle data into vortex-p. 

### The GADGET-unformatted reader

The prototype reader included in vortex-p is the one for GADGET-2 and GADGET-3 unformatted binary files. 

This reader is located in the `reader.f` file, and is split in two functions: `READ_GADGET_UNFORMATTED_NPART` and `READ_GADGET_UNFORMATTED`, both of them called within a subroutine called `READ_PARTICLES`. The two former subroutines depend on a module, called `GADGET_READ`, which is also included in the code (file `reader_gadget.f`) and contains the necessary structures and variables to read these GADGET files.

In particular, the data that needs to be read for a given snapshot is: 

- Total number of gas particles. This is provided by the `READ_GADGET_UNFORMATTED_NPART` function, which just scans the headers of all the snapshot files.
- Positions of gas particles.
- Velocities of gas particles.
- Masses of gas particles.
- Kernel lengths of gas particles, for purposes of interpolating results back to particles.

If the multiscale filter is to be used, then one of the two following variables must be read:
- Artificial bulk viscosity constant (as a proxy to locate strong shocks).
- Shock Mach number, obtained via an on-the-fly shock finder.

Additionally, if the velocity field is to be volume-weighted, the density of the gas particles must be read.

### The Arepo hdf5 reader

Also included in vortex-p is a reader for Arepo hdf5 output files. This reader is activated at [compilation time](get_vortexp#compilation) with the option `READER=1` in the make command. It is located in the same `reader.f` file.

> NOTE: both the Arepo and the GADGET readers should be relatively plug-and-play. But, since data formats can change depending on the version and flavour of the code, be sure to check the format of your input data files for possible variations.

### Creating your custom reader subroutine

You can alter the reader format by creating a couple of custom subroutines in the `reader.f` source file, or directly modifying the `READ_GADGET_UNFORMATTED_NPART` and `READ_GADGET_UNFORMATTED` subroutines. If creating your own subroutines, you can use the ones for GADGET-unformatted as a template, and do not forget to set the correct calls to the new functions inside the `READ_PARTICLES` subroutine.

#### Contributing

If you have created a general reader routine for a widely-used, publicly-available simulation code and want it included in the public version of vortex-p, do not hesitate to create a pull request or to [contact us](mailto:david.valles-perez@uv.es).