---
title: Read CR2 meteorological dataset
type: blog
date: 2019-12-12
authors: rvalenzuela
---

### This is a python snippet to read the csv file

```python
import pandas as pd

data_file = 'cr2_prDaily_2019.txt'
meta_file = 'cr2_prDaily_2019_stations.txt'

ds = pd.read_csv(data_file, skiprows=range(1,17),
                            parse_dates=[0],
                            index_col=0,
                            na_values=-9999.0)

st = pd.read_csv('cr2_prDaily_2019_stations.txt')


```