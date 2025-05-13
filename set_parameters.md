---
layout: default
title: Setting the parameters
nav_order: 4
has_children: true
last_modified_at: 2024-03-13T09:23:11
---

## Setting the parameters of vortex-p

In this section, we describe the run-time parameters of vortex-p, which are set in the `vortex.dat` file. These can be changed after [compilation of the code](get_vortexp#compilation).

> It is worth reminding here that, except for the possible parameters to flag strong shocks (only used if the multiscale filter is applied and no Mach number information is given), there are no dimensional parameters in vortex-p. This also means that the decompositions are unit-system-independent, and there is no need to perform unit conversions in IO.

### General parameters block

These parameters refer to the general behaviour of the code.

```
Files: first, last, every, num files per snapshot -------------------->
144,144,1,2
```
- Simulation snapshots to analyse. `vortex-p` will be executed for snapshot `first` to `last`, every `every` snapshots. Additionally, the GADGET-unformatted reader supports that the snapshot files are split in a number of `snapXXX.0`, `snapXXX.1`, etc. files. In this case, the last parameter specifies the number of files per snapshot.
If snapshots are numbered differently, it is straightforward to change the main loop of the code (look in the `vortex.f` file for the `DO IFI=1, NFILE` line). 

```
Cells per direction (NX,NY,NZ) --------------------------------------->
128,128,128
```
- Number of base grid cells in each direction. These grid sizes should be smaller or equal than the [compilation parameters `NMAX`, `NMAY` and `NMAZ`](get_vortexp#compilation-time-parameters). As a rule of thumb, choose $N_x \sim \sqrt[3]{N_\mathrm{part}}$.

```
Max box sidelength (in input length units) --------------------------->
50.e3
```
- The largest of the box side lengths, $L$ in the input length units. The base grid resolution will then be $L/N_x$.

```
Domain to keep particles (in input length units; x1,x2,y1,y2,z1,z2) -->
-19972.0312, 30027.9688, -27619.4688, 22380.5312, -24089.4375, 25910.5625
```
- Only particles inside this region will be considered. Note that $\max(x_2- x_1, y_2-y_1, z_2-z_1) \leq L$. Ideally, choose the domain to be considerably larger than your object of interest. For instance, if you are analysing a (re)simulation of a galaxy cluster, choose a box at least 3 or 4 times the extent of the cluster.


### Output customisation block
These parameters can be used to customise the outputs of vortex-p. For each entry below, the user can choose to output that quantity (1) or not (0).

```
Gridded data: kernel length, density (mutually exclusive), velocity -->
0,1,1
```
- This block refers to the input quantities assigned to the AMR grid. The first one is the length around each cell centre considered to assign the velocity field. The second one is the density field, computed with the same grid-assignment scheme as the velocity field. The third one is the velocity field itself. Note that the first and second outputs are mutually exclusive: only one of them can be set to 1.

```
Gridded results: vcomp, vsol, scalar_pot, vector_pot, div(v), curl(v)->
1,1,1,1,1,1
```
- This block refers to the output quantities on the AMR grid. In order, we have the compressive velocity, the solenoidal velocity, the scalar potential of the compressive velocity, the vector potential of the solenoidal velocity, the divergence of the velocity field, and the curl of the velocity field.

> Note: when running vortex-p with the turbulent filter, these results correspond to the turbulent velocity field.

```
Particle results: interpolation error, particle-wise results --------->
1,1
```
- This block contains the output quantities reinterpolated back to particles. The first one refers to an estimation of the error in the interpolation of the velocity field, defined here as a measure of the difference between the individual particle velocity and the smoothed velocity at its location. The second one refers to the particle-wise results, which contain the same information as the gridded results (total, compressive and solenoidal smoothed velocity), but reinterpolated back to the particles.

```
Filter: gridded Mach/ABVC, shocked cells, filtering length, vturb ---->
1,1,1,1
```
- This block contains the output quantities related to the multiscale filter. The first one refers to the gridded Mach number (or the artificial bulk viscosity constant, if the Mach number is not available and that option is chosen below). The second one refers to the cells flagged as shocked. The third one refers to the filtering length, which is the length scale used to split bulk and turbulent contributions at each location. The fourth one refers to the turbulent velocity itself.

### Mesh creation parameters block
These parameters control the mesh creation process, which is crucial for recovering a faithful representation of the velocity field with enough dynamical range.

```
Number of levels ----------------------------------------------------->
9
```
- Maximum number of refinement levels to create. Take into account that your best resolution will be $L/(N_x \cdot 2^\mathrm{NLEVELS})$, with $L, \, N_x$ the domain length and the number of base grid cells per direction. A typical suggestion is to set it to the best resolution (smallest kernel length), since below this resolution the dynamics are not resolved.

```
Number of particles for a cell to be refinable ----------------------->
8
```
- A cell will be flagged as refinable if it hosts more (or equal) particles than this threshold. This parameter is crucial to control the number of patches (and, thus, to keep memory usage and CPU time contained). If you think that the code is creating too many patches, you may want to increase this parameter. If you think that the code is underresolving important regions, you may want to decrease it.

```
Minimum size of a refinement patch to be accepted (<0: octree-like)--->
-6
```
- If this parameter is negative (default), the refinement will be octree-like, meaning that given blocks of the domain (of size NAMRX, defined in the [compilation parameters](get_vortexp#compilation-time-parameters) section) will be flagged for refinement if they contain enough refinable cells. This is the default behaviour, and it is recommended for most cases.

- If the parameter is positive, the refinement is more dynamic, and refinement regions can overlap to ensure an adequate coverage of the largest amount of refinable cells. Not all refinable cells get finally refined, but only if they are going to be covered by a patch whose minimum extent (measured in number of fine, i.e. refined, cells) is at least this number. The lower the value, the more aggressive the refinement. However, it will importantly increase the computational cost. A reasonable value is around 6-8, although in test runs it may be advisable to start with a larger number (i.e., 20) to decrease the computing time.

```
Cells not to be refined from the border (base grid) ------------------>
2
```
- So as to not interfere with the boundary conditions, AMR patches are not allowed to cover this number of cells from any face of the base grid. This may be set to a larger value if one wants to refine only in a more central region of the domain.

### Velocity interpolation parameters block
This block contains the parameters referring to the grid-assignment of the velocity field. Note that there is only one parameter here. The other relevant parameter, i.e. the kernel function, is [set in compilation time](get_vortexp#compilation-time-parameters).

```
Number of neighbours for interpolation ------------------------------->
58
```
- The number of neighbours to consider, around each cell centre, for the interpolation of the velocity field. This number should be large enough to ensure that the kernel function is well sampled, but not too large to avoid excessive computational cost.

> Note: in an SPH simulation, a reasonable rule of thumb is to set the kernel function and the number of neighbours to be the same as used when evolving the simulation, since this is the most physically-motivated definition of the underlying velocity field.

> Note: in a moving-mesh simulation, smaller values can be used. Reasonable numbers could be around 16.

### Poisson solver block 

These parameters control the Poisson solver, which is used to compute the potential fields.

```
SOR presion parameter, SOR max iter, border for AMR patches ---------->
1e-9,1000,2
``` 

- The first parameter is the precision parameter for the SOR solver. The SOR iterations in the AMR patches are stopped when the relative variation falls below this value. The second one is the maximum number of iterations, to prevent the SOR solver to get stuck in some rare situations. The third one is the number of ghost cells along each direction, used to enforce the boudnary conditions away from the region of interest of each patch.

### Multifiltering
These parameters control the multifiltering process, which is used to adaptively extract the turbulent contribution to the velocity field.

```
Apply filter (1: multiscale filter; 2: fix-scale filter) ------------->
1
```
- Use 0 for no filtering (work with the total velocity field), 1 for the multiscale filter, and 2 for filtering with a fixed length scale (to be specified below).
- Keep in mind that running the filter requires the `FILTER=1` flag in the [compilation process](get_vortexp#compilation). If, after compiling, you do not want to use the filter, you can set this parameter to 0 without the need of recompiling the code. 

```
Filtering parameters: tolerance, growing step, max. num. of its. ----->
0.1,1.05,200
```
- In the multiscale filter only: the filtering length is increased until the relative variation of the turbulent velocity fields falls below the tolerance set by the first parameter. Each iteration of the filter algorithm increases the filtering length by the factor set by the second parameter. The third parameter is the maximum number of iterations allowed, to avoid pathologic cases where the filter does not converge.

```
Maximum (for multiscale) or fix filt. length (input length units) ---->
1000.0
``` 
- If apply filter is 1, this sets the maximum filtering length for the multiscale filter (if you don't want to impose a maximum filtering length, just set this to a value larger than your domain size). If apply filter is 2, this sets the fixed filtering length.

```
Smooth filtering length before applying the filter (0=no, 1=yes) ----->
1
```
- If you are using the filter in option 1 (multiscale), this parameter sets whether to smooth the filtering length (which can have large cell-to-cell variations due to the convergence of the algorithm) before determining the filtering length. This is advised, and the default value of the parameter is thus 1 (apply the smoothing). If you set it to 0, the filtering length will be used as computed by the algorithm.

### On-the-fly shock detection (for multifiltering)
The last block of parameters refers to the flagging of strong shocks, which is used as an additional stopping condition for the iterations of the multiscale filter. This is only used if the multiscale filter is applied.

The first two parameters are used in case no Mach number information is available. The third one is used to flag strong shocks if the Mach number is available.

```
Threshold on velocity divergence (negative, input units) ------------->
-1.25
Threshold on artificial bulk viscosity constant ---------------------->
1.
```
- Without Mach number information, strong shocked cells are flagged as those with artificial viscosity above a certain threshold, $\alpha > \alpha^\mathrm{thr}$, and strong compression, $\nabla \cdot \mathbf{v} < (\nabla \cdot \mathbf{v})^\mathrm{thr}$. Note that the latter: (i) must be negative, and (ii) is the only dimensional parameter. Its value, if used, must be set in the input velocity / length units.

```
Use particle's MACH field (0=no, 1=yes), Mach threshold -------------->
1,2.0
```
- Alternatively, if there is Mach number information available (first parameter set to 1), strong shocked cells are flagged as those with Mach number above a certain threshold, $\mathcal{M} > \mathcal{M}^\mathrm{thr}$, set by the second parameter.