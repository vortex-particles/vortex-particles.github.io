---
layout: default
title: Parameters for the grid-based input
nav_order: 1
parent: Setting the parameters
last_modified_at: 2024-03-13T09:23:11
---

## Parameters for the grid-based input
In the spirit of unifying vortex and vortex-p, now vortex-p is the reference version to use both for particle-based (and analogue) inputs, and for grid-based inputs (e.g., AMR).

When inputting data from a grid-based simulation output, many parameters are not needed, and their value in the `vortex.dat` file does not have any effect.

In particular:

- $N_x$, $N_y$ and $N_z$ should be the base grid resolution of your simulation.
- You cannot select the domain: the grid is input as a whole.
- Naturally, everything referring to particle results is ignored (in the output customization block)
- If you run the code with the turbulent filter enabled, shocks are necessarily from Mach number data, which needs to be input into the code.
- The number of levels should be your simulation higher refinement level (where 0 is the base level resolution), or a lower number if you want to limit the analysis to a lower resolution.
    - All the rest of parameters regarding mesh creation are ignored.
- All parameters regarding velocity assignment/interpolation are ignored.
- On-the-fly shock finding is not available: you need to input the shock Mach number data if you want to use the turbulent filter.

>#### NOTE ON AMR INPUT
>
>It is implicitly assumed that the refinement factor is 2. The code does not support other refinement factors in its current version.
