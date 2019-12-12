---
title: Read CR2 meteorological dataset
type: blog
date: 2019-12-12
authors: rvalenzuela
---

### This is a python snippet to read the csv file

csv_file = 'cr2_prDaily_2019.txt'
ds=pd.read_csv(csv_file, skiprows=range(1,17),
						 parse_dates=[0],
						 index_col=0,
						 na_values=-9999.0)