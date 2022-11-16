```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
from lxml import html
import csv 
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
#Time Series packages
from matplotlib.dates import DateFormatter, date2num, WeekdayLocator, DayLocator, MONDAY
import datetime
import matplotlib
import os
```


```python
#Plot styles 
plt.style.use('fivethirtyeight')
plt.rcParams["figure.figsize"] = (13,9)
```


```python
# Get the current working directory
cwd = os.getcwd()

# Print the current working directory
print("Current working directory: {0}".format(cwd))

# Print the type of the returned object
print("os.getcwd() returns an object of type: {0}".format(type(cwd)))
```

    Current working directory: /Users/tylerbrown
    os.getcwd() returns an object of type: <class 'str'>



```python
SNHU_2022 = pd.read_html('https://snhupenmen.com/sports/baseball/stats/2022', header = 0)
SNHU_2021 = pd.read_html('https://snhupenmen.com/sports/baseball/stats/2021', header = 0)
```


```python
#Game by game table
GBG = SNHU_2022[6]
GBG2021 = SNHU_2021[6]
```


```python
#Splitting the scores into a seperate table
Runs = SNHU_2022[6]['Score'].str.split('-', expand = True)
Runs21 = SNHU_2021[6]['Score'].str.split('-', expand = True)
```


```python
#Adding the Runs to the game to game 
GBG[['SNHU','OPP']] = Runs
GBG2021[['SNHU','OPP']] = Runs21
```


```python
#Dropping Score column after adding indvidual scores 
GBG = GBG.drop(['Score'], axis = 1)
GBG2021 = GBG2021.drop(['Score'], axis = 1)
```


```python
#Time Series Batting Averages Dictionaries
SNHU_Schedule_Dictionary = {'Date': SNHU_2022[6].Date,
                           'Cumulative BA': SNHU_2022[6].H.cumsum()/ SNHU_2022[6].AB.cumsum(),
                            'Game to Game BA' : SNHU_2022[6].H/SNHU_2022[6].AB,
                            'Game to Game wOBA': ((GBG['BB'] * .69) + (GBG['HBP'] * .72) + 
                                                  (GBG['H'] - GBG['2B'] - GBG['3B'] - GBG['HR'])  +
                                                  (GBG['2B']*1.27) + (GBG['3B']*1.62) + (GBG['HR']))/
                                                    (GBG['AB'] + GBG['BB'] + GBG['SF'] + GBG['HBP']),
                            
                            'Cumulative wOBA': ((GBG['BB'] * .69).cumsum() + (GBG['HBP'] * .72).cumsum() + 
                                                (GBG['H'].cumsum() - GBG['2B'].cumsum() - GBG['3B'].cumsum() - GBG['HR'].cumsum())  +
                                                (GBG['2B']*1.27).cumsum() + (GBG['3B']*1.62).cumsum() + 
                                                (GBG['HR']).cumsum())/(GBG['AB'].cumsum()+ GBG['BB'].cumsum() + GBG['SF'].cumsum() + GBG['HBP'].cumsum())
                           }            
```


```python
Cum_Avg ={
    'Date': SNHU_2022[6].Date,
    'Game to Game BA': SNHU_2022[6].AVG    
    }     
```


```python
#Frames for averages 
overall = pd.DataFrame(SNHU_Schedule_Dictionary)
cumulative = pd.DataFrame(Cum_Avg)
```


```python
#Plotting the cumulative batting average with the game by game
overall.plot()
plt.title('Hitting Performance Game over Game')
plt.axvline(x=45, color = 'Black', label = 'End of Reg. Season')
plt.legend()
```




    <matplotlib.legend.Legend at 0x7febe16a1190>




    
![png](output_11_1.png)
    



```python
#Creating a dataframe for the schedule dictionary 
Schedule = pd.DataFrame(SNHU_Schedule_Dictionary)
```


```python
#Converting the schedule dates to datetimes
Schedule.index = pd.to_datetime(Schedule.index)
```


```python
#SPlitting names
Hitting_2022_Players = SNHU_2022[0].Player.str.split(',', expand = True)
```


```python
#Seperating Games played and games started
GP_GS = SNHU_2022[0]['GP-GS'].str.split('-', expand = True)
GP_GS = GP_GS.astype({0:'float'})
GP_GS = GP_GS.astype({1:'float'})
```


```python
#Creating a dictonary for key stats
Dictionary = {'Player' : Hitting_2022_Players[2] + ' ' + Hitting_2022_Players[0],
              'Games Played': GP_GS[0],
                    'BA': SNHU_2022[0].AVG,
                     'OPS':SNHU_2022[0].OPS,
                     'HBP': SNHU_2022[0].HBP,
                     'AB': SNHU_2022[0].AB,
                     'Hits':SNHU_2022[0].H,
                     'BB': SNHU_2022[0].BB,
                     'Double':SNHU_2022[0]['2B'] ,
                     'Triple':SNHU_2022[0]['3B'] ,
                     'HR':SNHU_2022[0].HR,
                     'SF': SNHU_2022[0].SF}
```


```python
#Dataframe for hitting Statistics
Frame = pd.DataFrame(Dictionary)
Frame = Frame.dropna()
At_Bats = Frame[Frame['AB'] < 50].index

```


```python
#Dropping player with less than 50 ABS
Frame.drop(At_Bats, inplace = True)
```


```python
#wOBA statistic added to frame
Frame['wOBA'] = ((Frame['BB'] * .69) + (Frame['HBP'] * .72) + (Frame['Hits'] - Frame['Double'] - Frame['Triple'] - Frame['HR'])  +(Frame['Double']*1.27) + (Frame['Triple']*1.62) + (Frame['HR']))/(Frame['AB'] + Frame['BB'] + Frame['SF'] + Frame['HBP'])

```


```python
plt.barh(Frame.Player,Frame.wOBA, label = 'SNHU wOBAs')
plt.axvline(x=.320, color = 'red', label = 'Average wOBA')
plt.axvline(x=.370, color = 'Yellow', label = 'Great wOBA')
plt.axvline(x=.400, color = 'Green', label = 'Excellent wOBA')
plt.legend()
plt.title('SNHU 2022 wOBA')
plt.figure(figsize=(10, 10))
plt.show()
```


    
![png](output_20_0.png)
    



    <Figure size 720x720 with 0 Axes>



```python
#Dropping the bottom totals by index
GBG=GBG.drop([GBG.index[51]])
```


```python
#Dropping 2021 index 37 NaN row
GBG2021=GBG2021.drop([GBG2021.index[37]])
```


```python

```
