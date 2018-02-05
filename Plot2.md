#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
Created on Sat Feb ,3 2018

@author: JQ
"""

#############################################
import pandas as pd
import os
import matplotlib.pyplot as plt
import numpy as np


# import potable water data
potable = pd.read_csv('UNdata_Export_PercPotableWater.csv')

# process potable:
df_potable = potable.drop(['Value Footnotes'], axis = 1)
df_potable = df_potable.pivot(index='Year', columns='Country or Area', values='Value')
df_potable['medianPotable'] = df_potable.median(axis=1)
df_potable = df_potable.reset_index()

# process the life expectancy
lifeExp = pd.read_csv('WHO_lifeExpectancy.csv', skiprows=1)

# simplify and just keep the first three columns, so can use the combined life expectancy at birth
lifeExp = lifeExp[['Country', 'Year', 'Both sexes']]

# now collect the 2015 data, which is the latest in the WHO data
potable2015 = potable[potable['Year'] == 2015]
life2015 = lifeExp[lifeExp['Year'] == 2015]

# merge dataframes
df_2015 = pd.merge(left = life2015[['Both sexes', 'Country']], right = potable2015[['Value', 'Country or Area']], how = 'inner', left_on = 'Country', right_on = 'Country or Area')

# linear fit
from scipy import stats

# select the data for each variable
x = pd.to_numeric(df_2015["Both sexes"])
y = pd.to_numeric(df_2015["Value"])

# create mask to hide the nan values
mask = ~np.isnan(x) & ~np.isnan(y)

# run linear fit
slope, intercept, r_value, p_value, std_err = stats.linregress(x[mask],y[mask])
line = slope*x+intercept

# plot scatterplot and fitted line
fig = plt.figure()
plt.plot(x, y, 'o', x, line)
plt.xlabel("Life expectancy at birth (years)")
plt.ylabel('% of Population with access to potable water')
plt.title('Linear relationship between potable water and life expectancy')
plt.show()

print(r_value)
