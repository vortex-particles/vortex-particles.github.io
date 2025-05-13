---
layout: default
title: Home
nav_order: 1
last_modified_at: 2024-03-12T10:39:00
---

## What is vortex-p?

<b>vortex-p</b> is a tool for the analysis of the velocity fields of astrophysical simulations of different nature (SPH, moving-mesh, meshless, etc.; but also grid-based AMR), usually spanning many orders of magnitude in scales involved. In particular, vortex-p is capable of performing:

- A Helmholtz-Hodge decomposition (HHD; i.e., the decomposition of the velocity field into a solenoidal and an irrotational/compressive part). While typical algorithms for this aim rely on uniform grids, severely limiting the dynamical range that can be covered, vortex-p internally uses an AMR representation of the velocity field and can, in principle, capture the full dynamical range of the simulation.
- A Reynolds decomposition (i.e., the decomposition of the velocity field into a bulk and a turbulent part). This is achieved by means of a multi-scale filtering of the velocity field, where the filtering scale around each point is determined by the local flow properties.


While [the original version of vortex](https://www.sciencedirect.com/science/article/pii/S0010465521000394?via%3Dihub) was original envisioned to work coupled to the outputs of the [MASCLET code](https://academic.oup.com/mnras/article/352/4/1426/1077772), vortex-p is a fully stand-alone tool, capable of working with the outcomes of a broad range of simulation methods.

See the code repository for downloading the code:

<center><button type="button" name="button" class="btn" onclick="location.href='https://github.com/dvallesp/vortex-p';">Go to the code repository</button></center>

### What is in these pages?

These pages contain the documentation of the code, including how to get vortex-p and the required libraries, how to compile it, a description of the parameters that can be tuned to run vortex-p, and how to load its outputs. For the scientific description of the code, please refer to the code papers (see below, in [Referencing vortex-p](#referencing-vortex-p))

## About

vortex-p has been developed at the Computational Cosmology Group of [Universitat de València](https://www.uv.es), with financial support from the Spanish [Agencia Estatal de Investigación](https://www.aei.gob.es/) (grant PID2022-138855NB-C33), the [Ministerio de Ciencia e Innovación (MCIN)](https://www.ciencia.gob.es/) within the Plan de Recuperación, Transformación y Resiliencia del Gobierno de España (grant ASFAE/2022/001), with funding from European Union NextGenerationEU (PRTR-C17.I1), and the [Generalitat Valenciana](https://ceice.gva.es/es/web/ciencia) (grant CIPROM/2022/49).

### Disclaimer

vortex-p and all its associated codes are a non-profit, open source tool developed in the context of a research project. While we share it in the hope that it will be useful, it comes without any guarantee to function. Please, be sure to always check the consistency of your results.

### Contact

If you have any questions, suggestions or want to contribute to vortex-p, please contact us at [this email address](mailto:david.valles-perez@uv.es).

### Referencing vortex-p

If publishing scientific results using vortex-p, please consider citing both the original vortex papers and the vortex-p code paper:

- For the original AMR-HHD algorithm: Vallés-Pérez, D., Planelles, S. & Quilis, V., "Unravelling cosmic velocity flows: a Helmholtz-Hodge decomposition algorithm for cosmological simulations", [Computer Physics Communications 263, 107892](https://doi.org/10.1016/j.cpc.2021.107892) (2021). [(arXiv link)](https://arxiv.org/abs/2102.06217) [(ADS link)](https://ui.adsabs.harvard.edu/abs/2021CoPhC.26307892V/abstract)
- For the AMR implementation of the multi-scale filter: Vallés-Pérez, D., Planelles, S. & Quilis, V., "Troubled cosmic flows: turbulence, enstrophy, and helicity from the assembly history of the intracluster medium", [MNRAS 504(1):510](https://doi.org/10.1093/mnras/stab880) (2021). [(arXiv link)](https://arxiv.org/abs/2103.13449) [(ADS link)](https://ui.adsabs.harvard.edu/abs/2021MNRAS.504..510V/abstract)
- For the vortex-p code: Vallés-Pérez, D. et al., "vortex-p: a Helmholtz-Hodge and Reynolds decomposition algorithm for particle-based simulations", [Computer Physics Communications 304, 109305](https://doi.org/10.1016/j.cpc.2024.109305) (2024). [(arxiv link)](https://arxiv.org/abs/2407.02562) [(ADS link)](https://ui.adsabs.harvard.edu/abs/2024CoPhC.30409305V/abstract)