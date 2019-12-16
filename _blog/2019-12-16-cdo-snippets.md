---
title: Climate Data Operators (CDO) snippers
type: blog
date: 2019-12-13
authors: rvalenzuela
---

CDO is a suit of command line tools to operate and edit NetCDF and grib files. Here are some snippets that I use the most. In addition, NCO is a similar suit of tools but dedicated to NetCDF only. The snippets are implemented in Bash, so that the operation is executed in a set of files.


# Add the values of two variables located in their own files
```bash
for i in $(seq 1979 1979); do
	echo $i
	cdo -b F32 enssum ./crr/era5_convective_rain_rate_$i.nc ./lsrr/era5_large_scale_rain_rate_$i.nc ./trr/era5_total_rain_rate_$i.nc;
done
```

# Crop a latlon box
```bash
for i in $(seq 1979 2018); do
	echo $i
	cdo sellonlatbox,-73,-69.75,-36.25,-32.5 ./trr/era5_total_rain_rate_$i.nc ./trr_box/era5_total_rain_rate_$i.nc;
done
```

# Replace the name of variables and their long_name (which is an attribute)
```bash
for i in $(seq 1979 2018); do
	echo $i
	cdo chname,crr,trr ./trr_box/era5_total_rain_rate_$i.nc ./trr_box/era5_total_rain_rate_$i.nc;
	ncatted -a long_name,trr,o,c,"Total rain rate" -O ./trr_box/era5_total_rain_rate_$i.nc ./trr_box/era5_total_rain_rate_$i.nc;
done
```