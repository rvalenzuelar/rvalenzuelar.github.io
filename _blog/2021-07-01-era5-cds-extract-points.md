---
title: How to extract multiple points from ERA5 using cdstoolbox 
type: blog
date: 2021-07-01
authors: rvalenzuela
---

[Climate Data Store (CDS)](https://cds.climate.copernicus.eu/#!/home) is a great place to test some ERA5 data. However, downloading time series from multiples points can be difficult if not immpossible. At least, I could not figure out a way to do it using the online cdstoolbox. Using the pythons' cdsapi also proved to be unsuccsesfull.

Here I will show you a way to download ERA5 time series using a combination of tools in python and bash. 

### Requirements

* cdsapi
* cdstoolbox-remote 

### First step

Create a templeate of the cdstoolbox script that you would use in the online CDS portal. You can use the template that CDS provides, for example:

```python
import cdstoolbox as ct

# Initialise the application
@ct.application(title='Extract time-series at multiple points')

# Define a download output for the application
@ct.output.download()

# Define application function
def application():
    """Define a function that extracts hourly Near Surface Air Temperature in 2018
    for two points and provides a download link.

    Application main steps:

    - retrieve temperature gridded data
    - extract data at given locations using ct.observation.interp_from_grid()
    - return extracted data
    """

    # Retrieve hourly surface temperature
    data = ct.catalogue.retrieve(
        'reanalysis-era5-single-levels',
        {
            'variable': '2m_temperature',
            'product_type': 'reanalysis',
            'year': 2018,
            'month': list(range(1, 13)),
            'day': list(range(1, 32)),
            'time': [
                '00:00', '01:00', '02:00', '03:00',
                '04:00', '05:00', '06:00', '07:00',
                '08:00', '09:00', '10:00', '11:00',
                '12:00', '13:00', '14:00', '15:00',
                '16:00', '17:00', '18:00', '19:00',
                '20:00', '21:00', '22:00', '23:00',
            ],
            'grid':['1', '1']
            }
    )

    # Interpolate data for two points
    points = ct.observation.interp_from_grid(data, lat=[10., 65.], lon=[32., 45.])

    print(points)

    return points
```

If you have `cdsapi` and `cdstoolbox-remote` installed in your current environment you should be able to run the script above without issues. However, importing other libraries such as `Pandas` or even `sys` will trigger an error when executing this script. I am not sure why this is happening, but I tried different ways and the response from the server always retrieved an error message, even when only using `sys` to capture a year from the command line.

Importing other libraries is important because you normally will want to change some parameters dynamically, like the request year. Or, as I mentioned above, if you want to provide the parameter from the command line.

Since I was not able to import other libraries in the script above, I created another "builder" script that will write the "worker" script.

The "builder" script is the following:

```python
import sys 

var = sys.argv[1]

yrs = int(sys.argv[2])
yre = yrs+1

if var == 't2m':
	var_name = '2m_temperature'
elif var == 'slp':
	var_name = 'mean_sea_level_pressure'

script = """import cdstoolbox as ct

# Initialise the application
@ct.application(title='Extract time-series at multiple points')

# Define a download output for the application
@ct.output.download()

# Define application function
def application():

    # Retrieve hourly surface temperature
    data = ct.catalogue.retrieve(
        'reanalysis-era5-single-levels',
        {{
            'variable': '{}',
            'product_type': 'reanalysis',
            'year': list(range({}, {})),
            'month': list(range(1, 13)),
            'day': list(range(1, 32)),
            'time': [
                '00:00', '01:00', '02:00', '03:00',
                '04:00', '05:00', '06:00', '07:00',
                '08:00', '09:00', '10:00', '11:00',
                '12:00', '13:00', '14:00', '15:00',
                '16:00', '17:00', '18:00', '19:00',
                '20:00', '21:00', '22:00', '23:00',
            ],
            'grid':['1', '1']
        }}
    )

    # Interpolate data for two points
    points = ct.observation.interp_from_grid(data,
                                             lat=[...],
                                             lon=[...]
                                             )

    return points
"""

# print(script.format(yrs, yre))

sname = 'extract_grid_points_ngl.py'.format(var)
with open(sname,'w') as f:
	f.write(script.format(var_name, yrs, yre))
```

In this way you can change any parameter and the call the script using bash. I have the lat and lon coordinates fixed, but you could change them dynamically using, for example, `Pandas`.

A finall script is written in `bash`. This script will change the years and parameters requested, so that I will have one file per year and variable (recommended).

```bash
for var in {'t2m','slp'};do
	echo $var
	for yr in {1991..2010};do
		echo $yr
		# Updates download scripts
		python era5_cds_script_builder.py $var $yr

		# Run the script
		python extract_grid_points_ngl.py 

		# get the name of just-downloaded file
		x=$(find . -name "*-*.nc")

		# and rename it
		mv $x "era5_"$var"_nglpoints_"$yr".nc"
	done
done
```

The `cdstoolbox` api downloads a file with a random name of fixed character numbers. These files have some dash in between, so I used the `-` character to identify the downloaded file and rename it to a more readable name. 

If you find any issue in the scripts above or just want to comment, please send me an email.

I hope these tips will save you time!