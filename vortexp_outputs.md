---
layout: default
title: Outputs structure
nav_order: 5
last_modified_at: 2024-03-13T10:03:41
---

## vortex-p outputs
Here, we describe the basic structure of vortex outputs, which are stored in `./output_files`. The data that is output is [highly customizable in the parameters file](set_parameters#output-customisation-block). All outputs are saved a number of files per snapshot. Apart from the grid description (`gridsXXXXX` file), which is in text format, all the rest of data is saved in unformatted binary form. To facilitate the loading of the data into python for further analysis, we provide a set of python readers for each of the output files, described [below](#python-readers).

The outputs are saved in the following files, where `XXXXX` is the snapshot number:

- General output files, necessary to use any gridded data:
    - `gridsXXXXX`: basic description of the AMR grid, necessary to locate each refinement patch in the domain. 
    - `grid_overlapsXXXXX`: description of the overlap between patches at the same and at different AMR levels, necessary to handle easily some AMR operations.
- Gridded data:
    - `gridded_densityXXXXX`: the gas density field on the AMR grid.
    - `gridded_kernel_lengthXXXXX`: the radius of the kernel used, around each cell centre, to compute the velocity field.
    - `gridded_velocityXXXXX`: the gas velocity field on the AMR grid.
- Gridded data and results for the Reynolds decomposition:
    - `gridded_machXXXXX`: the gas Mach number field on the AMR grid (used for the turbulent filter).
    - `gridded_abvcXXXXX`: the gas ABVC field on the AMR grid (used for the turbulent filter if no Mach number information is supplied).
    - `shockedXXXXX`: the cells flagged as shocked for the adaptive turbulent filter.
    - `gridded_filtlenXXXXX`: the local filtering length as determined by the adaptive turbulent filter.
    - `gridded_vturbXXXXX`: the turbulent velocity field on the AMR grid.
- Gridded results from the Helmholtz decomposition (this corresponds to the HHD of the total velocity, if the filter is not used; or to the HHD of the turbulent velocity, if the filter is used):
    - `divvXXXXX`: the divergence of the velocity field on the AMR grid.
    - `curlvXXXXX`: the curl of the velocity field on the AMR grid.
    - `spotXXXXX`: the scalar potential of the HHD on the AMR grid.
    - `vpotXXXXX`: the vector potential of the HHD on the AMR grid.
    - `vcompXXXXX`: the compressive velocity field on the AMR grid.
    - `vsolXXXXX`: the solenoidal velocity field on the AMR grid.
- Results reinterpolated back to the particles' locations:
    - `error-particlesXXXXX`: the "error" in the velocity interpolation, defined as the fractional difference between the particle input velocity, and the SPH-defined velocity at the particle location.
    - `velocity-particlesXXXXX`: the results of the decomposition, reinterpolated back to the particles' locations.

### Python readers
In order to handle easily these outputs, we provide a python library (`./python_reader` on the repository root) to reader all these output files, one by one.

Further computations with the AMR-gridded data, such as the generation of uniform grids from the AMR data to make slices and projections, can be easily performed with the [masclet_framework library](https://github.com/dvallesp/masclet_framework). Note that this library requires some additional (rather standard) dependencies (numpy, scipy, matplotlib, astropy, etc.), which can be installed via pip or your preferred package manager.

An example Jupyter notebook showing how to use the python readers and the masclet_framework library (which you will need to clone separately) is provided in the same `./python_reader` directory.