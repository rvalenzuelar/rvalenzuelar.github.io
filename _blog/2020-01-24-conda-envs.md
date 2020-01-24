---
title: Conda environemnt recipies 
type: blog
date: 2020-01-24
authors: rvalenzuela
---

There are times when you need to run things fast and your conda environment is not helping. Here I will include some environments I'm currently using for different purposes. In this way, if you need to create an env in a hurry, you can copy paste the conda command.

### Geospatial plotting/analysis
```bash
conda create --name geopan python geopandas xarray matplotlib ipython numpy cartopy descartes
```
If you use NetCDF files, it might complaing about a NetCDF 3 and 4 issue. This is fixed by installing netcdf4 with pip:
```bash
pip install netcdf4
```
