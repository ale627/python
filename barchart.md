

```python
from IPython.core.display import HTML
def css_styling():
    styles = open("./styles/custom.css", "r").read()
    return HTML(styles)
css_styling()
```






# Fetch the data from BarChart

Here we call the barchart getHistory API.  This is an adaptation of BlackArb's Python [tutorial][tut].

#### Steps:
- import libraries
- set directories and API key
- build URL to call API
- get_minute_data() function
- call the function

#### outputs: symbol, timestamp, tradingDay, open, high, low, close, volume, openInterest

[tut]: http://www.blackarbs.com/blog/how-to-get-free-intraday-stock-data-with-python-and-barcharts-ondemand-api/9/22/2015



```python
# -*- coding: utf-8 -*-

#----------IMPORT LIBRARIES----------#
import time
t0 = time.clock()

import pandas as pd
from pandas.tseries.offsets import BDay

import numpy as np
import datetime as dt

from copy import copy
import warnings
warnings.filterwarnings('ignore',category=pd.io.pytables.PerformanceWarning)
 

    
#----------SET DIRECTORIES----------#
    
project_dir = r'C:\Users/drale/Desktop/python/' 
price_path = project_dir + r'Stock_Price_Data\\'

# header=3 to skip metadata   
spy_components = pd.read_excel(project_dir +\
                             'assets/Watchlist.xls', header=3)
syms = spy_components.Identifier.dropna()
syms = syms.drop(syms.index[-1]).sort_values()


apikey = '99f7186682f41a3bd739984105aef52a'



#----------BUILD URL----------#

def construct_barChart_url(sym, start_date, freq, api_key=apikey):
    '''Function to construct barchart api url'''
    
    url = 'http://marketdata.websol.barchart.com/getHistory.csv?' +\
            'key={}&symbol={}&type={}&startDate={}'.format(api_key, sym, freq, start_date)
    return url




#----------F(X) DEFINITION----------#

def get_minute_data():
    '''Function to Retrieve <= 3 months of minute data for SP500 components'''
    
    # YYYY MM DD HH MM SS
    start = '20171001000000'
    #end = d
    freq = 'minutes'    
    prices = {}
    symbol_count = len(syms)
    
    try:
        for i, sym in enumerate(syms, start=1):
            api_url = construct_barChart_url(sym, start, freq, api_key=apikey)
            try:
                csvfile = pd.read_csv(api_url, parse_dates=['timestamp'])
                csvfile.set_index('timestamp', inplace=True)
                prices[sym] = csvfile
            except:
                continue

            print('{}..[fetched] | {} of {} tickers |'.format(sym, i, symbol_count)) 
    except Exception as e: 
        print(e)
    finally:
        pass
    
    px = pd.Panel.from_dict(prices)
    px.major_axis = px.major_axis.tz_localize('utc').tz_convert('US/Pacific')
    return px



#----------CALL FUNCTION----------#

pxx = get_minute_data()
pxx



# timer
secs      = np.round( ( time.clock()  - t0 ), 4 )
time_secs = "{timeSecs} seconds to run".format(timeSecs = secs)
mins      = np.round( ( (  time.clock() ) -  t0 )  / 60, 4 ) 
time_mins = "| {timeMins} minutes to run".format(timeMins = mins)
hours     = np.round( (  time.clock()  -  t0 )  / 60 / 60, 4 ) 
time_hrs  = "| {timeHrs} hours to run".format(timeHrs = hours)

print( time_secs, time_mins, time_hrs )
```

# Select stock
Choose a ticker to analyze, and test the output.


```python
stock = 'NFLX'

ohlc = pxx[stock]
ohlc.head(1)
```

# 1-min accumulation bars
Where 1 min candlestick closes on highs, flag as accumulated.


```python
#create accum column: for closing on highs, assign 1
ohlc["accum"] = np.where(ohlc["high"] == ohlc["close"], 1, 0)

#list matches
accumulation_bars = ohlc[(ohlc.accum == 1)]
accumulation_bars[["high", "close", "accum"]].tail(3)
```

# Consecutive accumulation
*Add the previous accumulated (or not) to current row*

Measures when two 1-min candlesticks in a row close at highs.


```python
#create conf column adding prev column
ohlc["conf"] = ohlc.accum + ohlc.accum.shift(1)

#where conf is 2, make a table
confirmation = ohlc[(ohlc.conf == 2)]
confirmation[["high", "close", "conf"]].tail(3)
```

## Roll up to daily signal

*sum the consecutive accumulation by day*

#### Return the 3 biggest daily signals


```python
#combine minutes into days
signal = confirmation.resample('D',how='sum')

#sort based on sum(conf)
#signal.sort_values(['double'], ascending=[False])
signal.nlargest(3,'conf')
```


```python
# get intraday min, max
# calc support/resistance
```


```python
try:
    store = pd.HDFStore(price_path + 'Minute_Symbol_Data.h5')
    store['minute_prices'] = pxx
    store.close()
except Exception as e:
    print(e)
finally:
    pass

```


```python
from IPython.display import display, HTML

df = pd.read_hdf(price_path + 'Minute_Symbol_Data.h5')
df
# df.select('AAPL', axis=0)
# display(df)
# pd.to_html(project_dir + 'test.html')
```


```python
#import matplotlib.pyplot as plt   
#from mpl_finance import candlestick_ohlc

%matplotlib inline
%pylab inline
pylab.rcParams['figure.figsize'] = (15, 9)


 
apple["close"].tail(4).plot(grid = True) 


```


```python
import plotly.plotly as py
import plotly.graph_objs as go

from datetime import datetime

df = apple

trace = go.Ohlc(x=df.index,
                open=df.Open,
                high=df.High,
                low=df.Low,
                close=df.Close)
data = [trace]
py.iplot(data, filename='simple_ohlc')
```


```python

```
