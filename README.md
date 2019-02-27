# HW3 for Data Sciecne
by Yancheng Zhu(yz3365) & Shili Wu(sw3302)

Youtube Vedio https://youtu.be/vCck81LXcsI
# Problem Description
I am now playing a game called 'warframe', and there is a trading webside for this game named 'warframe markect'(https://warframe.market/).

Suppose I want to buy an equipment called 'Volt Prime' from another players, as well as looking through the trading information on the website, using a scrapy seems to be a good choice to do so.

Here is the python code.

# Step1 data scrapy

Open the Jupyter notebook and import scrapy modules like 'requests', 'html' and 'bs4'.
```pythonscript
from bs4 import BeautifulSoup
from lxml import html
from lxml import etree
import xml
import requests
```

Next, open the webside for trading a 'Volt Prime'(https://warframe.market/items/volt_prime_set) and 
view the source. Copy the url of the website to creat a request.get.

```pythonscript
url = "https://warframe.market/items/volt_prime_set"
r = requests.get(url).text
s = etree.HTML(r)
trade = s[4].text
```
Then, the trading information is download as a text file.

# Step2 data transform

The text is a json file. We should transform it to a dictionary to read the content. Let's import mudole to deal with it.

```pythonscript   
import json
import pandas as pd
from pandas.io.json import json_normalize
```

```pythonscript
data=json.loads(trade)
data1=data['payload']['orders']
```
So, here data1 is the dictionary I need as trading information. The next step is to read it and generate a table to store the things I am interested in.

Import data science modules.
```pythonscript
import matplotlib
matplotlib.use('Agg')
from datascience import Table
%matplotlib inline
import matplotlib.pyplot as plt
import numpy as np
plt.style.use('fivethirtyeight')
```
Then, choose the information I need. Assume I want to know the player's gameID, the price, the region and so on, creat several arrays to store them.

```pythonscript
name=np.array([])
reputation=np.array([])
platinum=np.array([])
order_type=np.array([])
creat_time=np.array([])
region=np.array([])
platform=np.array([])
status=np.array([])
```
Read the dictionary to input elements. Here I plan to get 400 orders.
```pythonscript
for i in range (0,400):
    name=np.append(name,data1[i]['user']['ingame_name'])
    reputation=np.append(reputation,data1[i]['user']['reputation'])
    platinum=np.append(platinum,data1[i]['platinum'])
    order_type=np.append(order_type,data1[i]['order_type'])
    creat_time=np.append(creat_time,data1[i]['creation_date'][0:10])
    region=np.append(region,data1[i]['region'])
    platform=np.append(platform,data1[i]['platform'])
    status=np.append(status,data1[i]['user']['status'])
```
Finally, we generate the table with these arrays.
```pythonscript
trading=Table().with_columns('gameID',name,'status',status,'order_type',order_type,'platinum',platinum,'region',region,'reputation',reputation,'creat_time',creat_time)

```

# Step3 data analyze and store

Now, I want to know the orders of seller. Thus, we can build a new table to select the rows needed. 
```pythonscript
trading_sell=trading.where(2,'sell').sort(3)
print(trading_sell)
```
or I would like to know the sellers that are online right now.

```pythonscript
trading_sell_online=trading.where(2,'sell').where(1,'online').sort(3)
```


If I want to sell equipment to buyers, I can output a table like this.
```pythonscript
trading_buy=trading.where(2,'buy').sort(3,descending=True)
```
Here are the price distributions of 'seller' and 'buyer'.
```pythonscript
trading_sell.hist(3)
trading_buy.hist(3)
```

At last, store the table as a csv file.

```pythonscript
trading.to_csv('Volt Prime.csv')
```
