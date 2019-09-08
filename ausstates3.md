

```python
!pip install folium
import pandas as pd
pd.set_option('display.max_columns',None)
pd.set_option('display.max_rows', None)
import numpy as np
import folium
import json
import requests
import os
from geopy.geocoders import Nominatim
import requests
from pandas.io.json import json_normalize
import matplotlib.cm as cm
import matplotlib.colors as colors
!pip install bs4
from bs4 import BeautifulSoup
from sklearn.cluster import KMeans
print("Libraries imported")
```

    Requirement already satisfied: folium in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (0.10.0)
    Requirement already satisfied: requests in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (from folium) (2.21.0)
    Requirement already satisfied: branca>=0.3.0 in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (from folium) (0.3.1)
    Requirement already satisfied: jinja2>=2.9 in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (from folium) (2.10)
    Requirement already satisfied: numpy in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (from folium) (1.16.2)
    Requirement already satisfied: idna<2.9,>=2.5 in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (from requests->folium) (2.8)
    Requirement already satisfied: urllib3<1.25,>=1.21.1 in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (from requests->folium) (1.24.1)
    Requirement already satisfied: chardet<3.1.0,>=3.0.2 in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (from requests->folium) (3.0.4)
    Requirement already satisfied: certifi>=2017.4.17 in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (from requests->folium) (2019.3.9)
    Requirement already satisfied: six in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (from branca>=0.3.0->folium) (1.12.0)
    Requirement already satisfied: MarkupSafe>=0.23 in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (from jinja2>=2.9->folium) (1.1.1)
    Requirement already satisfied: bs4 in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (0.0.1)
    Requirement already satisfied: beautifulsoup4 in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (from bs4) (4.7.1)
    Requirement already satisfied: soupsieve>=1.2 in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (from beautifulsoup4->bs4) (1.8)
    Libraries imported
    

# Creation of a Pandas dataframe for Dementia Burden by Australian States


```python
df2 = {
    'States':['ACT','NSW','NT','QUEENSLAND','SOUTH AUSTRALIA','TASMANIA','VICTORIA','WESTERN AUSTRALIA'],
    'Dem_Prev':[5932*100/447116, 149250*100/447116, 1764*100/447116, 84940*100/447116, 37551*100/447116, 11270*100/447116, 114779*100/447116, 41630*100/447116]}
ausdem_df2 = pd.DataFrame(df2, columns = ['States', 'Dem_Prev'])
print(ausdem_df2)
```

                  States   Dem_Prev
    0                ACT   1.326725
    1                NSW  33.380599
    2                 NT   0.394528
    3         QUEENSLAND  18.997307
    4    SOUTH AUSTRALIA   8.398492
    5           TASMANIA   2.520599
    6           VICTORIA  25.670967
    7  WESTERN AUSTRALIA   9.310783
    

# General view of the map of Australia


```python
!pip install plotly
aus_map = folium.Map(location = [-25.734968, 134.489563], zoom_start = 4)
aus_map
```

    Requirement already satisfied: plotly in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (4.1.0)
    Requirement already satisfied: six in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (from plotly) (1.12.0)
    Requirement already satisfied: retrying>=1.3.3 in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (from plotly) (1.3.3)
    




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2Nkbi5naXRoYWNrLmNvbS9weXRob24tdmlzdWFsaXphdGlvbi9mb2xpdW0vbWFzdGVyL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5jc3MiLz4KICAgIDxzdHlsZT5odG1sLCBib2R5IHt3aWR0aDogMTAwJTtoZWlnaHQ6IDEwMCU7bWFyZ2luOiAwO3BhZGRpbmc6IDA7fTwvc3R5bGU+CiAgICA8c3R5bGU+I21hcCB7cG9zaXRpb246YWJzb2x1dGU7dG9wOjA7Ym90dG9tOjA7cmlnaHQ6MDtsZWZ0OjA7fTwvc3R5bGU+CiAgICAKICAgICAgICAgICAgPG1ldGEgbmFtZT0idmlld3BvcnQiIGNvbnRlbnQ9IndpZHRoPWRldmljZS13aWR0aCwKICAgICAgICAgICAgICAgIGluaXRpYWwtc2NhbGU9MS4wLCBtYXhpbXVtLXNjYWxlPTEuMCwgdXNlci1zY2FsYWJsZT1ubyIgLz4KICAgICAgICAgICAgPHN0eWxlPgogICAgICAgICAgICAgICAgI21hcF9iOWU5NjljNTUxODQ0ODIwOTEzODg3ZGQ5OTRjZjZiYSB7CiAgICAgICAgICAgICAgICAgICAgcG9zaXRpb246IHJlbGF0aXZlOwogICAgICAgICAgICAgICAgICAgIHdpZHRoOiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgbGVmdDogMC4wJTsKICAgICAgICAgICAgICAgICAgICB0b3A6IDAuMCU7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgIDwvc3R5bGU+CiAgICAgICAgCjwvaGVhZD4KPGJvZHk+ICAgIAogICAgCiAgICAgICAgICAgIDxkaXYgY2xhc3M9ImZvbGl1bS1tYXAiIGlkPSJtYXBfYjllOTY5YzU1MTg0NDgyMDkxMzg4N2RkOTk0Y2Y2YmEiID48L2Rpdj4KICAgICAgICAKPC9ib2R5Pgo8c2NyaXB0PiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFwX2I5ZTk2OWM1NTE4NDQ4MjA5MTM4ODdkZDk5NGNmNmJhID0gTC5tYXAoCiAgICAgICAgICAgICAgICAibWFwX2I5ZTk2OWM1NTE4NDQ4MjA5MTM4ODdkZDk5NGNmNmJhIiwKICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICBjZW50ZXI6IFstMjUuNzM0OTY4LCAxMzQuNDg5NTYzXSwKICAgICAgICAgICAgICAgICAgICBjcnM6IEwuQ1JTLkVQU0czODU3LAogICAgICAgICAgICAgICAgICAgIHpvb206IDQsCiAgICAgICAgICAgICAgICAgICAgem9vbUNvbnRyb2w6IHRydWUsCiAgICAgICAgICAgICAgICAgICAgcHJlZmVyQ2FudmFzOiBmYWxzZSwKICAgICAgICAgICAgICAgIH0KICAgICAgICAgICAgKTsKCiAgICAgICAgICAgIAoKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgdGlsZV9sYXllcl82NTU4MzZjMTRhN2U0YTIwOTE5MWNhN2VlN2IxOTJjNyA9IEwudGlsZUxheWVyKAogICAgICAgICAgICAgICAgImh0dHBzOi8ve3N9LnRpbGUub3BlbnN0cmVldG1hcC5vcmcve3p9L3t4fS97eX0ucG5nIiwKICAgICAgICAgICAgICAgIHsiYXR0cmlidXRpb24iOiAiRGF0YSBieSBcdTAwMjZjb3B5OyBcdTAwM2NhIGhyZWY9XCJodHRwOi8vb3BlbnN0cmVldG1hcC5vcmdcIlx1MDAzZU9wZW5TdHJlZXRNYXBcdTAwM2MvYVx1MDAzZSwgdW5kZXIgXHUwMDNjYSBocmVmPVwiaHR0cDovL3d3dy5vcGVuc3RyZWV0bWFwLm9yZy9jb3B5cmlnaHRcIlx1MDAzZU9EYkxcdTAwM2MvYVx1MDAzZS4iLCAiZGV0ZWN0UmV0aW5hIjogZmFsc2UsICJtYXhOYXRpdmVab29tIjogMTgsICJtYXhab29tIjogMTgsICJtaW5ab29tIjogMCwgIm5vV3JhcCI6IGZhbHNlLCAib3BhY2l0eSI6IDEsICJzdWJkb21haW5zIjogImFiYyIsICJ0bXMiOiBmYWxzZX0KICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjllOTY5YzU1MTg0NDgyMDkxMzg4N2RkOTk0Y2Y2YmEpOwogICAgICAgIAo8L3NjcmlwdD4=" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




```python
import plotly.graph_objects as go
```

# Pandas Dataframe for the geographical coordinates of Australian states plus dementia frequency plus general hospital data


```python
import folium   
import pandas as pd
import json
aus_data = pd.DataFrame({
    'lat':[-35.473469, -31.840233, -19.491411, -20.917574, -30.000233, -41.640079, -37.020100, -25.760321],
    'lon':[149.012375, 145.612793, 132.550964, 142.702789, 136.209152, 146.315918, 144.964600, 122.805176],
    'name':['ACT', 'NSW', 'NT', 'QUEENSLAND', 'SOUTH AUSTRALIA', 'TASMANIA', 'VICTORIA', 'WESTERN AUSTRALIA'],
    'value':[5932, 149250, 1764, 84940, 37551, 11270, 114779, 41630]
})
aus_data
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>lat</th>
      <th>lon</th>
      <th>name</th>
      <th>value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-35.473469</td>
      <td>149.012375</td>
      <td>ACT</td>
      <td>5932</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-31.840233</td>
      <td>145.612793</td>
      <td>NSW</td>
      <td>149250</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-19.491411</td>
      <td>132.550964</td>
      <td>NT</td>
      <td>1764</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-20.917574</td>
      <td>142.702789</td>
      <td>QUEENSLAND</td>
      <td>84940</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-30.000233</td>
      <td>136.209152</td>
      <td>SOUTH AUSTRALIA</td>
      <td>37551</td>
    </tr>
    <tr>
      <th>5</th>
      <td>-41.640079</td>
      <td>146.315918</td>
      <td>TASMANIA</td>
      <td>11270</td>
    </tr>
    <tr>
      <th>6</th>
      <td>-37.020100</td>
      <td>144.964600</td>
      <td>VICTORIA</td>
      <td>114779</td>
    </tr>
    <tr>
      <th>7</th>
      <td>-25.760321</td>
      <td>122.805176</td>
      <td>WESTERN AUSTRALIA</td>
      <td>41630</td>
    </tr>
  </tbody>
</table>
</div>




```python
hosp_data = pd.DataFrame({
    'name':['ACT', 'NSW', 'NT', 'QUEENSLAND', 'SOUTH AUSTRALIA', 'TASMANIA', 'VICTORIA', 'WESTERN AUSTRALIA'],
    'pubhosp':[3, 214, 5, 119, 75, 22, 148, 87],
    'pubpsych':[0, 8, 0, 4, 2, 1, 3, 4],
    'privhosp':[0, 205, 0, 109, 56, 0, 169, 62],
    'total':[3, 427, 5, 232, 133, 23, 320, 153]
})
hosp_data
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>pubhosp</th>
      <th>pubpsych</th>
      <th>privhosp</th>
      <th>total</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ACT</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NSW</td>
      <td>214</td>
      <td>8</td>
      <td>205</td>
      <td>427</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NT</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
      <td>5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>QUEENSLAND</td>
      <td>119</td>
      <td>4</td>
      <td>109</td>
      <td>232</td>
    </tr>
    <tr>
      <th>4</th>
      <td>SOUTH AUSTRALIA</td>
      <td>75</td>
      <td>2</td>
      <td>56</td>
      <td>133</td>
    </tr>
    <tr>
      <th>5</th>
      <td>TASMANIA</td>
      <td>22</td>
      <td>1</td>
      <td>0</td>
      <td>23</td>
    </tr>
    <tr>
      <th>6</th>
      <td>VICTORIA</td>
      <td>148</td>
      <td>3</td>
      <td>169</td>
      <td>320</td>
    </tr>
    <tr>
      <th>7</th>
      <td>WESTERN AUSTRALIA</td>
      <td>87</td>
      <td>4</td>
      <td>62</td>
      <td>153</td>
    </tr>
  </tbody>
</table>
</div>



# Merging dataframes of dementia burden, geographical coordinates and hospital data


```python
aus_gp = pd.merge(ausdem_df2, aus_data, how = 'left', left_on = 'States', right_on = 'name')
aus_gp.drop(labels = 'name', axis = 1, inplace = True)
aus_gp
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>States</th>
      <th>Dem_Prev</th>
      <th>lat</th>
      <th>lon</th>
      <th>value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ACT</td>
      <td>1.326725</td>
      <td>-35.473469</td>
      <td>149.012375</td>
      <td>5932</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NSW</td>
      <td>33.380599</td>
      <td>-31.840233</td>
      <td>145.612793</td>
      <td>149250</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NT</td>
      <td>0.394528</td>
      <td>-19.491411</td>
      <td>132.550964</td>
      <td>1764</td>
    </tr>
    <tr>
      <th>3</th>
      <td>QUEENSLAND</td>
      <td>18.997307</td>
      <td>-20.917574</td>
      <td>142.702789</td>
      <td>84940</td>
    </tr>
    <tr>
      <th>4</th>
      <td>SOUTH AUSTRALIA</td>
      <td>8.398492</td>
      <td>-30.000233</td>
      <td>136.209152</td>
      <td>37551</td>
    </tr>
    <tr>
      <th>5</th>
      <td>TASMANIA</td>
      <td>2.520599</td>
      <td>-41.640079</td>
      <td>146.315918</td>
      <td>11270</td>
    </tr>
    <tr>
      <th>6</th>
      <td>VICTORIA</td>
      <td>25.670967</td>
      <td>-37.020100</td>
      <td>144.964600</td>
      <td>114779</td>
    </tr>
    <tr>
      <th>7</th>
      <td>WESTERN AUSTRALIA</td>
      <td>9.310783</td>
      <td>-25.760321</td>
      <td>122.805176</td>
      <td>41630</td>
    </tr>
  </tbody>
</table>
</div>




```python
aus_gp2 = pd.merge(aus_gp, hosp_data, how = 'left', left_on = 'States', right_on = 'name', validate = "1:1")
aus_gp2.drop(labels = 'name', axis = 1, inplace = True)
aus_gp2
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>States</th>
      <th>Dem_Prev</th>
      <th>lat</th>
      <th>lon</th>
      <th>value</th>
      <th>pubhosp</th>
      <th>pubpsych</th>
      <th>privhosp</th>
      <th>total</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ACT</td>
      <td>1.326725</td>
      <td>-35.473469</td>
      <td>149.012375</td>
      <td>5932</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>NSW</td>
      <td>33.380599</td>
      <td>-31.840233</td>
      <td>145.612793</td>
      <td>149250</td>
      <td>214</td>
      <td>8</td>
      <td>205</td>
      <td>427</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NT</td>
      <td>0.394528</td>
      <td>-19.491411</td>
      <td>132.550964</td>
      <td>1764</td>
      <td>5</td>
      <td>0</td>
      <td>0</td>
      <td>5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>QUEENSLAND</td>
      <td>18.997307</td>
      <td>-20.917574</td>
      <td>142.702789</td>
      <td>84940</td>
      <td>119</td>
      <td>4</td>
      <td>109</td>
      <td>232</td>
    </tr>
    <tr>
      <th>4</th>
      <td>SOUTH AUSTRALIA</td>
      <td>8.398492</td>
      <td>-30.000233</td>
      <td>136.209152</td>
      <td>37551</td>
      <td>75</td>
      <td>2</td>
      <td>56</td>
      <td>133</td>
    </tr>
    <tr>
      <th>5</th>
      <td>TASMANIA</td>
      <td>2.520599</td>
      <td>-41.640079</td>
      <td>146.315918</td>
      <td>11270</td>
      <td>22</td>
      <td>1</td>
      <td>0</td>
      <td>23</td>
    </tr>
    <tr>
      <th>6</th>
      <td>VICTORIA</td>
      <td>25.670967</td>
      <td>-37.020100</td>
      <td>144.964600</td>
      <td>114779</td>
      <td>148</td>
      <td>3</td>
      <td>169</td>
      <td>320</td>
    </tr>
    <tr>
      <th>7</th>
      <td>WESTERN AUSTRALIA</td>
      <td>9.310783</td>
      <td>-25.760321</td>
      <td>122.805176</td>
      <td>41630</td>
      <td>87</td>
      <td>4</td>
      <td>62</td>
      <td>153</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Pre-processing

aus_gp2.dtypes
```




    States       object
    Dem_Prev    float64
    lat         float64
    lon         float64
    value         int64
    pubhosp       int64
    pubpsych      int64
    privhosp      int64
    total         int64
    dtype: object



# Descriptive data and visualizations


```python
aus_gp2['Dem_Prev'].plot(kind = 'box', figsize = (8, 6))
plt.title('Box plot of Dementia Burden')
plt.ylabel('Percentage (%)')
```




    Text(0, 0.5, 'Percentage (%)')




![png](output_14_1.png)



```python
# Descriptive data

aus_gp2.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Dem_Prev</th>
      <th>lat</th>
      <th>lon</th>
      <th>value</th>
      <th>pubhosp</th>
      <th>pubpsych</th>
      <th>privhosp</th>
      <th>total</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>8.000000</td>
      <td>8.000000</td>
      <td>8.000000</td>
      <td>8.000000</td>
      <td>8.000000</td>
      <td>8.000000</td>
      <td>8.000000</td>
      <td>8.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>12.500000</td>
      <td>-30.267928</td>
      <td>140.021721</td>
      <td>55889.500000</td>
      <td>84.125000</td>
      <td>2.750000</td>
      <td>75.125000</td>
      <td>162.000000</td>
    </tr>
    <tr>
      <th>std</th>
      <td>12.245438</td>
      <td>7.823409</td>
      <td>8.864748</td>
      <td>54751.311765</td>
      <td>74.600723</td>
      <td>2.659216</td>
      <td>79.549513</td>
      <td>155.958786</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.394528</td>
      <td>-41.640079</td>
      <td>122.805176</td>
      <td>1764.000000</td>
      <td>3.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>3.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2.222130</td>
      <td>-35.860127</td>
      <td>135.294605</td>
      <td>9935.500000</td>
      <td>17.750000</td>
      <td>0.750000</td>
      <td>0.000000</td>
      <td>18.500000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>8.854637</td>
      <td>-30.920233</td>
      <td>143.833694</td>
      <td>39590.500000</td>
      <td>81.000000</td>
      <td>2.500000</td>
      <td>59.000000</td>
      <td>143.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>20.665722</td>
      <td>-24.549634</td>
      <td>145.788574</td>
      <td>92399.750000</td>
      <td>126.250000</td>
      <td>4.000000</td>
      <td>124.000000</td>
      <td>254.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>33.380599</td>
      <td>-19.491411</td>
      <td>149.012375</td>
      <td>149250.000000</td>
      <td>214.000000</td>
      <td>8.000000</td>
      <td>205.000000</td>
      <td>427.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
aus_gp2.describe(include=['object'])
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>States</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>8</td>
    </tr>
    <tr>
      <th>unique</th>
      <td>8</td>
    </tr>
    <tr>
      <th>top</th>
      <td>WESTERN AUSTRALIA</td>
    </tr>
    <tr>
      <th>freq</th>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
import seaborn as sns
```

#Data Visualization


```python
import matplotlib.pyplot as plt
y = aus_gp2["Dem_Prev"]
x = aus_gp2["pubhosp"]
plt.scatter(x, y)
plt.title("Scatterplot of Dementia Prevalence vs Number of Public Hospitals")
plt.xlabel("Number of Public Hospitals")
plt.ylabel("Prevalence of Dementia")
```




    Text(0, 0.5, 'Prevalence of Dementia')




![png](output_19_1.png)



```python
y = aus_gp2["Dem_Prev"]
x = aus_gp2["pubpsych"]
plt.scatter(x, y)
plt.title("Scatterplot of Dementia Prevalence vs Number of Psychiatric Hospitals")
plt.xlabel("Number of Psychiatric Hospitals")
plt.ylabel("Prevalence of Dementia")
```




    Text(0, 0.5, 'Prevalence of Dementia')




![png](output_20_1.png)



```python
sns.regplot(x = "pubhosp", y = "Dem_Prev", data = aus_gp2)
plt.title("Graph of Dementia Burden vs Number of Public Hospitals")
plt.xlabel("Number of Public Hospitals")
plt.ylabel("Dementia Burden")
plt.show()
```


![png](output_21_0.png)



```python
sns.regplot(x = "pubpsych", y = "Dem_Prev", data = aus_gp2)
plt.title("Graph of Dementia Burden vs Number of Psychiatric Hospitals")
plt.xlabel("Number of Psychiatric Hospitals")
plt.ylabel("Dementia Burden")
plt.show()
```


![png](output_22_0.png)



```python
sns.regplot(x = "privhosp", y = "Dem_Prev", data = aus_gp2)
plt.title("Graph of Dementia Burden vs Number of Private Hospitals")
plt.xlabel("Number of Private Hospitals")
plt.ylabel("Dementia Burden")
plt.show()
```


![png](output_23_0.png)



```python
sns.regplot(x = "total", y = "Dem_Prev", data = aus_gp2)
plt.title("Graph of Dementia Prevalence vs Total Number of Hospitals")
plt.xlabel("Total Number of Hospitals")
plt.ylabel("Prevalence of Dementia")
plt.show()
```


![png](output_24_0.png)

# Correlation matrix including hospitals, dementia burden and geographical coordinates and, heatmap

```python
aus_gp2[['Dem_Prev', 'pubhosp', 'privhosp', 'pubpsych', 'total', 'lat', 'lon']].corr()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Dem_Prev</th>
      <th>pubhosp</th>
      <th>privhosp</th>
      <th>pubpsych</th>
      <th>total</th>
      <th>lat</th>
      <th>lon</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Dem_Prev</th>
      <td>1.000000</td>
      <td>0.983992</td>
      <td>0.995362</td>
      <td>0.884466</td>
      <td>0.993462</td>
      <td>-0.052623</td>
      <td>0.259231</td>
    </tr>
    <tr>
      <th>pubhosp</th>
      <td>0.983992</td>
      <td>1.000000</td>
      <td>0.984274</td>
      <td>0.937057</td>
      <td>0.996360</td>
      <td>-0.005577</td>
      <td>0.127752</td>
    </tr>
    <tr>
      <th>privhosp</th>
      <td>0.995362</td>
      <td>0.984274</td>
      <td>1.000000</td>
      <td>0.871334</td>
      <td>0.995738</td>
      <td>-0.034383</td>
      <td>0.201127</td>
    </tr>
    <tr>
      <th>pubpsych</th>
      <td>0.884466</td>
      <td>0.937057</td>
      <td>0.871334</td>
      <td>1.000000</td>
      <td>0.909718</td>
      <td>0.080754</td>
      <td>0.000511</td>
    </tr>
    <tr>
      <th>total</th>
      <td>0.993462</td>
      <td>0.996360</td>
      <td>0.995738</td>
      <td>0.909718</td>
      <td>1.000000</td>
      <td>-0.018829</td>
      <td>0.163705</td>
    </tr>
    <tr>
      <th>lat</th>
      <td>-0.052623</td>
      <td>-0.005577</td>
      <td>-0.034383</td>
      <td>0.080754</td>
      <td>-0.018829</td>
      <td>1.000000</td>
      <td>-0.606861</td>
    </tr>
    <tr>
      <th>lon</th>
      <td>0.259231</td>
      <td>0.127752</td>
      <td>0.201127</td>
      <td>0.000511</td>
      <td>0.163705</td>
      <td>-0.606861</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python

cor = aus_gp2.corr()
sns.heatmap(cor, square = True)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x231b1980240>




![png](output_27_1.png)


# Linear regression (simple and multiple linear regression) using Scikit-Learn and Scipy


```python
import scipy
from scipy import stats
from scipy.stats import linregress
```


```python

linregress(aus_gp2["pubhosp"], aus_gp2["Dem_Prev"])
```




    LinregressResult(slope=0.16151864982780498, intercept=-1.087756416764094, rvalue=0.9839916069670939, pvalue=1.0133379289318787e-05, stderr=0.01194261524289986)




```python
linregress(aus_gp2["pubpsych"], aus_gp2["Dem_Prev"])
```




    LinregressResult(slope=4.072881545394272, intercept=1.2995757501657526, rvalue=0.8844657936818925, pvalue=0.0035290652990349813, stderr=0.8771917423499831)




```python
linregress(aus_gp2["privhosp"], aus_gp2["Dem_Prev"])
```




    LinregressResult(slope=0.15322089396826152, intercept=0.9892803406343535, rvalue=0.9953623265663488, pvalue=2.485013124576096e-07, stderr=0.0060453595219430905)




```python
linregress(aus_gp2["total"], aus_gp2["Dem_Prev"])
```




    LinregressResult(slope=0.07800376053087549, intercept=-0.1366092060018289, rvalue=0.9934615614301954, pvalue=6.953925056560325e-07, stderr=0.0036595678515175616)




```python
from scipy.stats.stats import pearsonr
```


```python
pearson_coef, p_value = stats.pearsonr(aus_gp2['pubhosp'], aus_gp2['Dem_Prev'])
print("The Pearson correlation coefficient is", pearson_coef, "with a P-value of P =", p_value)
```

    The Pearson correlation coefficient is 0.9839916069670939 with a P-value of P = 1.0133379289318792e-05
    


```python
from sklearn.linear_model import LinearRegression
lm = LinearRegression()
```


```python
X = aus_gp2[['pubhosp']]
Y = aus_gp2[['Dem_Prev']]
lm.fit(X, Y)
```




    LinearRegression(copy_X=True, fit_intercept=True, n_jobs=None,
             normalize=False)




```python
lm.coef_
```




    array([[0.16151865]])




```python
X = aus_gp2[['pubhosp', 'lat', 'lon']]
Y = aus_gp2[['Dem_Prev']]
lm.fit(X, Y)
```




    LinearRegression(copy_X=True, fit_intercept=True, n_jobs=None,
             normalize=False)




```python
lm.coef_
```




    array([[0.15799866, 0.08780633, 0.2352561 ]])




```python
print("The R-square is:", lm.score(X, Y))
```

    The R-square is: 0.9883353768586333
    


```python
X = aus_gp2[['privhosp']]
Y = aus_gp2[['Dem_Prev']]
lm.fit(X, Y)
```




    LinearRegression(copy_X=True, fit_intercept=True, n_jobs=None,
             normalize=False)




```python
lm.coef_
```




    array([[0.15322089]])




```python
X = aus_gp2[['privhosp', 'lat', 'lon']]
Y = aus_gp2[['Dem_Prev']]
lm.fit(X, Y)
```




    LinearRegression(copy_X=True, fit_intercept=True, n_jobs=None,
             normalize=False)




```python
lm.coef_
```




    array([[0.15089857, 0.04645849, 0.11062449]])




```python
print("The R-square is:", lm.score(X, Y))
```

    The R-square is: 0.9949279085600867
    


```python
print("The R-square is:", lm.score(X, Y))
```

    The R-square is: 0.9949279085600867
    


```python
X = aus_gp2[['pubpsych']]
Y = aus_gp2[['Dem_Prev']]
lm.fit(X, Y)
```




    LinearRegression(copy_X=True, fit_intercept=True, n_jobs=None,
             normalize=False)




```python
lm.coef_
```




    array([[4.07288155]])




```python
X = aus_gp2[['pubpsych', 'lat', 'lon']]
Y = aus_gp2[['Dem_Prev']]
lm.fit(X, Y)
```




    LinearRegression(copy_X=True, fit_intercept=True, n_jobs=None,
             normalize=False)




```python
print("The R-square is:", lm.score(X, Y))
```

    The R-square is: 0.8509887036485619
    


```python
lm.coef_
```




    array([[4.05256343, 0.08263982, 0.40172993]])




```python
X = aus_gp2[['total']]
Y = aus_gp2[['Dem_Prev']]
lm.fit(X, Y)
```




    LinearRegression(copy_X=True, fit_intercept=True, n_jobs=None,
             normalize=False)




```python
lm.coef_
```




    array([[0.07800376]])




```python
X = aus_gp2[['total', 'lat', 'lon']]
Y = aus_gp2[['Dem_Prev']]
lm.fit(X, Y)
```




    LinearRegression(copy_X=True, fit_intercept=True, n_jobs=None,
             normalize=False)




```python
lm.coef_
```




    array([[0.07645763, 0.065135  , 0.17277145]])




```python
from sklearn.metrics import r2_score
```


```python
X = aus_gp2[['total']]
Y = aus_gp2[['Dem_Prev']]
lm.fit(X, Y)
```




    LinearRegression(copy_X=True, fit_intercept=True, n_jobs=None,
             normalize=False)




```python
print("The R-square is:", lm.score(X, Y))
```

    The R-square is: 0.986965874039322
    


```python
X = aus_gp2[['pubhosp']]
Y = aus_gp2[['Dem_Prev']]
lm.fit(X, Y)
```




    LinearRegression(copy_X=True, fit_intercept=True, n_jobs=None,
             normalize=False)




```python
print("The R-square is:", lm.score(X, Y))
```

    The R-square is: 0.9682394825816838
    


```python
X = aus_gp2[['pubpsych']]
Y = aus_gp2[['Dem_Prev']]
lm.fit(X, Y)
```




    LinearRegression(copy_X=True, fit_intercept=True, n_jobs=None,
             normalize=False)




```python
print("The R-square is:", lm.score(X, Y))
```

    The R-square is: 0.7822797401933401
    


```python
X = aus_gp2[['privhosp']]
Y = aus_gp2[['Dem_Prev']]
lm.fit(X, Y)
```




    LinearRegression(copy_X=True, fit_intercept=True, n_jobs=None,
             normalize=False)




```python
print("The R-square is:", lm.score(X, Y))
```

    The R-square is: 0.9907461611475749
    


```python
X = aus_gp2[['lat']]
Y = aus_gp2[['Dem_Prev']]
lm.fit(X, Y)
lm.coef_
print("The R-square is>", lm.score(X, Y))
```

    The R-square is> 0.002769198391169292
    


```python
lm.coef_
```




    array([[-0.0823674]])




```python
X = aus_gp2[['lon']]
Y = aus_gp2[['Dem_Prev']]
lm.fit(X, Y)
lm.coef_
print("The R-square is>", lm.score(X, Y))
```

    The R-square is> 0.0672005628472816
    


```python
lm.coef_
```




    array([[0.35809178]])




```python
linregress(aus_gp2['lat'], aus_gp['Dem_Prev'])
```




    LinregressResult(slope=-0.08236739564069347, intercept=10.006909640383675, rvalue=-0.05262317351860496, pvalue=0.9015135533349156, stderr=0.6381172955272034)




```python
linregress(aus_gp2['lon'], aus_gp['Dem_Prev'])
```




    LinregressResult(slope=0.35809178413369075, intercept=-37.6406278455984, rvalue=0.2592307135493044, pvalue=0.5352789754040121, stderr=0.5446611693400937)




```python
# Regression plots for the association of geographical coordinates with dementia burden
```


```python
sns.regplot(x = "lat", y = "Dem_Prev", data = aus_gp2)
plt.title("Graph of Dementia Burden vs State Latitude")
plt.xlabel("State Latitude")
plt.ylabel("Dementia Burden")
plt.show()
```


![png](output_73_0.png)



```python
sns.regplot(x = "lon", y = "Dem_Prev", data = aus_gp2)
plt.title("Graph of Dementia Burden vs State Longitude")
plt.xlabel("State Longitude")
plt.ylabel("Dementia Burden")
plt.show()
```


![png](output_74_0.png)



```python
# More visualizations: Bar graph

import matplotlib as mpl
aus_gp2.groupby('States').mean()['Dem_Prev'].plot(kind = 'bar')
plt.title("Graph of Dementia Burden(%) vs States")
plt.xlabel("Australian States")
plt.ylabel("Dementia Burden(%)")
```




    Text(0, 0.5, 'Dementia Burden(%)')




![png](output_75_1.png)


# Selected city geographical coordinates


```python
# Creating a pandas dataframe from a csv file

city_data = pd.read_csv(r'C:\Users\c3273214\Documents\AusCities3.csv')
city_data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>State</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Canberra</td>
      <td>-35.282001</td>
      <td>149.128998</td>
      <td>ACT</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Sunshine Coast</td>
      <td>-26.650000</td>
      <td>153.066666</td>
      <td>QLD</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Gold Coast</td>
      <td>-28.016666</td>
      <td>153.399994</td>
      <td>QLD</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Melbourne</td>
      <td>-37.840935</td>
      <td>144.946457</td>
      <td>VIC</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Adelaide</td>
      <td>-34.921230</td>
      <td>138.599503</td>
      <td>SA</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Grouping cities data by State

city_datagp = city_data.groupby(["State", "Latitude", "Longitude"], as_index = False).agg(lambda x:",".join(x))
city_datagp
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>State</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>City</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ACT</td>
      <td>-35.343784</td>
      <td>149.082977</td>
      <td>Phillip</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ACT</td>
      <td>-35.282001</td>
      <td>149.128998</td>
      <td>Canberra</td>
    </tr>
    <tr>
      <th>2</th>
      <td>NSW</td>
      <td>-36.080780</td>
      <td>146.916473</td>
      <td>Albury</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NSW</td>
      <td>-34.425072</td>
      <td>150.893143</td>
      <td>Wollongong</td>
    </tr>
    <tr>
      <th>4</th>
      <td>NSW</td>
      <td>-33.917290</td>
      <td>151.035889</td>
      <td>Bankstown</td>
    </tr>
    <tr>
      <th>5</th>
      <td>NSW</td>
      <td>-33.865143</td>
      <td>151.209900</td>
      <td>Sydney</td>
    </tr>
    <tr>
      <th>6</th>
      <td>NSW</td>
      <td>-33.807690</td>
      <td>150.987274</td>
      <td>Westmead</td>
    </tr>
    <tr>
      <th>7</th>
      <td>NSW</td>
      <td>-33.683212</td>
      <td>151.224396</td>
      <td>Terrey Hills</td>
    </tr>
    <tr>
      <th>8</th>
      <td>NSW</td>
      <td>-33.425018</td>
      <td>151.342224</td>
      <td>Gosford</td>
    </tr>
    <tr>
      <th>9</th>
      <td>NSW</td>
      <td>-33.283577</td>
      <td>149.101273</td>
      <td>Orange</td>
    </tr>
    <tr>
      <th>10</th>
      <td>NSW</td>
      <td>-30.296276</td>
      <td>153.114136</td>
      <td>Coffs Harbour</td>
    </tr>
    <tr>
      <th>11</th>
      <td>NT</td>
      <td>-12.462827</td>
      <td>130.841782</td>
      <td>Darwin</td>
    </tr>
    <tr>
      <th>12</th>
      <td>QLD</td>
      <td>-28.016666</td>
      <td>153.399994</td>
      <td>Gold Coast</td>
    </tr>
    <tr>
      <th>13</th>
      <td>QLD</td>
      <td>-27.529953</td>
      <td>152.407181</td>
      <td>Glenore Grove</td>
    </tr>
    <tr>
      <th>14</th>
      <td>QLD</td>
      <td>-27.470125</td>
      <td>153.021072</td>
      <td>Brisbane</td>
    </tr>
    <tr>
      <th>15</th>
      <td>QLD</td>
      <td>-26.650000</td>
      <td>153.066666</td>
      <td>Sunshine Coast</td>
    </tr>
    <tr>
      <th>16</th>
      <td>QLD</td>
      <td>-23.843138</td>
      <td>151.268356</td>
      <td>Gladstone</td>
    </tr>
    <tr>
      <th>17</th>
      <td>QLD</td>
      <td>-19.258965</td>
      <td>146.816956</td>
      <td>Townsville City</td>
    </tr>
    <tr>
      <th>18</th>
      <td>QLD</td>
      <td>-16.925491</td>
      <td>145.754120</td>
      <td>Cairns City</td>
    </tr>
    <tr>
      <th>19</th>
      <td>SA</td>
      <td>-37.824429</td>
      <td>140.783783</td>
      <td>Mount Gambier</td>
    </tr>
    <tr>
      <th>20</th>
      <td>SA</td>
      <td>-34.921230</td>
      <td>138.599503</td>
      <td>Adelaide</td>
    </tr>
    <tr>
      <th>21</th>
      <td>SA</td>
      <td>-34.906101</td>
      <td>138.593903</td>
      <td>North Adelaide</td>
    </tr>
    <tr>
      <th>22</th>
      <td>TAS</td>
      <td>-41.429825</td>
      <td>147.157135</td>
      <td>Launceston</td>
    </tr>
    <tr>
      <th>23</th>
      <td>VIC</td>
      <td>-37.840935</td>
      <td>144.946457</td>
      <td>Melbourne</td>
    </tr>
    <tr>
      <th>24</th>
      <td>VIC</td>
      <td>-37.649967</td>
      <td>144.880600</td>
      <td>Ziyou Today</td>
    </tr>
    <tr>
      <th>25</th>
      <td>VIC</td>
      <td>-36.757786</td>
      <td>144.278702</td>
      <td>Bendigo</td>
    </tr>
    <tr>
      <th>26</th>
      <td>VIC</td>
      <td>-34.206841</td>
      <td>142.136490</td>
      <td>Mildura</td>
    </tr>
    <tr>
      <th>27</th>
      <td>WA</td>
      <td>-31.953512</td>
      <td>115.857048</td>
      <td>Perth</td>
    </tr>
  </tbody>
</table>
</div>



# Location of cities in Australia plus hospital data per state


```python
address = 'Australia'
geolocator = Nominatim(user_agent = 'my_application')
location = geolocator.geocode(address)
latitude = location.latitude
longitude = location.longitude
print("The geographical coordinates of Australia are {},{}.".format(latitude, longitude))
```

    The geographical coordinates of Australia are -24.7761086,134.755.
    


```python
!pip install plotly
aus_map2 = folium.Map(location = [latitude, longitude], zoom_start = 4)
for lat, lon, city, state in zip(city_data['Latitude'], city_data['Longitude'], city_data['City'], city_data['State']):
    label = '{}'.format(city, state)
    label = folium.Popup(label, parse_html = True)
    folium.CircleMarker(
       [lat, lon],
       radius = 5,
       color = 'blue',
       fill = True,
       fill_color = '#3186cc',
       fill_opacity = 0.7,
       parse_html = False).add_to(aus_map2)
aus_map2       
```

    Requirement already satisfied: plotly in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (4.1.0)
    Requirement already satisfied: retrying>=1.3.3 in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (from plotly) (1.3.3)
    Requirement already satisfied: six in c:\users\c3273214\appdata\local\continuum\anaconda3\lib\site-packages (from plotly) (1.12.0)
    




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2Nkbi5naXRoYWNrLmNvbS9weXRob24tdmlzdWFsaXphdGlvbi9mb2xpdW0vbWFzdGVyL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5jc3MiLz4KICAgIDxzdHlsZT5odG1sLCBib2R5IHt3aWR0aDogMTAwJTtoZWlnaHQ6IDEwMCU7bWFyZ2luOiAwO3BhZGRpbmc6IDA7fTwvc3R5bGU+CiAgICA8c3R5bGU+I21hcCB7cG9zaXRpb246YWJzb2x1dGU7dG9wOjA7Ym90dG9tOjA7cmlnaHQ6MDtsZWZ0OjA7fTwvc3R5bGU+CiAgICAKICAgICAgICAgICAgPG1ldGEgbmFtZT0idmlld3BvcnQiIGNvbnRlbnQ9IndpZHRoPWRldmljZS13aWR0aCwKICAgICAgICAgICAgICAgIGluaXRpYWwtc2NhbGU9MS4wLCBtYXhpbXVtLXNjYWxlPTEuMCwgdXNlci1zY2FsYWJsZT1ubyIgLz4KICAgICAgICAgICAgPHN0eWxlPgogICAgICAgICAgICAgICAgI21hcF9hMDhjMmM5ZWJiOTk0NGM3YjRmZGI4MTU5M2ZjMzBhNiB7CiAgICAgICAgICAgICAgICAgICAgcG9zaXRpb246IHJlbGF0aXZlOwogICAgICAgICAgICAgICAgICAgIHdpZHRoOiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgbGVmdDogMC4wJTsKICAgICAgICAgICAgICAgICAgICB0b3A6IDAuMCU7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgIDwvc3R5bGU+CiAgICAgICAgCjwvaGVhZD4KPGJvZHk+ICAgIAogICAgCiAgICAgICAgICAgIDxkaXYgY2xhc3M9ImZvbGl1bS1tYXAiIGlkPSJtYXBfYTA4YzJjOWViYjk5NDRjN2I0ZmRiODE1OTNmYzMwYTYiID48L2Rpdj4KICAgICAgICAKPC9ib2R5Pgo8c2NyaXB0PiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFwX2EwOGMyYzllYmI5OTQ0YzdiNGZkYjgxNTkzZmMzMGE2ID0gTC5tYXAoCiAgICAgICAgICAgICAgICAibWFwX2EwOGMyYzllYmI5OTQ0YzdiNGZkYjgxNTkzZmMzMGE2IiwKICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICBjZW50ZXI6IFstMjQuNzc2MTA4NiwgMTM0Ljc1NV0sCiAgICAgICAgICAgICAgICAgICAgY3JzOiBMLkNSUy5FUFNHMzg1NywKICAgICAgICAgICAgICAgICAgICB6b29tOiA0LAogICAgICAgICAgICAgICAgICAgIHpvb21Db250cm9sOiB0cnVlLAogICAgICAgICAgICAgICAgICAgIHByZWZlckNhbnZhczogZmFsc2UsCiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgICk7CgogICAgICAgICAgICAKCiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIHRpbGVfbGF5ZXJfZDk5MjdjY2U5MzYxNGFkNDkzOTFiZTIzNTliYjViMTggPSBMLnRpbGVMYXllcigKICAgICAgICAgICAgICAgICJodHRwczovL3tzfS50aWxlLm9wZW5zdHJlZXRtYXAub3JnL3t6fS97eH0ve3l9LnBuZyIsCiAgICAgICAgICAgICAgICB7ImF0dHJpYnV0aW9uIjogIkRhdGEgYnkgXHUwMDI2Y29weTsgXHUwMDNjYSBocmVmPVwiaHR0cDovL29wZW5zdHJlZXRtYXAub3JnXCJcdTAwM2VPcGVuU3RyZWV0TWFwXHUwMDNjL2FcdTAwM2UsIHVuZGVyIFx1MDAzY2EgaHJlZj1cImh0dHA6Ly93d3cub3BlbnN0cmVldG1hcC5vcmcvY29weXJpZ2h0XCJcdTAwM2VPRGJMXHUwMDNjL2FcdTAwM2UuIiwgImRldGVjdFJldGluYSI6IGZhbHNlLCAibWF4TmF0aXZlWm9vbSI6IDE4LCAibWF4Wm9vbSI6IDE4LCAibWluWm9vbSI6IDAsICJub1dyYXAiOiBmYWxzZSwgIm9wYWNpdHkiOiAxLCAic3ViZG9tYWlucyI6ICJhYmMiLCAidG1zIjogZmFsc2V9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2EwOGMyYzllYmI5OTQ0YzdiNGZkYjgxNTkzZmMzMGE2KTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl84NTYzZTMyZjdhMzc0NmUwYTJhODMwZjFkMDZiZmYwYSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNS4yODIwMDEsIDE0OS4xMjg5OThdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF9hMDhjMmM5ZWJiOTk0NGM3YjRmZGI4MTU5M2ZjMzBhNik7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZTZiNjIwZjhkNmJhNGM5MTgyYTI3MjQyYTBkN2Y2NjEgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMjYuNjUsIDE1My4wNjY2NjZdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF9hMDhjMmM5ZWJiOTk0NGM3YjRmZGI4MTU5M2ZjMzBhNik7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMWZkMmI2NGIxZjkzNDdmMDkyMDg3ZDFhZDQ2ZTg4ZWIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMjguMDE2NjY1OTk5OTk5OTk3LCAxNTMuMzk5OTk0XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTA4YzJjOWViYjk5NDRjN2I0ZmRiODE1OTNmYzMwYTYpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2E5YzQ2MzZjNzI0ODQ5NDY4NmQ0YWM0MDc0MzZlZDUwID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM3Ljg0MDkzNDk5OTk5OTk5NSwgMTQ0Ljk0NjQ1N10sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2EwOGMyYzllYmI5OTQ0YzdiNGZkYjgxNTkzZmMzMGE2KTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl84ZWMwYjI2NTlmY2I0ZTgwOGNlZDJhMTQyYTBlZTg4YyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNC45MjEyMywgMTM4LjU5OTUwM10sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2EwOGMyYzllYmI5OTQ0YzdiNGZkYjgxNTkzZmMzMGE2KTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8zNDY0OTM4NWQ0MDg0ZWQ2OTE1MzdmMmVlNGI5NjM2YSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy00MS40Mjk4MjUsIDE0Ny4xNTcxMzVdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF9hMDhjMmM5ZWJiOTk0NGM3YjRmZGI4MTU5M2ZjMzBhNik7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNTFlYjhmZGExMDQ3NDNmNDk4YTBhN2QyNjZmNWRhNWYgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzQuOTA2MTAxLCAxMzguNTkzOTAzXSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTA4YzJjOWViYjk5NDRjN2I0ZmRiODE1OTNmYzMwYTYpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2JiZmUyZDk4OTg5NjQyNjE4ZTQ4NDVlOWFhYTIzMmJiID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTE5LjI1ODk2NSwgMTQ2LjgxNjk1Nl0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2EwOGMyYzllYmI5OTQ0YzdiNGZkYjgxNTkzZmMzMGE2KTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl85YTY2NTg5ZWMxMTM0ZGY1ODYwM2MxYWM0NzI2MTlhYyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0xNi45MjU0OTEsIDE0NS43NTQxMl0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2EwOGMyYzllYmI5OTQ0YzdiNGZkYjgxNTkzZmMzMGE2KTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8zYTA4ZWU3ZjQxMDI0YzQ2YjJhOTkxOTRmZDcxYWZiOSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zMS45NTM1MTIsIDExNS44NTcwNDc5OTk5OTk5OV0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2EwOGMyYzllYmI5OTQ0YzdiNGZkYjgxNTkzZmMzMGE2KTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9lZTVlOGI0MTVlNzM0ZGU3OWVmYWU4MmVlMGMwZDlmZSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNC4yMDY4NDEsIDE0Mi4xMzY0ODk5OTk5OTk5OF0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2EwOGMyYzllYmI5OTQ0YzdiNGZkYjgxNTkzZmMzMGE2KTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8zYzIyNDM0NmIyN2U0MjM2OTY3Y2E4ZjZhZGI5YWQ0OSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy42NDk5NjcsIDE0NC44ODA2XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTA4YzJjOWViYjk5NDRjN2I0ZmRiODE1OTNmYzMwYTYpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzRiNzE2YmI1NjU0NzQwYzZiZGIzZTY3MTk0ODk0NGI4ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTMwLjI5NjI3NjAwMDAwMDAwMiwgMTUzLjExNDEzNl0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2EwOGMyYzllYmI5OTQ0YzdiNGZkYjgxNTkzZmMzMGE2KTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8yY2Q4OGNhYmZiNTQ0ZDdhYWMxYjZmMDc4N2MxZGU4ZCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zMy4yODM1NzcsIDE0OS4xMDEyNzNdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF9hMDhjMmM5ZWJiOTk0NGM3YjRmZGI4MTU5M2ZjMzBhNik7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZWU2ZTQ0NGU1YTFjNDQ3NmFkZWI4MjRkMjMzYzU0ODcgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzYuNzU3Nzg1OTk5OTk5OTk2LCAxNDQuMjc4NzAyXSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTA4YzJjOWViYjk5NDRjN2I0ZmRiODE1OTNmYzMwYTYpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzJiNzZhNmYxYjg4NDQ2ZWU4Zjg1OWQxNmZjMTVjMTQ3ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM2LjA4MDc4LCAxNDYuOTE2NDczXSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTA4YzJjOWViYjk5NDRjN2I0ZmRiODE1OTNmYzMwYTYpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzFiYTBmNWMxMzVlNTQ2ZGNhOTFiNjE2MmIzZWZiNDg1ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM0LjQyNTA3MiwgMTUwLjg5MzE0Mjk5OTk5OTk4XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTA4YzJjOWViYjk5NDRjN2I0ZmRiODE1OTNmYzMwYTYpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzc0YTdjYjI1OGY3NjRjOWI4Nzk4ZWNlODIxM2Q0OWI4ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTEyLjQ2MjgyNywgMTMwLjg0MTc4Ml0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2EwOGMyYzllYmI5OTQ0YzdiNGZkYjgxNTkzZmMzMGE2KTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8wYjdjNzRkYWQ3ZTQ0NWE2YTNkMjYxMjczOTA5NTFlOCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zMy42ODMyMTIsIDE1MS4yMjQzOTU5OTk5OTk5OF0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2EwOGMyYzllYmI5OTQ0YzdiNGZkYjgxNTkzZmMzMGE2KTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl83MWRmY2I3ZGI3NDU0MGIzYjQxZmMzODg1MmI1YTljOSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zMy45MTcyOSwgMTUxLjAzNTg4OV0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2EwOGMyYzllYmI5OTQ0YzdiNGZkYjgxNTkzZmMzMGE2KTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl82NWY0YWFmNWU2Zjg0N2YxOWIyOWZmODA2ODQyNTJjMSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zMy44MDc2OSwgMTUwLjk4NzI3Mzk5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTA4YzJjOWViYjk5NDRjN2I0ZmRiODE1OTNmYzMwYTYpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2NkMGU4ZDRmOWYwYTRkNTBhMTNiYjFmMjE3NzRjODg0ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTI3LjQ3MDEyNSwgMTUzLjAyMTA3Ml0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2EwOGMyYzllYmI5OTQ0YzdiNGZkYjgxNTkzZmMzMGE2KTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9mNThiZjdiZTU0ZmU0YWVhYTJjMmVhYjZiNDJiMzMxZiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0yMy44NDMxMzgsIDE1MS4yNjgzNTZdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF9hMDhjMmM5ZWJiOTk0NGM3YjRmZGI4MTU5M2ZjMzBhNik7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMDliZGVhNmVjMDY0NGIzM2IzZWZmMjI5NjViOTI0ODIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzUuMzQzNzg0LCAxNDkuMDgyOTc3XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfYTA4YzJjOWViYjk5NDRjN2I0ZmRiODE1OTNmYzMwYTYpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzk0ZDI0ZWVmZmM5YzQyMzE5NmJkM2I3NTkwNDg0ZDU1ID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTMzLjQyNTAxOCwgMTUxLjM0MjIyNF0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2EwOGMyYzllYmI5OTQ0YzdiNGZkYjgxNTkzZmMzMGE2KTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl80YjM2NDRhZDdjOTc0Y2NlYjJlMzEwYjEyY2MxYjQzYiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNy44MjQ0Mjg5OTk5OTk5OTUsIDE0MC43ODM3ODNdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF9hMDhjMmM5ZWJiOTk0NGM3YjRmZGI4MTU5M2ZjMzBhNik7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMTNhMzcxZDgzNzljNGU3MGI1ZDRjYmEzNmQ3ODI4OGQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzMuODY1MTQyOTk5OTk5OTk2LCAxNTEuMjA5OV0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2EwOGMyYzllYmI5OTQ0YzdiNGZkYjgxNTkzZmMzMGE2KTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8wNTA1NjQyZTAyZDQ0YzYyOWJkNjQwMzRmOWQ3N2IyNSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0yNy41Mjk5NTMwMDAwMDAwMDMsIDE1Mi40MDcxODFdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF9hMDhjMmM5ZWJiOTk0NGM3YjRmZGI4MTU5M2ZjMzBhNik7CiAgICAgICAgCjwvc2NyaXB0Pg==" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>



#A closer look at NSW


```python
nsw_data = city_data[city_data['State'].str.contains('NSW')].reset_index(drop = True)
print("The shape of the dataframe:", nsw_data.shape)
nsw_data[0:10]
```

    The shape of the dataframe: (9, 4)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>State</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Coffs Harbour</td>
      <td>-30.296276</td>
      <td>153.114136</td>
      <td>NSW</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Orange</td>
      <td>-33.283577</td>
      <td>149.101273</td>
      <td>NSW</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Albury</td>
      <td>-36.080780</td>
      <td>146.916473</td>
      <td>NSW</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Wollongong</td>
      <td>-34.425072</td>
      <td>150.893143</td>
      <td>NSW</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Terrey Hills</td>
      <td>-33.683212</td>
      <td>151.224396</td>
      <td>NSW</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Bankstown</td>
      <td>-33.917290</td>
      <td>151.035889</td>
      <td>NSW</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Westmead</td>
      <td>-33.807690</td>
      <td>150.987274</td>
      <td>NSW</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Gosford</td>
      <td>-33.425018</td>
      <td>151.342224</td>
      <td>NSW</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Sydney</td>
      <td>-33.865143</td>
      <td>151.209900</td>
      <td>NSW</td>
    </tr>
  </tbody>
</table>
</div>




```python
address = 'Sydney'
geolocator = Nominatim(user_agent = 'my_application')
location = geolocator.geocode(address)
latitude = location.latitude
longitude = location.longitude
print("The geographical coordinates of Sydney are {},{}.".format(latitude, longitude))
```

    The geographical coordinates of Sydney are -33.8548157,151.2164539.
    


```python
nsw_map = folium.Map(location = [latitude, longitude], zoom_start = 6)
for lat, lon, city, state in zip(nsw_data['Latitude'], nsw_data['Longitude'], nsw_data['City'], nsw_data['State']):
    label = '{}'.format(city, state)
    label = folium.Popup(label, parse_html = True)
    folium.CircleMarker(
       [lat, lon],
       radius = 5,
       color = 'blue',
       fill = True,
       fill_color = '#3186cc',
       fill_opacity = 0.7,
       parse_html = False).add_to(nsw_map)
nsw_map       
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2Nkbi5naXRoYWNrLmNvbS9weXRob24tdmlzdWFsaXphdGlvbi9mb2xpdW0vbWFzdGVyL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5jc3MiLz4KICAgIDxzdHlsZT5odG1sLCBib2R5IHt3aWR0aDogMTAwJTtoZWlnaHQ6IDEwMCU7bWFyZ2luOiAwO3BhZGRpbmc6IDA7fTwvc3R5bGU+CiAgICA8c3R5bGU+I21hcCB7cG9zaXRpb246YWJzb2x1dGU7dG9wOjA7Ym90dG9tOjA7cmlnaHQ6MDtsZWZ0OjA7fTwvc3R5bGU+CiAgICAKICAgICAgICAgICAgPG1ldGEgbmFtZT0idmlld3BvcnQiIGNvbnRlbnQ9IndpZHRoPWRldmljZS13aWR0aCwKICAgICAgICAgICAgICAgIGluaXRpYWwtc2NhbGU9MS4wLCBtYXhpbXVtLXNjYWxlPTEuMCwgdXNlci1zY2FsYWJsZT1ubyIgLz4KICAgICAgICAgICAgPHN0eWxlPgogICAgICAgICAgICAgICAgI21hcF8xM2IzNzk4ZTcwODk0YmQ2OTYzOTkyZDdiZTVmNDgxZiB7CiAgICAgICAgICAgICAgICAgICAgcG9zaXRpb246IHJlbGF0aXZlOwogICAgICAgICAgICAgICAgICAgIHdpZHRoOiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgbGVmdDogMC4wJTsKICAgICAgICAgICAgICAgICAgICB0b3A6IDAuMCU7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgIDwvc3R5bGU+CiAgICAgICAgCjwvaGVhZD4KPGJvZHk+ICAgIAogICAgCiAgICAgICAgICAgIDxkaXYgY2xhc3M9ImZvbGl1bS1tYXAiIGlkPSJtYXBfMTNiMzc5OGU3MDg5NGJkNjk2Mzk5MmQ3YmU1ZjQ4MWYiID48L2Rpdj4KICAgICAgICAKPC9ib2R5Pgo8c2NyaXB0PiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFwXzEzYjM3OThlNzA4OTRiZDY5NjM5OTJkN2JlNWY0ODFmID0gTC5tYXAoCiAgICAgICAgICAgICAgICAibWFwXzEzYjM3OThlNzA4OTRiZDY5NjM5OTJkN2JlNWY0ODFmIiwKICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICBjZW50ZXI6IFstMzMuODU0ODE1NywgMTUxLjIxNjQ1MzldLAogICAgICAgICAgICAgICAgICAgIGNyczogTC5DUlMuRVBTRzM4NTcsCiAgICAgICAgICAgICAgICAgICAgem9vbTogNiwKICAgICAgICAgICAgICAgICAgICB6b29tQ29udHJvbDogdHJ1ZSwKICAgICAgICAgICAgICAgICAgICBwcmVmZXJDYW52YXM6IGZhbHNlLAogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApOwoKICAgICAgICAgICAgCgogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciB0aWxlX2xheWVyX2EzYmI4OTdjN2QwMjQ5NWE5ODAwMDJlMDU4NjRkMTllID0gTC50aWxlTGF5ZXIoCiAgICAgICAgICAgICAgICAiaHR0cHM6Ly97c30udGlsZS5vcGVuc3RyZWV0bWFwLm9yZy97en0ve3h9L3t5fS5wbmciLAogICAgICAgICAgICAgICAgeyJhdHRyaWJ1dGlvbiI6ICJEYXRhIGJ5IFx1MDAyNmNvcHk7IFx1MDAzY2EgaHJlZj1cImh0dHA6Ly9vcGVuc3RyZWV0bWFwLm9yZ1wiXHUwMDNlT3BlblN0cmVldE1hcFx1MDAzYy9hXHUwMDNlLCB1bmRlciBcdTAwM2NhIGhyZWY9XCJodHRwOi8vd3d3Lm9wZW5zdHJlZXRtYXAub3JnL2NvcHlyaWdodFwiXHUwMDNlT0RiTFx1MDAzYy9hXHUwMDNlLiIsICJkZXRlY3RSZXRpbmEiOiBmYWxzZSwgIm1heE5hdGl2ZVpvb20iOiAxOCwgIm1heFpvb20iOiAxOCwgIm1pblpvb20iOiAwLCAibm9XcmFwIjogZmFsc2UsICJvcGFjaXR5IjogMSwgInN1YmRvbWFpbnMiOiAiYWJjIiwgInRtcyI6IGZhbHNlfQogICAgICAgICAgICApLmFkZFRvKG1hcF8xM2IzNzk4ZTcwODk0YmQ2OTYzOTkyZDdiZTVmNDgxZik7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfZTA4N2Q2NTNjNjdkNDI5Yjk3YzgyNGRkYWNjMzQ5MWUgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzAuMjk2Mjc2MDAwMDAwMDAyLCAxNTMuMTE0MTM2XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfMTNiMzc5OGU3MDg5NGJkNjk2Mzk5MmQ3YmU1ZjQ4MWYpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2QxYmFkOGY3MWUxNjRhNDQ5OTYwOTVjYWRmMTM4NDVjID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTMzLjI4MzU3NywgMTQ5LjEwMTI3M10sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzEzYjM3OThlNzA4OTRiZDY5NjM5OTJkN2JlNWY0ODFmKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9hOTFlOWY5YzZiNTY0YmY5ODliYWY3NGExMDE5ODA2ZiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNi4wODA3OCwgMTQ2LjkxNjQ3M10sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzEzYjM3OThlNzA4OTRiZDY5NjM5OTJkN2JlNWY0ODFmKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl84MWM5ZTg4M2Q3NWM0MTBkYTE3OTc2MjU0OTQwNzQzNSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zNC40MjUwNzIsIDE1MC44OTMxNDI5OTk5OTk5OF0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzEzYjM3OThlNzA4OTRiZDY5NjM5OTJkN2JlNWY0ODFmKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl85MGI3NDQyZjNlZmE0MGYyYTg2YjFiZjlmZGQyMjc1ZiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zMy42ODMyMTIsIDE1MS4yMjQzOTU5OTk5OTk5OF0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzEzYjM3OThlNzA4OTRiZDY5NjM5OTJkN2JlNWY0ODFmKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9kMTE0ZTY0ODc5NWY0ZWQ1OTJkOGYwOTMzMjBkYjA5YSA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zMy45MTcyOSwgMTUxLjAzNTg4OV0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzEzYjM3OThlNzA4OTRiZDY5NjM5OTJkN2JlNWY0ODFmKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl9mZDk4MDFjMGQ5MGE0NmU0YmY0NTRmODFiOTg5ZDk3YyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zMy44MDc2OSwgMTUwLjk4NzI3Mzk5OTk5OTk5XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfMTNiMzc5OGU3MDg5NGJkNjk2Mzk5MmQ3YmU1ZjQ4MWYpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzhiZDNiMmNmNTZlMzRjZjI4NjRlOWZjZDJiNTVmN2MzID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTMzLjQyNTAxOCwgMTUxLjM0MjIyNF0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzEzYjM3OThlNzA4OTRiZDY5NjM5OTJkN2JlNWY0ODFmKTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl83Yjg0NDhlOWVlNjk0NDBmYTRiNzA2MTk5ZDI4NDc5NyA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0zMy44NjUxNDI5OTk5OTk5OTYsIDE1MS4yMDk5XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfMTNiMzc5OGU3MDg5NGJkNjk2Mzk5MmQ3YmU1ZjQ4MWYpOwogICAgICAgIAo8L3NjcmlwdD4=" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




```python
CLIENT_ID = 'L351DHR2FSTSDETU0K5X3IZMC5XCTH5GJMOTYKAE0BEVPVO3'
CLIENT_SECRET = 'U0NV33KA4DBR4L5ZDYKM2XXXOO4RFH5DI1XNRFUGJDIF2MVR'
VERSION = '20180605'
print('Your Credentials:')
print('CLIENT_ID:' + CLIENT_ID)
print('CLIENT_SECRET:' + CLIENT_SECRET)
```

    Your Credentials:
    CLIENT_ID:L351DHR2FSTSDETU0K5X3IZMC5XCTH5GJMOTYKAE0BEVPVO3
    CLIENT_SECRET:U0NV33KA4DBR4L5ZDYKM2XXXOO4RFH5DI1XNRFUGJDIF2MVR
    


```python
nsw_data.loc[8, "City"]
```




    'Sydney'




```python
city_latitude = nsw_data.loc[8, "Latitude"]
city_longitude = nsw_data.loc[8, "Longitude"]
city_name = nsw_data.loc[8, "City"]
print("The Latitude and Longitude of {} are {}, {}.".format(city_name, city_latitude, city_longitude)) 
```

    The Latitude and Longitude of Sydney are -33.865142999999996, 151.2099.
    


```python
radius = 5000
limit = 100
query = "hospitals"
url = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&ll={},{}&v={}&radius={}&limit={}&query={}'.format(CLIENT_ID, CLIENT_SECRET, city_latitude, city_longitude, VERSION, radius, limit, query)
url
```




    'https://api.foursquare.com/v2/venues/explore?&client_id=L351DHR2FSTSDETU0K5X3IZMC5XCTH5GJMOTYKAE0BEVPVO3&client_secret=U0NV33KA4DBR4L5ZDYKM2XXXOO4RFH5DI1XNRFUGJDIF2MVR&ll=-33.865142999999996,151.2099&v=20180605&radius=5000&limit=100&query=hospitals'




```python
results = requests.get(url).json()
results
```




    {'meta': {'code': 200, 'requestId': '5d73a790b77c77002c603d12'},
     'response': {'headerLocation': 'Sydney',
      'headerFullLocation': 'Sydney',
      'headerLocationGranularity': 'city',
      'query': 'hospitals',
      'totalResults': 43,
      'suggestedBounds': {'ne': {'lat': -33.82014295499995,
        'lng': 151.2639927880043},
       'sw': {'lat': -33.91014304500004, 'lng': 151.15580721199572}},
      'groups': [{'type': 'Recommended Places',
        'name': 'recommended',
        'items': [{'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b058770f964a520279322e3',
           'name': 'Sydney Hospital',
           'location': {'address': '8 Macquarie St',
            'lat': -33.868355137475156,
            'lng': 151.21274254448159,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.868355137475156,
              'lng': 151.21274254448159}],
            'distance': 443,
            'postalCode': '2000',
            'cc': 'AU',
            'city': 'Sydney',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['8 Macquarie St',
             'Sydney NSW 2000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b058770f964a520279322e3-0'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4d87c6c0d85f3704e813c4db',
           'name': 'Sydney Hospital Hand Clinic',
           'location': {'address': 'Macquarie St.',
            'lat': -33.867909681694776,
            'lng': 151.2123074315786,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.867909681694776,
              'lng': 151.2123074315786}],
            'distance': 379,
            'postalCode': '2000',
            'cc': 'AU',
            'city': 'Sydney',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Macquarie St.', 'Sydney NSW 2000', 'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4d87c6c0d85f3704e813c4db-1'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '505c04a7e4b0cc983370707a',
           'name': 'Sydney Sexual Health Centre',
           'location': {'address': 'Level 3, Nightingale Wing, Sydney Hospital',
            'crossStreet': 'Macquarie St',
            'lat': -33.86786778078052,
            'lng': 151.21270895004272,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.86786778078052,
              'lng': 151.21270895004272}],
            'distance': 399,
            'postalCode': '2000',
            'cc': 'AU',
            'city': 'Sydney',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Level 3, Nightingale Wing, Sydney Hospital (Macquarie St)',
             'Sydney NSW 2000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-505c04a7e4b0cc983370707a-2'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4bc6b673db8fa593a2c89c37',
           'name': 'Sydney Eye Hospital',
           'location': {'address': '8 Macquarie St.',
            'lat': -33.86821793095142,
            'lng': 151.21266032394652,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.86821793095142,
              'lng': 151.21266032394652}],
            'distance': 426,
            'postalCode': '2000',
            'cc': 'AU',
            'city': 'Sydney',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['8 Macquarie St.',
             'Sydney NSW 2000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4bc6b673db8fa593a2c89c37-3'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '5bea449d018cbb002caa59de',
           'name': 'Boneham Optometrist Eyecare Plus',
           'location': {'address': 'Suite 17, Level 4, 428 George St',
            'lat': -33.86996,
            'lng': 151.20738,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.86996,
              'lng': 151.20738}],
            'distance': 584,
            'postalCode': '2000',
            'cc': 'AU',
            'city': 'Sydney',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Suite 17, Level 4, 428 George St',
             'Sydney NSW 2000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '515148020'}},
          'referralId': 'e-0-5bea449d018cbb002caa59de-4'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4e5f01c7d1640a5975d265e5',
           'name': 'Genea',
           'location': {'address': '321 Kent St',
            'lat': -33.86756,
            'lng': 151.20396,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.86756,
              'lng': 151.20396}],
            'distance': 611,
            'postalCode': '2000',
            'cc': 'AU',
            'city': 'Sydney',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['321 Kent St', 'Sydney NSW 2000', 'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '537042380'}},
          'referralId': 'e-0-4e5f01c7d1640a5975d265e5-5'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '5444894b498e71ae5f79d946',
           'name': 'East Sydney Private Hospital',
           'location': {'address': '75 Crown Street',
            'lat': -33.872718933780895,
            'lng': 151.21678694623998,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.872718933780895,
              'lng': 151.21678694623998}],
            'distance': 1056,
            'postalCode': '2000',
            'cc': 'AU',
            'city': 'Sydney',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['75 Crown Street',
             'Sydney NSW 2000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-5444894b498e71ae5f79d946-6'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '5491f16a498ed165fa64349d',
           'name': 'Sydney Cardiology',
           'location': {'address': '37 Bligh Street',
            'lat': -33.8659187,
            'lng': 151.2097205,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.8659187,
              'lng': 151.2097205}],
            'distance': 87,
            'postalCode': '2000',
            'cc': 'AU',
            'city': 'Sydney',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['37 Bligh Street',
             'Sydney NSW 2000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '511234545'}},
          'referralId': 'e-0-5491f16a498ed165fa64349d-7'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b0dc43bf964a520ac4f23e3',
           'name': "St Vincent's Hospital",
           'location': {'address': '390 Victoria St.',
            'crossStreet': 'Oxford St.',
            'lat': -33.88013803962198,
            'lng': 151.22061209403043,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.88013803962198,
              'lng': 151.22061209403043}],
            'distance': 1940,
            'postalCode': '2010',
            'cc': 'AU',
            'city': 'Darlinghurst',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['390 Victoria St. (Oxford St.)',
             'Darlinghurst NSW 2010',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b0dc43bf964a520ac4f23e3-8'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b9abfc7f964a5204ad235e3',
           'name': "St Luke's Hospital",
           'location': {'address': '18 Roslyn St.',
            'crossStreet': 'Ward Ave.',
            'lat': -33.87438921040412,
            'lng': 151.22595776424458,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.87438921040412,
              'lng': 151.22595776424458}],
            'distance': 1806,
            'postalCode': '2011',
            'cc': 'AU',
            'city': 'Potts Point',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['18 Roslyn St. (Ward Ave.)',
             'Potts Point NSW 2011',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b9abfc7f964a5204ad235e3-9'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4cfaf602d7206ea8df483c69',
           'name': 'Sacred Heart Palliative Care',
           'location': {'lat': -33.880358,
            'lng': 151.219826,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.880358,
              'lng': 151.219826}],
            'distance': 1926,
            'cc': 'AU',
            'country': 'Australia',
            'formattedAddress': ['Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4cfaf602d7206ea8df483c69-10'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4cf584c30a71224b8ed61893',
           'name': "O'Brien Centre, St Vincent's Hospital",
           'location': {'address': 'Burton Street',
            'crossStreet': 'Victoria Street',
            'lat': -33.87952645648002,
            'lng': 151.22149427272592,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.87952645648002,
              'lng': 151.22149427272592}],
            'distance': 1926,
            'cc': 'AU',
            'city': 'Darlinghurst',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Burton Street (Victoria Street)',
             'Darlinghurst NSW',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4cf584c30a71224b8ed61893-11'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b4a3676f964a520ee7e26e3',
           'name': "St Vincent's Private Hospital",
           'location': {'address': '406 Victoria St.',
            'crossStreet': 'Oxford St.',
            'lat': -33.88061572195354,
            'lng': 151.2197043563408,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.88061572195354,
              'lng': 151.2197043563408}],
            'distance': 1946,
            'postalCode': '2010',
            'cc': 'AU',
            'city': 'Darlinghurst',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['406 Victoria St. (Oxford St.)',
             'Darlinghurst NSW 2010',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b4a3676f964a520ee7e26e3-12'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '56568e0338fae7c089a1353b',
           'name': 'VeyeP',
           'location': {'address': 'Shop 21 242 Darling Street',
            'crossStreet': 'Eaton Street',
            'lat': -33.85838869548908,
            'lng': 151.1838698387146,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.85838869548908,
              'lng': 151.1838698387146}],
            'distance': 2520,
            'postalCode': '2041',
            'cc': 'AU',
            'city': 'Balmain',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Shop 21 242 Darling Street (Eaton Street)',
             'Balmain NSW 2041',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '511916178'}},
          'referralId': 'e-0-56568e0338fae7c089a1353b-13'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b24a649f964a520376924e3',
           'name': 'Royal Prince Alfred Hospital (RPA)',
           'location': {'address': 'Missenden Rd.',
            'crossStreet': 'John Hopkins Dr.',
            'lat': -33.88950695462588,
            'lng': 151.18256484846572,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.88950695462588,
              'lng': 151.18256484846572}],
            'distance': 3706,
            'postalCode': '2050',
            'cc': 'AU',
            'city': 'Camperdown',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Missenden Rd. (John Hopkins Dr.)',
             'Camperdown NSW 2050',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b24a649f964a520376924e3-14'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4baa7e8ff964a520e16e3ae3',
           'name': "St Vincent's Clinic",
           'location': {'address': '438 Victoria St.',
            'crossStreet': 'Oxford St.',
            'lat': -33.8813687550805,
            'lng': 151.21952415421745,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.8813687550805,
              'lng': 151.21952415421745}],
            'distance': 2013,
            'postalCode': '2010',
            'cc': 'AU',
            'city': 'Darlinghurst',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['438 Victoria St. (Oxford St.)',
             'Darlinghurst NSW 2010',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d177941735',
             'name': "Doctor's Office",
             'pluralName': "Doctor's Offices",
             'shortName': "Doctor's Office",
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_doctorsoffice_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4baa7e8ff964a520e16e3ae3-15'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b6b4f65f964a520eeff2be3',
           'name': 'Balmain Hospital',
           'location': {'address': 'Booth St & Sorrie St',
            'lat': -33.859107545360594,
            'lng': 151.18188480706007,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.859107545360594,
              'lng': 151.18188480706007}],
            'distance': 2675,
            'postalCode': '2041',
            'cc': 'AU',
            'city': 'Balmain',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Booth St & Sorrie St',
             'Balmain NSW 2041',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b6b4f65f964a520eeff2be3-16'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4dc131c1ced7c686e1b5eaf5',
           'name': 'Albion Street Centre',
           'location': {'address': '150 Albion St',
            'crossStreet': 'Crown St',
            'lat': -33.88336767430956,
            'lng': 151.21362224423015,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.88336767430956,
              'lng': 151.21362224423015}],
            'distance': 2057,
            'postalCode': '2010',
            'cc': 'AU',
            'city': 'Surry Hills',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['150 Albion St (Crown St)',
             'Surry Hills NSW 2010',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4dc131c1ced7c686e1b5eaf5-17'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b689b73f964a52098822be3',
           'name': 'Sydney Dental Hospital',
           'location': {'address': '2-18 Chalmers St.',
            'lat': -33.88439989280431,
            'lng': 151.20763199187346,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.88439989280431,
              'lng': 151.20763199187346}],
            'distance': 2153,
            'postalCode': '2010',
            'cc': 'AU',
            'city': 'Surry Hills',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['2-18 Chalmers St.',
             'Surry Hills NSW 2010',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b689b73f964a52098822be3-18'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b132badf964a520129523e3',
           'name': 'Mater Hospital',
           'location': {'address': '25 Rocklands Rd.',
            'lat': -33.831721381200644,
            'lng': 151.20161482309732,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.831721381200644,
              'lng': 151.20161482309732}],
            'distance': 3798,
            'postalCode': '2060',
            'cc': 'AU',
            'city': 'North Sydney',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['25 Rocklands Rd.',
             'North Sydney NSW 2060',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b132badf964a520129523e3-19'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4bce580268f976b0a4fe6583',
           'name': 'Centenary Institute',
           'location': {'address': 'Bldg 93, RPA Hospital',
            'crossStreet': 'Missenden Rd.',
            'lat': -33.888225939643796,
            'lng': 151.18362901961342,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.888225939643796,
              'lng': 151.18362901961342}],
            'distance': 3535,
            'postalCode': '2050',
            'cc': 'AU',
            'city': 'Camperdown',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Bldg 93, RPA Hospital (Missenden Rd.)',
             'Camperdown NSW 2050',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4bce580268f976b0a4fe6583-20'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4d277d67342d6dcbf232eeca',
           'name': 'Wolper Jewish Hospital',
           'location': {'address': 'Trelawney st',
            'lat': -33.884213647218274,
            'lng': 151.24058339379874,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.884213647218274,
              'lng': 151.24058339379874}],
            'distance': 3542,
            'cc': 'AU',
            'city': 'Woollahra',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Trelawney st', 'Woollahra NSW', 'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4d277d67342d6dcbf232eeca-21'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4ec432ad0e617a27f98ebb25',
           'name': 'RPA Birth Centre',
           'location': {'address': 'Royal Prince Alfred Hospital',
            'crossStreet': 'Missenden Rd.',
            'lat': -33.888603,
            'lng': 151.183423,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.888603,
              'lng': 151.183423}],
            'distance': 3578,
            'postalCode': '2050',
            'cc': 'AU',
            'city': 'Camperdown',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Royal Prince Alfred Hospital (Missenden Rd.)',
             'Camperdown NSW 2050',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4ec432ad0e617a27f98ebb25-22'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '5ad4806ba42362528c79e0a0',
           'name': 'Balanced Bods Health',
           'location': {'address': '4/281 Pacific Hwy',
            'lat': -33.8330648,
            'lng': 151.2047892,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.8330648,
              'lng': 151.2047892}],
            'distance': 3602,
            'postalCode': '2060',
            'cc': 'AU',
            'city': 'North Sydney',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['4/281 Pacific Hwy',
             'North Sydney NSW 2060',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '490678045'}},
          'referralId': 'e-0-5ad4806ba42362528c79e0a0-23'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4cb5c4201b0af04dbd8bcb25',
           'name': 'RPA Intensive Care Unit (ICU)',
           'location': {'address': 'Royal Prince Alfred Hospital',
            'crossStreet': 'Missenden Rd.',
            'lat': -33.888789738174275,
            'lng': 151.18324204745676,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.888789738174275,
              'lng': 151.18324204745676}],
            'distance': 3605,
            'postalCode': '2050',
            'cc': 'AU',
            'city': 'Camperdown',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Royal Prince Alfred Hospital (Missenden Rd.)',
             'Camperdown NSW 2050',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4cb5c4201b0af04dbd8bcb25-24'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b184635f964a5200ed023e3',
           'name': 'RPA Women & Babies',
           'location': {'address': 'Royal Prince Alfred Hospital',
            'crossStreet': 'Missenden Rd.',
            'lat': -33.88967038338814,
            'lng': 151.1835301564085,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.88967038338814,
              'lng': 151.1835301564085}],
            'distance': 3659,
            'postalCode': '2050',
            'cc': 'AU',
            'city': 'Camperdown',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Royal Prince Alfred Hospital (Missenden Rd.)',
             'Camperdown NSW 2050',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b184635f964a5200ed023e3-25'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4cd32da06be6a14326f49202',
           'name': 'Gloucester House',
           'location': {'address': 'Royal Prince Alfred Hospital',
            'lat': -33.889772040501576,
            'lng': 151.18354498328364,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.889772040501576,
              'lng': 151.18354498328364}],
            'distance': 3667,
            'postalCode': '2050',
            'cc': 'AU',
            'city': 'Camperdown',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Royal Prince Alfred Hospital',
             'Camperdown NSW 2050',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4cd32da06be6a14326f49202-26'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4c4673d919fde21e34950576',
           'name': 'RPA QEII Building 10',
           'location': {'address': 'Royal Prince Alfred Hospital',
            'crossStreet': '57 - 59 Missenden Rd.',
            'lat': -33.88816140780927,
            'lng': 151.18100152987162,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.88816140780927,
              'lng': 151.18100152987162}],
            'distance': 3701,
            'postalCode': '2050',
            'cc': 'AU',
            'city': 'Camperdown',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Royal Prince Alfred Hospital (57 - 59 Missenden Rd.)',
             'Camperdown NSW 2050',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4c4673d919fde21e34950576-27'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4e790ae2c65b010445972725',
           'name': 'RPA Fracture Clinic',
           'location': {'address': 'Royal Prince Alfred Hospital',
            'crossStreet': 'QEll Building, 59 Missenden Rd.',
            'lat': -33.88856,
            'lng': 151.18112,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.88856,
              'lng': 151.18112}],
            'distance': 3724,
            'postalCode': '2050',
            'cc': 'AU',
            'city': 'Camperdown',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Royal Prince Alfred Hospital (QEll Building, 59 Missenden Rd.)',
             'Camperdown NSW 2050',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4e790ae2c65b010445972725-28'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4e558f7eb993b6af27b518ab',
           'name': 'RPA King George V Building',
           'location': {'address': 'Royal Prince Alfred Hospital',
            'crossStreet': 'Missenden Rd.',
            'lat': -33.88962019534626,
            'lng': 151.18214942531267,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.88962019534626,
              'lng': 151.18214942531267}],
            'distance': 3741,
            'postalCode': '2050',
            'cc': 'AU',
            'city': 'Camperdown',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Royal Prince Alfred Hospital (Missenden Rd.)',
             'Camperdown NSW 2050',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4e558f7eb993b6af27b518ab-29'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4d912542939e54812e9bc79e',
           'name': "The Chris O'Brien Lifehouse at RPA",
           'location': {'address': '119 Missenden Rd.',
            'crossStreet': 'btwn Brown & Salisbury',
            'lat': -33.89036839617582,
            'lng': 151.18218344986317,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.89036839617582,
              'lng': 151.18218344986317}],
            'distance': 3800,
            'postalCode': '2050',
            'cc': 'AU',
            'city': 'Camperdown',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['119 Missenden Rd. (btwn Brown & Salisbury)',
             'Camperdown NSW 2050',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4d912542939e54812e9bc79e-30'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4d80221cab666ea8178e00c1',
           'name': 'Mater Imaging',
           'location': {'address': '25 Rocklands Rd.',
            'lat': -33.83150950529342,
            'lng': 151.2017722489722,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.83150950529342,
              'lng': 151.2017722489722}],
            'distance': 3818,
            'postalCode': '2060',
            'cc': 'AU',
            'city': 'North Sydney',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['25 Rocklands Rd.',
             'North Sydney NSW 2060',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4d80221cab666ea8178e00c1-31'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4caed6111463a1431eb38fa9',
           'name': 'RPA Radiation Oncology',
           'location': {'lat': -33.89033255769612,
            'lng': 151.1818079824441,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.89033255769612,
              'lng': 151.1818079824441}],
            'distance': 3821,
            'cc': 'AU',
            'country': 'Australia',
            'formattedAddress': ['Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4caed6111463a1431eb38fa9-32'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '5d0ac9eaacb00b002c1cdd11',
           'name': 'The Dietologist',
           'location': {'address': 'Suite 318, Level 3/100 Carillon Ave',
            'lat': -33.8916757,
            'lng': 151.1834304,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.8916757,
              'lng': 151.1834304}],
            'distance': 3835,
            'postalCode': '2042',
            'cc': 'AU',
            'city': 'Newtown',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Suite 318, Level 3/100 Carillon Ave',
             'Newtown NSW 2042',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '549857764'}},
          'referralId': 'e-0-5d0ac9eaacb00b002c1cdd11-33'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4d8d227bfa9437046dbde1c5',
           'name': 'Alexandria Veterinary Hospital',
           'location': {'address': '138-142 Botany Rd',
            'lat': -33.89915,
            'lng': 151.1998,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.89915,
              'lng': 151.1998}],
            'distance': 3898,
            'postalCode': '2015',
            'cc': 'AU',
            'city': 'Alexandria',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['138-142 Botany Rd',
             'Alexandria NSW 2015',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4d8d227bfa9437046dbde1c5-34'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b18e803f964a52070d623e3',
           'name': 'Rozelle Hospital',
           'location': {'address': 'Church St & Glover St',
            'lat': -33.86681440452535,
            'lng': 151.16474816187645,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.86681440452535,
              'lng': 151.16474816187645}],
            'distance': 4177,
            'postalCode': '2040',
            'cc': 'AU',
            'city': 'Leichhardt',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Church St & Glover St',
             'Leichhardt NSW 2040',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b18e803f964a52070d623e3-35'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b8c3fd8f964a5208ac632e3',
           'name': 'RPA Emergency Department',
           'location': {'address': 'Royal Prince Alfred Hospital',
            'crossStreet': '50 Missenden Rd.',
            'lat': -33.88928842592762,
            'lng': 151.18209304750036,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.88928842592762,
              'lng': 151.18209304750036}],
            'distance': 3718,
            'postalCode': '2050',
            'cc': 'AU',
            'city': 'Camperdown',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Royal Prince Alfred Hospital (50 Missenden Rd.)',
             'Camperdown NSW 2050',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b8c3fd8f964a5208ac632e3-36'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b7227c5f964a5209a712de3',
           'name': 'Alfred Medical Imaging',
           'location': {'address': 'Suite 1, RPAH Medical Centre',
            'crossStreet': '100 Carillion Ave.',
            'lat': -33.89153664705511,
            'lng': 151.18300926001265,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.89153664705511,
              'lng': 151.18300926001265}],
            'distance': 3848,
            'postalCode': '2042',
            'cc': 'AU',
            'city': 'Newtown',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['Suite 1, RPAH Medical Centre (100 Carillion Ave.)',
             'Newtown NSW 2042',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b7227c5f964a5209a712de3-37'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4bf087343a15d13aadb03e9f',
           'name': 'Northside Clinic',
           'location': {'address': '2 Greenwich Rd.',
            'lat': -33.82524,
            'lng': 151.18838,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.82524,
              'lng': 151.18838}],
            'distance': 4867,
            'postalCode': '2065',
            'cc': 'AU',
            'city': 'Greenwich',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['2 Greenwich Rd.',
             'Greenwich NSW 2065',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4bf087343a15d13aadb03e9f-38'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4ccfea3b86cbb71304a7f936',
           'name': 'mosman private hospital',
           'location': {'address': '1 Ellamatta rd',
            'lat': -33.83481622745654,
            'lng': 151.24786243412254,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.83481622745654,
              'lng': 151.24786243412254}],
            'distance': 4869,
            'postalCode': '2088',
            'cc': 'AU',
            'city': 'mosman',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['1 Ellamatta rd',
             'Mosman NSW 2088',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4ccfea3b86cbb71304a7f936-39'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4c6ce5c465eda09367344dd0',
           'name': 'Greenwich Hospital',
           'location': {'lat': -33.82718459178881,
            'lng': 151.18346967802225,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.82718459178881,
              'lng': 151.18346967802225}],
            'distance': 4881,
            'cc': 'AU',
            'country': 'Australia',
            'formattedAddress': ['Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4c6ce5c465eda09367344dd0-40'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b25621bf964a520fb7024e3',
           'name': 'Paddington Cat Hospital',
           'location': {'address': '210 Oxford St.',
            'crossStreet': 'Young St.',
            'lat': -33.88483372936981,
            'lng': 151.22529368753402,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.88483372936981,
              'lng': 151.22529368753402}],
            'distance': 2613,
            'postalCode': '2021',
            'cc': 'AU',
            'city': 'Paddington',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['210 Oxford St. (Young St.)',
             'Paddington NSW 2021',
             'Australia']},
           'categories': [{'id': '4d954af4a243a5684765b473',
             'name': 'Veterinarian',
             'pluralName': 'Veterinarians',
             'shortName': 'Veterinarians',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_veterinarian_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b25621bf964a520fb7024e3-41'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b5b8391f964a520db0329e3',
           'name': 'Bondi Junction Veterinary Hospital',
           'location': {'address': '12 Ebley St.',
            'lat': -33.89445615826303,
            'lng': 151.24397980080965,
            'labeledLatLngs': [{'label': 'display',
              'lat': -33.89445615826303,
              'lng': 151.24397980080965}],
            'distance': 4535,
            'postalCode': '2022',
            'cc': 'AU',
            'city': 'Bondi Junction',
            'state': 'NSW',
            'country': 'Australia',
            'formattedAddress': ['12 Ebley St.',
             'Bondi Junction NSW 2022',
             'Australia']},
           'categories': [{'id': '4d954af4a243a5684765b473',
             'name': 'Veterinarian',
             'pluralName': 'Veterinarians',
             'shortName': 'Veterinarians',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_veterinarian_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b5b8391f964a520db0329e3-42'}]}]}}




```python
def get_category_type(row):
    try:
        categories_list = row['categories']
    except:
        categories_list = row['venue.categories']
    if len(categories_list) == 0:
        return None
    else:
        return categories_list[0]['name']
```


```python
venues = results['response']['groups'][0]['items']
nearby_venues = json_normalize(venues)
filtered_columns = ['venue.name', 'venue.categories', 'venue.location.latitude', 'venue.location.longitude']
nearby_venues = nearby_venues.loc[:, filtered_columns]
nearby_venues['venue.categories'] = nearby_venues.apply(get_category_type, axis = 1)
nearby_venues.columns = [col.split(".")[-1] for col in nearby_venues.columns]
print("The size of the dataframe is:", nearby_venues.shape)
nearby_venues
```

    The size of the dataframe is: (43, 4)
    

    C:\Users\c3273214\AppData\Local\Continuum\anaconda3\lib\site-packages\pandas\core\indexing.py:1494: FutureWarning:
    
    
    Passing list-likes to .loc or [] with any missing label will raise
    KeyError in the future, you can use .reindex() as an alternative.
    
    See the documentation here:
    https://pandas.pydata.org/pandas-docs/stable/indexing.html#deprecate-loc-reindex-listlike
    
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>categories</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Sydney Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Sydney Hospital Hand Clinic</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Sydney Sexual Health Centre</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Sydney Eye Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Boneham Optometrist Eyecare Plus</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Genea</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td>East Sydney Private Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Sydney Cardiology</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>St Vincent's Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9</th>
      <td>St Luke's Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Sacred Heart Palliative Care</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>11</th>
      <td>O'Brien Centre, St Vincent's Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>12</th>
      <td>St Vincent's Private Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>13</th>
      <td>VeyeP</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Royal Prince Alfred Hospital (RPA)</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>15</th>
      <td>St Vincent's Clinic</td>
      <td>Doctor's Office</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Balmain Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Albion Street Centre</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Sydney Dental Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Mater Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Centenary Institute</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Wolper Jewish Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>22</th>
      <td>RPA Birth Centre</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Balanced Bods Health</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>24</th>
      <td>RPA Intensive Care Unit (ICU)</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>25</th>
      <td>RPA Women &amp; Babies</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Gloucester House</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>27</th>
      <td>RPA QEII Building 10</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>28</th>
      <td>RPA Fracture Clinic</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>29</th>
      <td>RPA King George V Building</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>30</th>
      <td>The Chris O'Brien Lifehouse at RPA</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Mater Imaging</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>32</th>
      <td>RPA Radiation Oncology</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>33</th>
      <td>The Dietologist</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Alexandria Veterinary Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Rozelle Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>36</th>
      <td>RPA Emergency Department</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Alfred Medical Imaging</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>38</th>
      <td>Northside Clinic</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>39</th>
      <td>mosman private hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>40</th>
      <td>Greenwich Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>41</th>
      <td>Paddington Cat Hospital</td>
      <td>Veterinarian</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>42</th>
      <td>Bondi Junction Veterinary Hospital</td>
      <td>Veterinarian</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>


# A closer look at ACT and its hospitals

```python
act_data = city_data[city_data['State'].str.contains('ACT')].reset_index(drop = True)
print("The shape of the dataframe:", nsw_data.shape)
act_data
```

    The shape of the dataframe: (9, 4)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>State</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Canberra</td>
      <td>-35.282001</td>
      <td>149.128998</td>
      <td>ACT</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Phillip</td>
      <td>-35.343784</td>
      <td>149.082977</td>
      <td>ACT</td>
    </tr>
  </tbody>
</table>
</div>




```python
address = 'Canberra'
geolocator = Nominatim(user_agent = 'my_application')
location = geolocator.geocode(address)
latitude = location.latitude
longitude = location.longitude
print("The geographical coordinates of Canberra are {},{}.".format(latitude, longitude))
```

    The geographical coordinates of Canberra are -35.2975906,149.1012676.
    


```python
act_map = folium.Map(location = [latitude, longitude], zoom_start = 6)
for lat, lon, city, state in zip(act_data['Latitude'], act_data['Longitude'], act_data['City'], act_data['State']):
    label = '{}'.format(city, state)
    label = folium.Popup(label, parse_html = True)
    folium.CircleMarker(
       [lat, lon],
       radius = 5,
       color = 'blue',
       fill = True,
       fill_color = '#3186cc',
       fill_opacity = 0.7,
       parse_html = False).add_to(act_map)
act_map       
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2Nkbi5naXRoYWNrLmNvbS9weXRob24tdmlzdWFsaXphdGlvbi9mb2xpdW0vbWFzdGVyL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5jc3MiLz4KICAgIDxzdHlsZT5odG1sLCBib2R5IHt3aWR0aDogMTAwJTtoZWlnaHQ6IDEwMCU7bWFyZ2luOiAwO3BhZGRpbmc6IDA7fTwvc3R5bGU+CiAgICA8c3R5bGU+I21hcCB7cG9zaXRpb246YWJzb2x1dGU7dG9wOjA7Ym90dG9tOjA7cmlnaHQ6MDtsZWZ0OjA7fTwvc3R5bGU+CiAgICAKICAgICAgICAgICAgPG1ldGEgbmFtZT0idmlld3BvcnQiIGNvbnRlbnQ9IndpZHRoPWRldmljZS13aWR0aCwKICAgICAgICAgICAgICAgIGluaXRpYWwtc2NhbGU9MS4wLCBtYXhpbXVtLXNjYWxlPTEuMCwgdXNlci1zY2FsYWJsZT1ubyIgLz4KICAgICAgICAgICAgPHN0eWxlPgogICAgICAgICAgICAgICAgI21hcF9iN2UwN2ViYzExZWI0NTcwYjM1ODFmMjc4NzA1ZTY4NCB7CiAgICAgICAgICAgICAgICAgICAgcG9zaXRpb246IHJlbGF0aXZlOwogICAgICAgICAgICAgICAgICAgIHdpZHRoOiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgbGVmdDogMC4wJTsKICAgICAgICAgICAgICAgICAgICB0b3A6IDAuMCU7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgIDwvc3R5bGU+CiAgICAgICAgCjwvaGVhZD4KPGJvZHk+ICAgIAogICAgCiAgICAgICAgICAgIDxkaXYgY2xhc3M9ImZvbGl1bS1tYXAiIGlkPSJtYXBfYjdlMDdlYmMxMWViNDU3MGIzNTgxZjI3ODcwNWU2ODQiID48L2Rpdj4KICAgICAgICAKPC9ib2R5Pgo8c2NyaXB0PiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFwX2I3ZTA3ZWJjMTFlYjQ1NzBiMzU4MWYyNzg3MDVlNjg0ID0gTC5tYXAoCiAgICAgICAgICAgICAgICAibWFwX2I3ZTA3ZWJjMTFlYjQ1NzBiMzU4MWYyNzg3MDVlNjg0IiwKICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICBjZW50ZXI6IFstMzUuMjk3NTkwNiwgMTQ5LjEwMTI2NzZdLAogICAgICAgICAgICAgICAgICAgIGNyczogTC5DUlMuRVBTRzM4NTcsCiAgICAgICAgICAgICAgICAgICAgem9vbTogNiwKICAgICAgICAgICAgICAgICAgICB6b29tQ29udHJvbDogdHJ1ZSwKICAgICAgICAgICAgICAgICAgICBwcmVmZXJDYW52YXM6IGZhbHNlLAogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApOwoKICAgICAgICAgICAgCgogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciB0aWxlX2xheWVyXzVmYTYyMDIxZjMyYzQ5ZjM4NWI3MzJmZTM5MGZkNTY1ID0gTC50aWxlTGF5ZXIoCiAgICAgICAgICAgICAgICAiaHR0cHM6Ly97c30udGlsZS5vcGVuc3RyZWV0bWFwLm9yZy97en0ve3h9L3t5fS5wbmciLAogICAgICAgICAgICAgICAgeyJhdHRyaWJ1dGlvbiI6ICJEYXRhIGJ5IFx1MDAyNmNvcHk7IFx1MDAzY2EgaHJlZj1cImh0dHA6Ly9vcGVuc3RyZWV0bWFwLm9yZ1wiXHUwMDNlT3BlblN0cmVldE1hcFx1MDAzYy9hXHUwMDNlLCB1bmRlciBcdTAwM2NhIGhyZWY9XCJodHRwOi8vd3d3Lm9wZW5zdHJlZXRtYXAub3JnL2NvcHlyaWdodFwiXHUwMDNlT0RiTFx1MDAzYy9hXHUwMDNlLiIsICJkZXRlY3RSZXRpbmEiOiBmYWxzZSwgIm1heE5hdGl2ZVpvb20iOiAxOCwgIm1heFpvb20iOiAxOCwgIm1pblpvb20iOiAwLCAibm9XcmFwIjogZmFsc2UsICJvcGFjaXR5IjogMSwgInN1YmRvbWFpbnMiOiAiYWJjIiwgInRtcyI6IGZhbHNlfQogICAgICAgICAgICApLmFkZFRvKG1hcF9iN2UwN2ViYzExZWI0NTcwYjM1ODFmMjc4NzA1ZTY4NCk7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfOGFmM2IxZjI1YjMzNDI3ZmJhYWRlN2E0ODhhNWRjZjIgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMzUuMjgyMDAxLCAxNDkuMTI4OTk4XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfYjdlMDdlYmMxMWViNDU3MGIzNTgxZjI3ODcwNWU2ODQpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzM3MTIyOTg3NGYzODRiYzdhOTc0NzVjN2Y3M2ViNWVjID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTM1LjM0Mzc4NCwgMTQ5LjA4Mjk3N10sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwX2I3ZTA3ZWJjMTFlYjQ1NzBiMzU4MWYyNzg3MDVlNjg0KTsKICAgICAgICAKPC9zY3JpcHQ+" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




```python
city_latitude = act_data.loc[0, "Latitude"]
city_longitude = act_data.loc[0, "Longitude"]
city_name = act_data.loc[0, "City"]
print("The Latitude and Longitude of {} are {}, {}.".format(city_name, city_latitude, city_longitude)) 
```

    The Latitude and Longitude of Canberra are -35.282001, 149.128998.
    


```python
radius = 5000
limit = 100
query = "hospitals"
url2 = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&ll={},{}&v={}&radius={}&limit={}&query={}'.format(CLIENT_ID, CLIENT_SECRET, city_latitude, city_longitude, VERSION, radius, limit, query)
url2
```




    'https://api.foursquare.com/v2/venues/explore?&client_id=L351DHR2FSTSDETU0K5X3IZMC5XCTH5GJMOTYKAE0BEVPVO3&client_secret=U0NV33KA4DBR4L5ZDYKM2XXXOO4RFH5DI1XNRFUGJDIF2MVR&ll=-35.282001,149.128998&v=20180605&radius=5000&limit=100&query=hospitals'




```python
results2 = requests.get(url2).json()
results2
```




    {'meta': {'code': 200, 'requestId': '5d73a793bbed21003819baaf'},
     'response': {'warning': {'text': 'There aren\'t a lot of results for "hospitals." Try something more general, reset your filters, or expand the search area.'},
      'headerLocation': 'Canberra',
      'headerFullLocation': 'Canberra',
      'headerLocationGranularity': 'city',
      'query': 'hospitals',
      'totalResults': 2,
      'suggestedBounds': {'ne': {'lat': -35.23700095499996,
        'lng': 149.18402063259512},
       'sw': {'lat': -35.327001045000046, 'lng': 149.07397536740487}},
      'groups': [{'type': 'Recommended Places',
        'name': 'recommended',
        'items': [{'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4e7836f9b0fb305db5bcdb05',
           'name': 'Simpson Optometry',
           'location': {'address': '70 Bunda Street',
            'crossStreet': 'at Garema Pl.',
            'lat': -35.27795,
            'lng': 149.1322,
            'labeledLatLngs': [{'label': 'display',
              'lat': -35.27795,
              'lng': 149.1322}],
            'distance': 536,
            'postalCode': '2601',
            'cc': 'AU',
            'state': 'Australian Capital Territory',
            'country': 'Australia',
            'formattedAddress': ['70 Bunda Street (at Garema Pl.)',
             'City ACT 2601',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '509140932'}},
          'referralId': 'e-0-4e7836f9b0fb305db5bcdb05-0'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b8fdcd2f964a520186633e3',
           'name': 'Calvary Hospital',
           'location': {'address': 'Mary Potter Cct.',
            'crossStreet': 'at Haydon Dr.',
            'lat': -35.25309153571022,
            'lng': 149.0900700499176,
            'labeledLatLngs': [{'label': 'display',
              'lat': -35.25309153571022,
              'lng': 149.0900700499176}],
            'distance': 4782,
            'postalCode': '2617',
            'cc': 'AU',
            'city': 'Bruce',
            'state': 'ACT',
            'country': 'Australia',
            'formattedAddress': ['Mary Potter Cct. (at Haydon Dr.)',
             'Bruce ACT 2617',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b8fdcd2f964a520186633e3-1'}]}]}}




```python
def get_category_type(row):
    try:
        categories_list2 = row['categories']
    except:
        categories_list2 = row['venue.categories']
    if len(categories_list2) == 0:
        return None
    else:
        return categories_list2[0]['name']
```


```python
venues2 = results2['response']['groups'][0]['items']
nearby_venues2 = json_normalize(venues2)
filtered_columns = ['venue.name', 'venue.categories', 'venue.location.latitude', 'venue.location.longitude']
nearby_venues2 = nearby_venues2.loc[:, filtered_columns]
nearby_venues2['venue.categories'] = nearby_venues2.apply(get_category_type, axis = 1)
nearby_venues2.columns = [col.split(".")[-1] for col in nearby_venues2.columns]
print("The size of the dataframe is:", nearby_venues2.shape)
nearby_venues2
```

    The size of the dataframe is: (2, 4)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>categories</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Simpson Optometry</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Calvary Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Queensland hospital data
```


```python
qld_data = city_data[city_data['State'].str.contains('QLD')].reset_index(drop = True)
print("The shape of the dataframe:", qld_data.shape)
qld_data[0:10]
```

    The shape of the dataframe: (7, 4)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>State</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Sunshine Coast</td>
      <td>-26.650000</td>
      <td>153.066666</td>
      <td>QLD</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Gold Coast</td>
      <td>-28.016666</td>
      <td>153.399994</td>
      <td>QLD</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Townsville City</td>
      <td>-19.258965</td>
      <td>146.816956</td>
      <td>QLD</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Cairns City</td>
      <td>-16.925491</td>
      <td>145.754120</td>
      <td>QLD</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Brisbane</td>
      <td>-27.470125</td>
      <td>153.021072</td>
      <td>QLD</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Gladstone</td>
      <td>-23.843138</td>
      <td>151.268356</td>
      <td>QLD</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Glenore Grove</td>
      <td>-27.529953</td>
      <td>152.407181</td>
      <td>QLD</td>
    </tr>
  </tbody>
</table>
</div>




```python
address = 'Brisbane'
geolocator = Nominatim(user_agent = 'my_application')
location = geolocator.geocode(address)
latitude = location.latitude
longitude = location.longitude
print("The geographical coordinates of Brisbane are {},{}.".format(latitude, longitude))
```

    The geographical coordinates of Brisbane are -27.4689682,153.0234991.
    


```python
nsw_map3 = folium.Map(location = [latitude, longitude], zoom_start = 6)
for lat, lon, city, state in zip(qld_data['Latitude'], qld_data['Longitude'], qld_data['City'], qld_data['State']):
    label = '{}'.format(city, state)
    label = folium.Popup(label, parse_html = True)
    folium.CircleMarker(
       [lat, lon],
       radius = 5,
       color = 'blue',
       fill = True,
       fill_color = '#3186cc',
       fill_opacity = 0.7,
       parse_html = False).add_to(nsw_map3)
nsw_map3       
```




<div style="width:100%;"><div style="position:relative;width:100%;height:0;padding-bottom:60%;"><iframe src="data:text/html;charset=utf-8;base64,PCFET0NUWVBFIGh0bWw+CjxoZWFkPiAgICAKICAgIDxtZXRhIGh0dHAtZXF1aXY9ImNvbnRlbnQtdHlwZSIgY29udGVudD0idGV4dC9odG1sOyBjaGFyc2V0PVVURi04IiAvPgogICAgCiAgICAgICAgPHNjcmlwdD4KICAgICAgICAgICAgTF9OT19UT1VDSCA9IGZhbHNlOwogICAgICAgICAgICBMX0RJU0FCTEVfM0QgPSBmYWxzZTsKICAgICAgICA8L3NjcmlwdD4KICAgIAogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuanMiPjwvc2NyaXB0PgogICAgPHNjcmlwdCBzcmM9Imh0dHBzOi8vY29kZS5qcXVlcnkuY29tL2pxdWVyeS0xLjEyLjQubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9qcy9ib290c3RyYXAubWluLmpzIj48L3NjcmlwdD4KICAgIDxzY3JpcHQgc3JjPSJodHRwczovL2NkbmpzLmNsb3VkZmxhcmUuY29tL2FqYXgvbGlicy9MZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy8yLjAuMi9sZWFmbGV0LmF3ZXNvbWUtbWFya2Vycy5qcyI+PC9zY3JpcHQ+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuLmpzZGVsaXZyLm5ldC9ucG0vbGVhZmxldEAxLjUuMS9kaXN0L2xlYWZsZXQuY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vbWF4Y2RuLmJvb3RzdHJhcGNkbi5jb20vYm9vdHN0cmFwLzMuMi4wL2Nzcy9ib290c3RyYXAubWluLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL21heGNkbi5ib290c3RyYXBjZG4uY29tL2Jvb3RzdHJhcC8zLjIuMC9jc3MvYm9vdHN0cmFwLXRoZW1lLm1pbi5jc3MiLz4KICAgIDxsaW5rIHJlbD0ic3R5bGVzaGVldCIgaHJlZj0iaHR0cHM6Ly9tYXhjZG4uYm9vdHN0cmFwY2RuLmNvbS9mb250LWF3ZXNvbWUvNC42LjMvY3NzL2ZvbnQtYXdlc29tZS5taW4uY3NzIi8+CiAgICA8bGluayByZWw9InN0eWxlc2hlZXQiIGhyZWY9Imh0dHBzOi8vY2RuanMuY2xvdWRmbGFyZS5jb20vYWpheC9saWJzL0xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLzIuMC4yL2xlYWZsZXQuYXdlc29tZS1tYXJrZXJzLmNzcyIvPgogICAgPGxpbmsgcmVsPSJzdHlsZXNoZWV0IiBocmVmPSJodHRwczovL3Jhd2Nkbi5naXRoYWNrLmNvbS9weXRob24tdmlzdWFsaXphdGlvbi9mb2xpdW0vbWFzdGVyL2ZvbGl1bS90ZW1wbGF0ZXMvbGVhZmxldC5hd2Vzb21lLnJvdGF0ZS5jc3MiLz4KICAgIDxzdHlsZT5odG1sLCBib2R5IHt3aWR0aDogMTAwJTtoZWlnaHQ6IDEwMCU7bWFyZ2luOiAwO3BhZGRpbmc6IDA7fTwvc3R5bGU+CiAgICA8c3R5bGU+I21hcCB7cG9zaXRpb246YWJzb2x1dGU7dG9wOjA7Ym90dG9tOjA7cmlnaHQ6MDtsZWZ0OjA7fTwvc3R5bGU+CiAgICAKICAgICAgICAgICAgPG1ldGEgbmFtZT0idmlld3BvcnQiIGNvbnRlbnQ9IndpZHRoPWRldmljZS13aWR0aCwKICAgICAgICAgICAgICAgIGluaXRpYWwtc2NhbGU9MS4wLCBtYXhpbXVtLXNjYWxlPTEuMCwgdXNlci1zY2FsYWJsZT1ubyIgLz4KICAgICAgICAgICAgPHN0eWxlPgogICAgICAgICAgICAgICAgI21hcF80NDRiZTYwYTNlYjI0MWM4YWRlOWZhNTM2MTE1MmIwOSB7CiAgICAgICAgICAgICAgICAgICAgcG9zaXRpb246IHJlbGF0aXZlOwogICAgICAgICAgICAgICAgICAgIHdpZHRoOiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgaGVpZ2h0OiAxMDAuMCU7CiAgICAgICAgICAgICAgICAgICAgbGVmdDogMC4wJTsKICAgICAgICAgICAgICAgICAgICB0b3A6IDAuMCU7CiAgICAgICAgICAgICAgICB9CiAgICAgICAgICAgIDwvc3R5bGU+CiAgICAgICAgCjwvaGVhZD4KPGJvZHk+ICAgIAogICAgCiAgICAgICAgICAgIDxkaXYgY2xhc3M9ImZvbGl1bS1tYXAiIGlkPSJtYXBfNDQ0YmU2MGEzZWIyNDFjOGFkZTlmYTUzNjExNTJiMDkiID48L2Rpdj4KICAgICAgICAKPC9ib2R5Pgo8c2NyaXB0PiAgICAKICAgIAogICAgICAgICAgICB2YXIgbWFwXzQ0NGJlNjBhM2ViMjQxYzhhZGU5ZmE1MzYxMTUyYjA5ID0gTC5tYXAoCiAgICAgICAgICAgICAgICAibWFwXzQ0NGJlNjBhM2ViMjQxYzhhZGU5ZmE1MzYxMTUyYjA5IiwKICAgICAgICAgICAgICAgIHsKICAgICAgICAgICAgICAgICAgICBjZW50ZXI6IFstMjcuNDY4OTY4MiwgMTUzLjAyMzQ5OTFdLAogICAgICAgICAgICAgICAgICAgIGNyczogTC5DUlMuRVBTRzM4NTcsCiAgICAgICAgICAgICAgICAgICAgem9vbTogNiwKICAgICAgICAgICAgICAgICAgICB6b29tQ29udHJvbDogdHJ1ZSwKICAgICAgICAgICAgICAgICAgICBwcmVmZXJDYW52YXM6IGZhbHNlLAogICAgICAgICAgICAgICAgfQogICAgICAgICAgICApOwoKICAgICAgICAgICAgCgogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciB0aWxlX2xheWVyX2JlY2ZiZGFlOGE4OTRlZWY4ODkyMWYyYTNkNDg4ZWJmID0gTC50aWxlTGF5ZXIoCiAgICAgICAgICAgICAgICAiaHR0cHM6Ly97c30udGlsZS5vcGVuc3RyZWV0bWFwLm9yZy97en0ve3h9L3t5fS5wbmciLAogICAgICAgICAgICAgICAgeyJhdHRyaWJ1dGlvbiI6ICJEYXRhIGJ5IFx1MDAyNmNvcHk7IFx1MDAzY2EgaHJlZj1cImh0dHA6Ly9vcGVuc3RyZWV0bWFwLm9yZ1wiXHUwMDNlT3BlblN0cmVldE1hcFx1MDAzYy9hXHUwMDNlLCB1bmRlciBcdTAwM2NhIGhyZWY9XCJodHRwOi8vd3d3Lm9wZW5zdHJlZXRtYXAub3JnL2NvcHlyaWdodFwiXHUwMDNlT0RiTFx1MDAzYy9hXHUwMDNlLiIsICJkZXRlY3RSZXRpbmEiOiBmYWxzZSwgIm1heE5hdGl2ZVpvb20iOiAxOCwgIm1heFpvb20iOiAxOCwgIm1pblpvb20iOiAwLCAibm9XcmFwIjogZmFsc2UsICJvcGFjaXR5IjogMSwgInN1YmRvbWFpbnMiOiAiYWJjIiwgInRtcyI6IGZhbHNlfQogICAgICAgICAgICApLmFkZFRvKG1hcF80NDRiZTYwYTNlYjI0MWM4YWRlOWZhNTM2MTE1MmIwOSk7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNjBlNTJiNzA0ODIxNDc3MWFiYzY4ZGNhNmI3YTc0N2UgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMjYuNjUsIDE1My4wNjY2NjZdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF80NDRiZTYwYTNlYjI0MWM4YWRlOWZhNTM2MTE1MmIwOSk7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfMzJhZTU1NmFlOWU2NGFkNjhkN2E0Y2YzMGEzZTQ0MTQgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMjguMDE2NjY1OTk5OTk5OTk3LCAxNTMuMzk5OTk0XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfNDQ0YmU2MGEzZWIyNDFjOGFkZTlmYTUzNjExNTJiMDkpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyX2QyMjg0ZGM4NDRhNjRkZjI5NmNmMjU2YWIwNjQ2NmYxID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTE5LjI1ODk2NSwgMTQ2LjgxNjk1Nl0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzQ0NGJlNjBhM2ViMjQxYzhhZGU5ZmE1MzYxMTUyYjA5KTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl8zNDE2ZDQwYjAwNTQ0MTRiODRmNDc5YjVmM2U1YTU4NCA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0xNi45MjU0OTEsIDE0NS43NTQxMl0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzQ0NGJlNjBhM2ViMjQxYzhhZGU5ZmE1MzYxMTUyYjA5KTsKICAgICAgICAKICAgIAogICAgICAgICAgICB2YXIgY2lyY2xlX21hcmtlcl83MzRiN2ZjMzg5YWM0MWY1ODU1ZjYyMDBjYTI4NDFkMiA9IEwuY2lyY2xlTWFya2VyKAogICAgICAgICAgICAgICAgWy0yNy40NzAxMjUsIDE1My4wMjEwNzJdLAogICAgICAgICAgICAgICAgeyJidWJibGluZ01vdXNlRXZlbnRzIjogdHJ1ZSwgImNvbG9yIjogImJsdWUiLCAiZGFzaEFycmF5IjogbnVsbCwgImRhc2hPZmZzZXQiOiBudWxsLCAiZmlsbCI6IHRydWUsICJmaWxsQ29sb3IiOiAiIzMxODZjYyIsICJmaWxsT3BhY2l0eSI6IDAuNywgImZpbGxSdWxlIjogImV2ZW5vZGQiLCAibGluZUNhcCI6ICJyb3VuZCIsICJsaW5lSm9pbiI6ICJyb3VuZCIsICJvcGFjaXR5IjogMS4wLCAicmFkaXVzIjogNSwgInN0cm9rZSI6IHRydWUsICJ3ZWlnaHQiOiAzfQogICAgICAgICAgICApLmFkZFRvKG1hcF80NDRiZTYwYTNlYjI0MWM4YWRlOWZhNTM2MTE1MmIwOSk7CiAgICAgICAgCiAgICAKICAgICAgICAgICAgdmFyIGNpcmNsZV9tYXJrZXJfNGZhOGE5NjQyMWQ2NGMyY2I1NTg3ZWU5MjhiZDFlOGMgPSBMLmNpcmNsZU1hcmtlcigKICAgICAgICAgICAgICAgIFstMjMuODQzMTM4LCAxNTEuMjY4MzU2XSwKICAgICAgICAgICAgICAgIHsiYnViYmxpbmdNb3VzZUV2ZW50cyI6IHRydWUsICJjb2xvciI6ICJibHVlIiwgImRhc2hBcnJheSI6IG51bGwsICJkYXNoT2Zmc2V0IjogbnVsbCwgImZpbGwiOiB0cnVlLCAiZmlsbENvbG9yIjogIiMzMTg2Y2MiLCAiZmlsbE9wYWNpdHkiOiAwLjcsICJmaWxsUnVsZSI6ICJldmVub2RkIiwgImxpbmVDYXAiOiAicm91bmQiLCAibGluZUpvaW4iOiAicm91bmQiLCAib3BhY2l0eSI6IDEuMCwgInJhZGl1cyI6IDUsICJzdHJva2UiOiB0cnVlLCAid2VpZ2h0IjogM30KICAgICAgICAgICAgKS5hZGRUbyhtYXBfNDQ0YmU2MGEzZWIyNDFjOGFkZTlmYTUzNjExNTJiMDkpOwogICAgICAgIAogICAgCiAgICAgICAgICAgIHZhciBjaXJjbGVfbWFya2VyXzdmOWI2NzQ2NzEwMjQ4NDc5MWRiMDc4ZTEwMzUxN2QxID0gTC5jaXJjbGVNYXJrZXIoCiAgICAgICAgICAgICAgICBbLTI3LjUyOTk1MzAwMDAwMDAwMywgMTUyLjQwNzE4MV0sCiAgICAgICAgICAgICAgICB7ImJ1YmJsaW5nTW91c2VFdmVudHMiOiB0cnVlLCAiY29sb3IiOiAiYmx1ZSIsICJkYXNoQXJyYXkiOiBudWxsLCAiZGFzaE9mZnNldCI6IG51bGwsICJmaWxsIjogdHJ1ZSwgImZpbGxDb2xvciI6ICIjMzE4NmNjIiwgImZpbGxPcGFjaXR5IjogMC43LCAiZmlsbFJ1bGUiOiAiZXZlbm9kZCIsICJsaW5lQ2FwIjogInJvdW5kIiwgImxpbmVKb2luIjogInJvdW5kIiwgIm9wYWNpdHkiOiAxLjAsICJyYWRpdXMiOiA1LCAic3Ryb2tlIjogdHJ1ZSwgIndlaWdodCI6IDN9CiAgICAgICAgICAgICkuYWRkVG8obWFwXzQ0NGJlNjBhM2ViMjQxYzhhZGU5ZmE1MzYxMTUyYjA5KTsKICAgICAgICAKPC9zY3JpcHQ+" style="position:absolute;width:100%;height:100%;left:0;top:0;border:none !important;" allowfullscreen webkitallowfullscreen mozallowfullscreen></iframe></div></div>




```python
city_latitude = qld_data.loc[4, "Latitude"]
city_longitude = qld_data.loc[4, "Longitude"]
city_name = qld_data.loc[4, "City"]
print("The Latitude and Longitude of {} are {}, {}.".format(city_name, city_latitude, city_longitude)) 
```

    The Latitude and Longitude of Brisbane are -27.470125, 153.021072.
    


```python
radius = 5000
limit = 100
query = "hospitals"
url3 = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&ll={},{}&v={}&radius={}&limit={}&query={}'.format(CLIENT_ID, CLIENT_SECRET, city_latitude, city_longitude, VERSION, radius, limit, query)
url3
```




    'https://api.foursquare.com/v2/venues/explore?&client_id=L351DHR2FSTSDETU0K5X3IZMC5XCTH5GJMOTYKAE0BEVPVO3&client_secret=U0NV33KA4DBR4L5ZDYKM2XXXOO4RFH5DI1XNRFUGJDIF2MVR&ll=-27.470125,153.021072&v=20180605&radius=5000&limit=100&query=hospitals'




```python
results3 = requests.get(url3).json()
results3
```




    {'meta': {'code': 200, 'requestId': '5d73a79686bc49002cc121d0'},
     'response': {'headerLocation': 'Brisbane',
      'headerFullLocation': 'Brisbane',
      'headerLocationGranularity': 'city',
      'query': 'hospitals',
      'totalResults': 29,
      'suggestedBounds': {'ne': {'lat': -27.425124954999955,
        'lng': 153.0716957813864},
       'sw': {'lat': -27.515125045000044, 'lng': 152.9704482186136}},
      'groups': [{'type': 'Recommended Places',
        'name': 'recommended',
        'items': [{'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4c1087ebce57c928891382d2',
           'name': 'Brisbane Private Hospital',
           'location': {'address': '259 Wickham Terrace',
            'lat': -27.4646367,
            'lng': 153.0225792,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.4646367,
              'lng': 153.0225792}],
            'distance': 628,
            'postalCode': '4000',
            'cc': 'AU',
            'city': 'Brisbane',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['259 Wickham Terrace',
             'Brisbane QLD 4000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '554984978'}},
          'referralId': 'e-0-4c1087ebce57c928891382d2-0'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4fbecca2e4b07160bc7361c4',
           'name': 'Eyes on Edward',
           'location': {'address': '57 Edward Street',
            'lat': -27.471004324943973,
            'lng': 153.02938594672327,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.471004324943973,
              'lng': 153.02938594672327}],
            'distance': 826,
            'postalCode': '4000',
            'cc': 'AU',
            'city': 'Brisbane',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['57 Edward Street',
             'Brisbane QLD 4000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '496470014'}},
          'referralId': 'e-0-4fbecca2e4b07160bc7361c4-1'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b6bad6cf964a5203c152ce3',
           'name': "St Andrew's Hospital",
           'location': {'address': '457 Wickham Terrace',
            'lat': -27.461170887927572,
            'lng': 153.02107862576773,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.461170887927572,
              'lng': 153.02107862576773}],
            'distance': 996,
            'postalCode': '4000',
            'cc': 'AU',
            'city': 'Spring Hill',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['457 Wickham Terrace',
             'Spring Hill QLD 4000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b6bad6cf964a5203c152ce3-2'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b136e44f964a520fc9623e3',
           'name': 'Mater Adult Hospital',
           'location': {'address': 'Raymond Terrace',
            'crossStreet': 'Btw Annerley Rd. & Stanley St.',
            'lat': -27.48520535545116,
            'lng': 153.02823071400553,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.48520535545116,
              'lng': 153.02823071400553}],
            'distance': 1821,
            'postalCode': '4101',
            'cc': 'AU',
            'city': 'South Brisbane',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['Raymond Terrace (Btw Annerley Rd. & Stanley St.)',
             'South Brisbane QLD 4101',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b136e44f964a520fc9623e3-3'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4cc542cd914137043e10c855',
           'name': "St Vincent's Hospital",
           'location': {'address': '411 Main St.',
            'lat': -27.47362,
            'lng': 153.03499,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.47362,
              'lng': 153.03499}],
            'distance': 1428,
            'postalCode': '4169',
            'cc': 'AU',
            'city': 'Kangaroo Point',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['411 Main St.',
             'Kangaroo Point QLD 4169',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4cc542cd914137043e10c855-4'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '5ba50551e1f2280039fe39ee',
           'name': 'Homlab Homeopathic Clinic',
           'location': {'address': '2/72 Vulture Street',
            'lat': -27.48071,
            'lng': 153.01148,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.48071,
              'lng': 153.01148}],
            'distance': 1511,
            'postalCode': '4101',
            'cc': 'AU',
            'city': 'West End',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['2/72 Vulture Street',
             'West End QLD 4101',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '511683490'}},
          'referralId': 'e-0-5ba50551e1f2280039fe39ee-5'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '5013577ee4b0b96017bcf0e7',
           'name': "Queensland Children's Hospital",
           'location': {'address': 'Stanley Street',
            'lat': -27.48379797178258,
            'lng': 153.02591350441637,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.48379797178258,
              'lng': 153.02591350441637}],
            'distance': 1595,
            'cc': 'AU',
            'city': 'South Brisbane',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['Stanley Street',
             'South Brisbane QLD',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-5013577ee4b0b96017bcf0e7-6'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '53ebeb5a498e6c9591c8846c',
           'name': 'Lady Cilento Childrens Hospital (LCCH)',
           'location': {'address': 'Raymond Terrace',
            'lat': -27.48377382472694,
            'lng': 153.02641229478692,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.48377382472694,
              'lng': 153.02641229478692}],
            'distance': 1608,
            'postalCode': '4101',
            'cc': 'AU',
            'city': 'South Brisbane',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['Raymond Terrace',
             'South Brisbane QLD 4101',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-53ebeb5a498e6c9591c8846c-7'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4cc4b3f538aaa09382a91462',
           'name': 'Mater Medical Centre',
           'location': {'address': '293 Vulture St.',
            'lat': -27.48403,
            'lng': 153.02795,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.48403,
              'lng': 153.02795}],
            'distance': 1690,
            'postalCode': '4101',
            'cc': 'AU',
            'city': 'South Brisbane',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['293 Vulture St.',
             'South Brisbane QLD 4101',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4cc4b3f538aaa09382a91462-8'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b331ee3f964a520ba1725e3',
           'name': "Mater Children's Private Hospital",
           'location': {'address': 'Raymond Tce.',
            'lat': -27.484420110413158,
            'lng': 153.0279314233998,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.484420110413158,
              'lng': 153.0279314233998}],
            'distance': 1729,
            'postalCode': '4508',
            'cc': 'AU',
            'city': 'East Brisbane',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['Raymond Tce.',
             'East Brisbane QLD 4508',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b331ee3f964a520ba1725e3-9'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '513ea7cfe4b0d0aa94fea10a',
           'name': "Salmon Building (ex Mater Children's Hospital)",
           'location': {'address': 'Stanley St',
            'crossStreet': 'Raymond Tce',
            'lat': -27.48454470778856,
            'lng': 153.02798856585179,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.48454470778856,
              'lng': 153.02798856585179}],
            'distance': 1744,
            'postalCode': '4101',
            'cc': 'AU',
            'city': 'South Brisbane',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['Stanley St (Raymond Tce)',
             'South Brisbane QLD 4101',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-513ea7cfe4b0d0aa94fea10a-10'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b70b30ef964a520622a2de3',
           'name': 'Mater Private Hospital',
           'location': {'address': '27-43 Raymond Tce',
            'lat': -27.48457480034943,
            'lng': 153.02821312871842,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.48457480034943,
              'lng': 153.02821312871842}],
            'distance': 1756,
            'postalCode': '4101',
            'cc': 'AU',
            'city': 'South Brisbane',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['27-43 Raymond Tce',
             'South Brisbane QLD 4101',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b70b30ef964a520622a2de3-11'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4e0bfd6118a89ad0106652ad',
           'name': 'Mater Hospital Specialist Clinics',
           'location': {'address': 'Stanley St.',
            'lat': -27.484886,
            'lng': 153.02813839053618,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.484886,
              'lng': 153.02813839053618}],
            'distance': 1785,
            'postalCode': '4102',
            'cc': 'AU',
            'city': 'Woolloongabba',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['Stanley St.',
             'Woolloongabba QLD 4102',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4e0bfd6118a89ad0106652ad-12'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b556fd0f964a5204be427e3',
           'name': "Mater Mothers' Hospital",
           'location': {'address': 'Raymond Tce.',
            'lat': -27.485605558770054,
            'lng': 153.02753234964885,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.485605558770054,
              'lng': 153.02753234964885}],
            'distance': 1837,
            'postalCode': '4101',
            'cc': 'AU',
            'city': 'South Brisbane',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['Raymond Tce.',
             'South Brisbane QLD 4101',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b556fd0f964a5204be427e3-13'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4c671ac59cedd13a3db577a1',
           'name': 'Mater Mothers Birthing Suites',
           'location': {'address': '27-43 Raymond Terrace',
            'lat': -27.48607814768606,
            'lng': 153.02753448486328,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.48607814768606,
              'lng': 153.02753448486328}],
            'distance': 1887,
            'postalCode': '4101',
            'cc': 'AU',
            'city': 'South Brisbane',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['27-43 Raymond Terrace',
             'South Brisbane QLD 4101',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4c671ac59cedd13a3db577a1-14'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '5a0a324f446ea60cf15db8e6',
           'name': 'IMATIS PTY LTD',
           'location': {'address': '167 Eagle Street',
            'lat': -27.4661,
            'lng': 153.03067,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.4661,
              'lng': 153.03067}],
            'distance': 1048,
            'postalCode': '4000',
            'cc': 'AU',
            'city': 'Brisbane',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['167 Eagle Street',
             'Brisbane QLD 4000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-5a0a324f446ea60cf15db8e6-15'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b32cb9df964a520101425e3',
           'name': 'The Wesley Hospital',
           'location': {'address': '451 Coronation Dr.',
            'crossStreet': 'Chasley St.',
            'lat': -27.47765189336474,
            'lng': 152.99800955639148,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.47765189336474,
              'lng': 152.99800955639148}],
            'distance': 2426,
            'postalCode': '4066',
            'cc': 'AU',
            'city': 'Auchenflower',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['451 Coronation Dr. (Chasley St.)',
             'Auchenflower QLD 4066',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '513233787'}},
          'referralId': 'e-0-4b32cb9df964a520101425e3-16'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4e0a83abfa761ac4b9ac3f7e',
           'name': 'RBWH Block 7',
           'location': {'address': 'Cnr Butterfield St. and Bowen Bridge Rd.',
            'lat': -27.44906017703294,
            'lng': 153.02832811058877,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.44906017703294,
              'lng': 153.02832811058877}],
            'distance': 2452,
            'postalCode': '4006',
            'cc': 'AU',
            'city': 'Herston',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['Cnr Butterfield St. and Bowen Bridge Rd.',
             'Herston QLD 4006',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4e0a83abfa761ac4b9ac3f7e-17'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4d85f3a711da6a31195ec762',
           'name': 'New Farm Clinic',
           'location': {'address': 'Sargent St.',
            'lat': -27.473559493859582,
            'lng': 153.04564509180375,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.473559493859582,
              'lng': 153.04564509180375}],
            'distance': 2456,
            'postalCode': '4005',
            'cc': 'AU',
            'city': 'New Farm',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['Sargent St.', 'New Farm QLD 4005', 'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4d85f3a711da6a31195ec762-18'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b06613ff964a52053eb22e3',
           'name': "Royal Brisbane & Women's Hospital (RBWH)",
           'location': {'address': 'Bowen Bridge Rd.',
            'crossStreet': 'at Butterfield St.',
            'lat': -27.447233868603774,
            'lng': 153.02730114609002,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.447233868603774,
              'lng': 153.02730114609002}],
            'distance': 2621,
            'postalCode': '4006',
            'cc': 'AU',
            'city': 'Herston',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['Bowen Bridge Rd. (at Butterfield St.)',
             'Herston QLD 4006',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b06613ff964a52053eb22e3-19'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4d4104ad1bd2a143525cf97c',
           'name': 'Hawken Eyes',
           'location': {'address': '2/228 Hawken Drive',
            'lat': -27.50243221287948,
            'lng': 153.0064539344539,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.50243221287948,
              'lng': 153.0064539344539}],
            'distance': 3875,
            'postalCode': '4067',
            'cc': 'AU',
            'city': 'St Lucia',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['2/228 Hawken Drive',
             'St Lucia QLD 4067',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4d4104ad1bd2a143525cf97c-20'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4c06ed0cb4aa0f4734e66462',
           'name': 'RBH Orthopaedic Clinic',
           'location': {'address': 'Bowen Bridge Rd.',
            'lat': -27.447735418091195,
            'lng': 153.02822462690267,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.447735418091195,
              'lng': 153.02822462690267}],
            'distance': 2590,
            'cc': 'AU',
            'city': 'Herston',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['Bowen Bridge Rd.', 'Herston QLD', 'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4c06ed0cb4aa0f4734e66462-21'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '51b546ed498e2f0a08bf145a',
           'name': 'Joyce Tweddell Building',
           'location': {'lat': -27.446928732460957,
            'lng': 153.02733567791014,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.446928732460957,
              'lng': 153.02733567791014}],
            'distance': 2655,
            'cc': 'AU',
            'country': 'Australia',
            'formattedAddress': ['Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-51b546ed498e2f0a08bf145a-22'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4edf0c8f4901ab1d4edbd4a0',
           'name': 'RBWH - Oncology/Haematology',
           'location': {'lat': -27.4466690955894,
            'lng': 153.0276132415698,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.4466690955894,
              'lng': 153.0276132415698}],
            'distance': 2689,
            'cc': 'AU',
            'city': 'Brisbane',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['Brisbane QLD', 'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4edf0c8f4901ab1d4edbd4a0-23'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '51b54563498efbbd1dfec8aa',
           'name': 'Ned Hanlon Building',
           'location': {'lat': -27.44650359634661,
            'lng': 153.0281153941777,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.44650359634661,
              'lng': 153.0281153941777}],
            'distance': 2719,
            'cc': 'AU',
            'country': 'Australia',
            'formattedAddress': ['Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-51b54563498efbbd1dfec8aa-24'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4f0026c9d3e364d5fcb23f6c',
           'name': 'RBWH- ICU',
           'location': {'crossStreet': 'Bowen Bridge Road',
            'lat': -27.446636077962022,
            'lng': 153.02909849072947,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.446636077962022,
              'lng': 153.02909849072947}],
            'distance': 2732,
            'cc': 'AU',
            'city': 'Herston',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['Bowen Bridge Road', 'Herston QLD', 'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4f0026c9d3e364d5fcb23f6c-25'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b0753f6f964a52014fc22e3',
           'name': 'Princess Alexandra Hospital',
           'location': {'address': '199 Ipswich Rd.',
            'lat': -27.499508099502002,
            'lng': 153.0336787072365,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.499508099502002,
              'lng': 153.0336787072365}],
            'distance': 3499,
            'postalCode': '4102',
            'cc': 'AU',
            'neighborhood': 'Wooloongabba',
            'city': 'Woolloongabba',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['199 Ipswich Rd.',
             'Woolloongabba QLD 4102',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b0753f6f964a52014fc22e3-26'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b7de780f964a52012d92fe3',
           'name': 'QIMR Berghofer Central',
           'location': {'address': '300 Herston Rd.',
            'lat': -27.449496437437137,
            'lng': 153.0274068776339,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.449496437437137,
              'lng': 153.0274068776339}],
            'distance': 2380,
            'postalCode': '4006',
            'cc': 'AU',
            'city': 'Herston',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['300 Herston Rd.',
             'Herston QLD 4006',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b7de780f964a52012d92fe3-27'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4d950d6daf3d236a5a8ee9c6',
           'name': 'Ondol Oriental Medicine Clinic',
           'location': {'address': '129 Sylvan Rd',
            'lat': -27.47954,
            'lng': 152.98719,
            'labeledLatLngs': [{'label': 'display',
              'lat': -27.47954,
              'lng': 152.98719}],
            'distance': 3506,
            'postalCode': '4066',
            'cc': 'AU',
            'city': 'Toowong',
            'state': 'QLD',
            'country': 'Australia',
            'formattedAddress': ['129 Sylvan Rd',
             'Toowong QLD 4066',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '495652067'}},
          'referralId': 'e-0-4d950d6daf3d236a5a8ee9c6-28'}]}]}}




```python
def get_category_type(row):
    try:
        categories_list3 = row['categories']
    except:
        categories_list3 = row['venue.categories']
    if len(categories_list3) == 0:
        return None
    else:
        return categories_list3[0]['name']
```


```python
venues3 = results3['response']['groups'][0]['items']
nearby_venues3 = json_normalize(venues3)
filtered_columns = ['venue.name', 'venue.categories', 'venue.location.latitude', 'venue.location.longitude']
nearby_venues3 = nearby_venues3.loc[:, filtered_columns]
nearby_venues3['venue.categories'] = nearby_venues3.apply(get_category_type, axis = 1)
nearby_venues3.columns = [col.split(".")[-1] for col in nearby_venues3.columns]
print("The size of the dataframe is:", nearby_venues3.shape)
nearby_venues3
```

    The size of the dataframe is: (29, 4)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>categories</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Brisbane Private Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Eyes on Edward</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>St Andrew's Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Mater Adult Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>St Vincent's Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Homlab Homeopathic Clinic</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Queensland Children's Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Lady Cilento Childrens Hospital (LCCH)</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Mater Medical Centre</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Mater Children's Private Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Salmon Building (ex Mater Children's Hospital)</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Mater Private Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Mater Hospital Specialist Clinics</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Mater Mothers' Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Mater Mothers Birthing Suites</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>15</th>
      <td>IMATIS PTY LTD</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>16</th>
      <td>The Wesley Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>17</th>
      <td>RBWH Block 7</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>18</th>
      <td>New Farm Clinic</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Royal Brisbane &amp; Women's Hospital (RBWH)</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Hawken Eyes</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>21</th>
      <td>RBH Orthopaedic Clinic</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Joyce Tweddell Building</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>23</th>
      <td>RBWH - Oncology/Haematology</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Ned Hanlon Building</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>25</th>
      <td>RBWH- ICU</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Princess Alexandra Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>27</th>
      <td>QIMR Berghofer Central</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Ondol Oriental Medicine Clinic</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
# South Australia Hospital Data

sa_data = city_data[city_data['State'].str.contains('SA')].reset_index(drop = True)
print("The shape of the dataframe:", sa_data.shape)
sa_data[0:10]
```

    The shape of the dataframe: (3, 4)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>State</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Adelaide</td>
      <td>-34.921230</td>
      <td>138.599503</td>
      <td>SA</td>
    </tr>
    <tr>
      <th>1</th>
      <td>North Adelaide</td>
      <td>-34.906101</td>
      <td>138.593903</td>
      <td>SA</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Mount Gambier</td>
      <td>-37.824429</td>
      <td>140.783783</td>
      <td>SA</td>
    </tr>
  </tbody>
</table>
</div>




```python
address = 'Adelaide'
geolocator = Nominatim(user_agent = 'my_application')
location = geolocator.geocode(address)
latitude = location.latitude
longitude = location.longitude
print("The geographical coordinates of Adelaide are {},{}.".format(latitude, longitude))
```

    The geographical coordinates of Adelaide are -34.9281805,138.5999312.
    


```python
city_latitude = sa_data.loc[0, "Latitude"]
city_longitude = sa_data.loc[0, "Longitude"]
city_name = sa_data.loc[0, "City"]
print("The Latitude and Longitude of {} are {}, {}.".format(city_name, city_latitude, city_longitude)) 
```

    The Latitude and Longitude of Adelaide are -34.92123, 138.599503.
    


```python
radius = 5000
limit = 100
query = "hospitals"
url4 = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&ll={},{}&v={}&radius={}&limit={}&query={}'.format(CLIENT_ID, CLIENT_SECRET, city_latitude, city_longitude, VERSION, radius, limit, query)
url4
```




    'https://api.foursquare.com/v2/venues/explore?&client_id=L351DHR2FSTSDETU0K5X3IZMC5XCTH5GJMOTYKAE0BEVPVO3&client_secret=U0NV33KA4DBR4L5ZDYKM2XXXOO4RFH5DI1XNRFUGJDIF2MVR&ll=-34.92123,138.599503&v=20180605&radius=5000&limit=100&query=hospitals'




```python
results4 = requests.get(url4).json()
results4
```




    {'meta': {'code': 200, 'requestId': '5d73a799c58ed7002c757aa1'},
     'response': {'suggestedFilters': {'header': 'Tap to show:',
       'filters': [{'name': 'Open now', 'key': 'openNow'}]},
      'headerLocation': 'Adelaide',
      'headerFullLocation': 'Adelaide',
      'headerLocationGranularity': 'city',
      'query': 'hospitals',
      'totalResults': 23,
      'suggestedBounds': {'ne': {'lat': -34.87622995499996,
        'lng': 138.65428266022406},
       'sw': {'lat': -34.966230045000046, 'lng': 138.54472333977594}},
      'groups': [{'type': 'Recommended Places',
        'name': 'recommended',
        'items': [{'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4d3a411ecc48224bc0853b4f',
           'name': 'Shades Shop',
           'location': {'address': '101 Rundle Mall',
            'lat': -34.9227761747536,
            'lng': 138.60202985393755,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.9227761747536,
              'lng': 138.60202985393755}],
            'distance': 287,
            'postalCode': '5000',
            'cc': 'AU',
            'city': 'Adelaide',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['101 Rundle Mall',
             'Adelaide SA 5000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '530239671'}},
          'referralId': 'e-0-4d3a411ecc48224bc0853b4f-0'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4e44f330ae605a14b7ecad5e',
           'name': 'Opt Shop Optometry',
           'location': {'address': '36/155 Rundle Mall',
            'lat': -34.92271607550722,
            'lng': 138.60323319429816,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.92271607550722,
              'lng': 138.60323319429816}],
            'distance': 378,
            'postalCode': '5000',
            'cc': 'AU',
            'city': 'Adelaide',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['36/155 Rundle Mall',
             'Adelaide SA 5000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '532985665'}},
          'referralId': 'e-0-4e44f330ae605a14b7ecad5e-1'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4d795f75b359f04d9214c614',
           'name': 'RAH Chest Clinic',
           'location': {'address': '275 North Terrace',
            'crossStreet': 'Frome Street',
            'lat': -34.921095,
            'lng': 138.608184,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.921095,
              'lng': 138.608184}],
            'distance': 792,
            'postalCode': '5001',
            'cc': 'AU',
            'city': 'Adelaide',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['275 North Terrace (Frome Street)',
             'Adelaide SA 5001',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4d795f75b359f04d9214c614-2'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b5392a5f964a520eba227e3',
           'name': 'Adelaide Memorial Hospital',
           'location': {'address': 'Sir Edwin Smith Avenue',
            'lat': -34.913148,
            'lng': 138.6005057,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.913148,
              'lng': 138.6005057}],
            'distance': 904,
            'postalCode': '5006',
            'cc': 'AU',
            'city': 'North Adelaide',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['Sir Edwin Smith Avenue',
             'North Adelaide SA 5006',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '557484165'}},
          'referralId': 'e-0-4b5392a5f964a520eba227e3-3'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4cf76cad9d11b1f7affbc5ed',
           'name': 'CMAX',
           'location': {'address': 'Royal Adelaide Hospital. North Terrace.',
            'lat': -34.92010606783194,
            'lng': 138.60965251922607,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.92010606783194,
              'lng': 138.60965251922607}],
            'distance': 934,
            'postalCode': '5000',
            'cc': 'AU',
            'city': 'Adelaide',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['Royal Adelaide Hospital. North Terrace.',
             'Adelaide SA 5000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4cf76cad9d11b1f7affbc5ed-4'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '5d51fe274d53120008143f70',
           'name': 'Breastscreen SA',
           'location': {'address': '167 Flinders Street',
            'lat': -34.9271526,
            'lng': 138.6068654,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.9271526,
              'lng': 138.6068654}],
            'distance': 941,
            'postalCode': '5000',
            'cc': 'AU',
            'city': 'Adelaide',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['167 Flinders Street',
             'Adelaide SA 5000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-5d51fe274d53120008143f70-5'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '5974507b65cdf8348abf3afd',
           'name': 'New Royal Adelaide Hospital',
           'location': {'lat': -34.922092,
            'lng': 138.58804,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.922092,
              'lng': 138.58804}],
            'distance': 1050,
            'postalCode': '5000',
            'cc': 'AU',
            'city': 'Adelaide',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['Adelaide SA 5000', 'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-5974507b65cdf8348abf3afd-6'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b7df5daf964a52044dc2fe3',
           'name': 'Womens and Childrens Hospital',
           'location': {'address': '72 King William Rd',
            'lat': -34.91141405267055,
            'lng': 138.59979271888733,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.91141405267055,
              'lng': 138.59979271888733}],
            'distance': 1093,
            'postalCode': '5006',
            'cc': 'AU',
            'city': 'North Adelaide',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['72 King William Rd',
             'North Adelaide SA 5006',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '557340144'}},
          'referralId': 'e-0-4b7df5daf964a52044dc2fe3-7'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4ca53f59931bb60cee9c88e2',
           'name': 'Calvary Wakefield Hospital',
           'location': {'address': '277 Wakefield St',
            'lat': -34.927846670294734,
            'lng': 138.61085495536392,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.927846670294734,
              'lng': 138.61085495536392}],
            'distance': 1271,
            'postalCode': '5000',
            'cc': 'AU',
            'city': 'Adelaide',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['277 Wakefield St',
             'Adelaide SA 5000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4ca53f59931bb60cee9c88e2-8'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b6caa24f964a5207e4a2ce3',
           'name': 'Calvary North Adelaide Hospital',
           'location': {'address': '89 Strangways Terrace',
            'crossStreet': 'Ward St.',
            'lat': -34.91025218173691,
            'lng': 138.58888834775388,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.91025218173691,
              'lng': 138.58888834775388}],
            'distance': 1559,
            'postalCode': '5006',
            'cc': 'AU',
            'city': 'North Adelaide',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['89 Strangways Terrace (Ward St.)',
             'North Adelaide SA 5006',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b6caa24f964a5207e4a2ce3-9'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4e49e677fa76a0c058ca558e',
           'name': 'Parkwynd Private Hospital',
           'location': {'address': '137 East Terrace',
            'lat': -34.92848,
            'lng': 138.6142456,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.92848,
              'lng': 138.6142456}],
            'distance': 1569,
            'postalCode': '5000',
            'cc': 'AU',
            'city': 'Adelaide',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['137 East Terrace',
             'Adelaide SA 5000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '553487017'}},
          'referralId': 'e-0-4e49e677fa76a0c058ca558e-10'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4c01c0b43ab80f471db390ff',
           'name': 'Mary Potter Hospice',
           'location': {'address': '89 Strangways Tce',
            'lat': -34.909895719276506,
            'lng': 138.58925453127327,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.909895719276506,
              'lng': 138.58925453127327}],
            'distance': 1570,
            'cc': 'AU',
            'city': 'North Adelaide',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['89 Strangways Tce',
             'North Adelaide SA',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4c01c0b43ab80f471db390ff-11'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '5d37c9beee95980008ebf7ab',
           'name': 'Royal Adelaide Hospital',
           'location': {'address': 'Port Road',
            'lat': -34.9162178,
            'lng': 138.5784482,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.9162178,
              'lng': 138.5784482}],
            'distance': 2001,
            'postalCode': '5000',
            'cc': 'AU',
            'city': 'Adelaide',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['Port Road', 'Adelaide SA 5000', 'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-5d37c9beee95980008ebf7ab-12'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b58d18cf964a520c76c28e3',
           'name': "St Andrew's Hospital",
           'location': {'address': '350 South Tce',
            'crossStreet': 'btw Vincent and St John',
            'lat': -34.93450961001105,
            'lng': 138.6148544990968,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.93450961001105,
              'lng': 138.6148544990968}],
            'distance': 2036,
            'postalCode': '5000',
            'cc': 'AU',
            'city': 'Adelaide',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['350 South Tce (btw Vincent and St John)',
             'Adelaide SA 5000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b58d18cf964a520c76c28e3-13'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4c03647b187ec928c0a7b57b',
           'name': 'The Adelaide Clinic',
           'location': {'address': '33 Park Terrace',
            'lat': -34.90648,
            'lng': 138.61372,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.90648,
              'lng': 138.61372}],
            'distance': 2092,
            'postalCode': '5081',
            'cc': 'AU',
            'city': 'Gilberton',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['33 Park Terrace',
             'Gilberton SA 5081',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4c03647b187ec928c0a7b57b-14'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b7125c4f964a520223b2de3',
           'name': 'Ashford Hospital',
           'location': {'address': '55 Anzac Highway',
            'lat': -34.948207,
            'lng': 138.576731,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.948207,
              'lng': 138.576731}],
            'distance': 3651,
            'postalCode': '5035',
            'cc': 'AU',
            'city': 'Ashford',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['55 Anzac Highway',
             'Ashford SA 5035',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '555738008'}},
          'referralId': 'e-0-4b7125c4f964a520223b2de3-15'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '50178a3b90e7f08dd845a32c',
           'name': 'SPORTSMED·SA Clinic and Hospital',
           'location': {'address': '32 Payneham Road',
            'lat': -34.913631512024644,
            'lng': 138.62489777866503,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.913631512024644,
              'lng': 138.62489777866503}],
            'distance': 2467,
            'postalCode': '5069',
            'cc': 'AU',
            'city': 'Stepney',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['32 Payneham Road',
             'Stepney SA 5069',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-50178a3b90e7f08dd845a32c-16'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4ca99356f47ea14301588021',
           'name': 'Calvary Rehabilitation Hospital',
           'location': {'address': 'North East Rd',
            'lat': -34.89179039844143,
            'lng': 138.61096747956657,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.89179039844143,
              'lng': 138.61096747956657}],
            'distance': 3440,
            'cc': 'AU',
            'state': 'South Australia',
            'country': 'Australia',
            'formattedAddress': ['North East Rd', 'Walkerville SA', 'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4ca99356f47ea14301588021-17'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4dc8959eae60385d1109663a',
           'name': 'Glenside Campus Hospital',
           'location': {'address': 'Amber Woods Dr.',
            'lat': -34.94168595639768,
            'lng': 138.62843595522918,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.94168595639768,
              'lng': 138.62843595522918}],
            'distance': 3486,
            'postalCode': '5065',
            'cc': 'AU',
            'city': 'Glenside',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['Amber Woods Dr.',
             'Glenside SA 5065',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4dc8959eae60385d1109663a-18'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '5d37c9bac46dcb00082a5106',
           'name': 'Helen Mayo House',
           'location': {'address': '226 Fullarton Road',
            'lat': -34.94327390000001,
            'lng': 138.6273346,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.94327390000001,
              'lng': 138.6273346}],
            'distance': 3531,
            'postalCode': '5065',
            'cc': 'AU',
            'city': 'Glenside',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['226 Fullarton Road',
             'Glenside SA 5065',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-5d37c9bac46dcb00082a5106-19'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '5d51fe398232d50008c77f0b',
           'name': 'Glenside Health Services',
           'location': {'address': '226 Fullarton Road',
            'lat': -34.94327390000001,
            'lng': 138.6273346,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.94327390000001,
              'lng': 138.6273346}],
            'distance': 3531,
            'postalCode': '5065',
            'cc': 'AU',
            'city': 'Glenside',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['226 Fullarton Road',
             'Glenside SA 5065',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-5d51fe398232d50008c77f0b-20'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b9959ecf964a5200f7535e3',
           'name': 'Burnside War Memorial Hospital',
           'location': {'address': 'Kensington Rd',
            'lat': -34.9275272997054,
            'lng': 138.63866025018265,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.9275272997054,
              'lng': 138.63866025018265}],
            'distance': 3642,
            'cc': 'AU',
            'city': 'Adelaide',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['Kensington Rd', 'Adelaide SA', 'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b9959ecf964a5200f7535e3-21'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '5d5e797500ce8300078ad3a6',
           'name': 'Podiatry and Orthotics by Adelaide Foot and Ankle',
           'location': {'address': '115 Prospect Road Prospect',
            'lat': -34.8852263,
            'lng': 138.5941281,
            'labeledLatLngs': [{'label': 'display',
              'lat': -34.8852263,
              'lng': 138.5941281}],
            'distance': 4037,
            'postalCode': '5082',
            'cc': 'AU',
            'city': 'Adelaide',
            'state': 'SA',
            'country': 'Australia',
            'formattedAddress': ['115 Prospect Road Prospect',
             'Adelaide SA 5082',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '553883342'}},
          'referralId': 'e-0-5d5e797500ce8300078ad3a6-22'}]}]}}




```python
def get_category_type(row):
    try:
        categories_list4 = row['categories']
    except:
        categories_list4 = row['venue.categories']
    if len(categories_list4) == 0:
        return None
    else:
        return categories_list4[0]['name']
```


```python
venues4 = results4['response']['groups'][0]['items']
nearby_venues4 = json_normalize(venues4)
filtered_columns = ['venue.name', 'venue.categories', 'venue.location.latitude', 'venue.location.longitude']
nearby_venues4 = nearby_venues4.loc[:, filtered_columns]
nearby_venues4['venue.categories'] = nearby_venues4.apply(get_category_type, axis = 1)
nearby_venues4.columns = [col.split(".")[-1] for col in nearby_venues4.columns]
print("The size of the dataframe is:", nearby_venues4.shape)
nearby_venues4
```

    The size of the dataframe is: (23, 4)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>categories</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Shades Shop</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Opt Shop Optometry</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>RAH Chest Clinic</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Adelaide Memorial Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>CMAX</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Breastscreen SA</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td>New Royal Adelaide Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Womens and Childrens Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Calvary Wakefield Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Calvary North Adelaide Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Parkwynd Private Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Mary Potter Hospice</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Royal Adelaide Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>13</th>
      <td>St Andrew's Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>The Adelaide Clinic</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Ashford Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>16</th>
      <td>SPORTSMED·SA Clinic and Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Calvary Rehabilitation Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Glenside Campus Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Helen Mayo House</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Glenside Health Services</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Burnside War Memorial Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Podiatry and Orthotics by Adelaide Foot and Ankle</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Western Australia Hospital information

wa_data = city_data[city_data['State'].str.contains('WA')].reset_index(drop = True)
print("The shape of the dataframe:", wa_data.shape)
wa_data[0:10]
```

    The shape of the dataframe: (1, 4)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>State</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Perth</td>
      <td>-31.953512</td>
      <td>115.857048</td>
      <td>WA</td>
    </tr>
  </tbody>
</table>
</div>




```python
address = 'Perth'
geolocator = Nominatim(user_agent = 'my_application')
location = geolocator.geocode(address)
latitude = location.latitude
longitude = location.longitude
print("The geographical coordinates of Perth are {},{}.".format(latitude, longitude))
```

    The geographical coordinates of Perth are -31.9527121,115.8604796.
    


```python
city_latitude = wa_data.loc[0, "Latitude"]
city_longitude = wa_data.loc[0, "Longitude"]
city_name = wa_data.loc[0, "City"]
print("The Latitude and Longitude of {} are {}, {}.".format(city_name, city_latitude, city_longitude)) 
```

    The Latitude and Longitude of Perth are -31.953512, 115.85704799999999.
    


```python
radius = 5000
limit = 100
query = "hospitals"
url5 = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&ll={},{}&v={}&radius={}&limit={}&query={}'.format(CLIENT_ID, CLIENT_SECRET, city_latitude, city_longitude, VERSION, radius, limit, query)
url5
```




    'https://api.foursquare.com/v2/venues/explore?&client_id=L351DHR2FSTSDETU0K5X3IZMC5XCTH5GJMOTYKAE0BEVPVO3&client_secret=U0NV33KA4DBR4L5ZDYKM2XXXOO4RFH5DI1XNRFUGJDIF2MVR&ll=-31.953512,115.85704799999999&v=20180605&radius=5000&limit=100&query=hospitals'




```python
results5 = requests.get(url5).json()
```


```python
def get_category_type(row):
    try:
        categories_list5 = row['categories']
    except:
        categories_list5 = row['venue.categories']
    if len(categories_list5) == 0:
        return None
    else:
        return categories_list5[0]['name']
```


```python
venues5 = results5['response']['groups'][0]['items']
nearby_venues5 = json_normalize(venues5)
filtered_columns = ['venue.name', 'venue.categories', 'venue.location.latitude', 'venue.location.longitude']
nearby_venues5 = nearby_venues5.loc[:, filtered_columns]
nearby_venues5['venue.categories'] = nearby_venues5.apply(get_category_type, axis = 1)
nearby_venues5.columns = [col.split(".")[-1] for col in nearby_venues5.columns]
print("The size of the dataframe is:", nearby_venues5.shape)
nearby_venues5
```

    The size of the dataframe is: (19, 4)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>categories</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Royal Perth Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Royal Perth Hospital Radiology</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Mount Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Princess Margaret Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Hollywood Private Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>St John of God Mt Lawley Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td>The Atomic Age Eyewear Company</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7</th>
      <td>St John of God Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Genea Hollywood Fertility</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Subiaco Veterinary Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>10</th>
      <td>King Edward Memorial Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Australian Red Cross Blood Service</td>
      <td>Medical Center</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Sir Charles Gairdner Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Perth Children’s Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>SCGH Cancer centre</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Pathwest Pathology</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Queen Elizabeth II Medical Centre</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Rodin Clinic - Plastic Surgery Perth</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>18</th>
      <td>South Perth Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
tas_data = city_data[city_data['State'].str.contains('TAS')].reset_index(drop = True)
print("The shape of the dataframe:", tas_data.shape)
tas_data[0:10]
```

    The shape of the dataframe: (1, 4)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>State</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Launceston</td>
      <td>-41.429825</td>
      <td>147.157135</td>
      <td>TAS</td>
    </tr>
  </tbody>
</table>
</div>




```python
address = 'Launceston'
geolocator = Nominatim(user_agent = 'my_application')
location = geolocator.geocode(address)
latitude = location.latitude
longitude = location.longitude
print("The geographical coordinates of Launceston are {},{}.".format(latitude, longitude))
```

    The geographical coordinates of Launceston are -41.4340813,147.1373496.
    


```python
city_latitude = tas_data.loc[0, "Latitude"]
city_longitude = tas_data.loc[0, "Longitude"]
city_name = tas_data.loc[0, "City"]
print("The Latitude and Longitude of {} are {}, {}.".format(city_name, city_latitude, city_longitude)) 
```

    The Latitude and Longitude of Launceston are -41.429825, 147.157135.
    


```python
radius = 5000
limit = 100
query = "hospitals"
url6 = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&ll={},{}&v={}&radius={}&limit={}&query={}'.format(CLIENT_ID, CLIENT_SECRET, city_latitude, city_longitude, VERSION, radius, limit, query)
url6
```




    'https://api.foursquare.com/v2/venues/explore?&client_id=L351DHR2FSTSDETU0K5X3IZMC5XCTH5GJMOTYKAE0BEVPVO3&client_secret=U0NV33KA4DBR4L5ZDYKM2XXXOO4RFH5DI1XNRFUGJDIF2MVR&ll=-41.429825,147.157135&v=20180605&radius=5000&limit=100&query=hospitals'




```python
results6 = requests.get(url6).json()
```


```python
def get_category_type(row):
    try:
        categories_list6 = row['categories']
    except:
        categories_list6 = row['venue.categories']
    if len(categories_list6) == 0:
        return None
    else:
        return categories_list6[0]['name']
```


```python
venues6 = results6['response']['groups'][0]['items']
nearby_venues6 = json_normalize(venues6)
filtered_columns = ['venue.name', 'venue.categories', 'venue.location.latitude', 'venue.location.longitude']
nearby_venues6 = nearby_venues6.loc[:, filtered_columns]
nearby_venues6['venue.categories'] = nearby_venues6.apply(get_category_type, axis = 1)
nearby_venues6.columns = [col.split(".")[-1] for col in nearby_venues6.columns]
print("The size of the dataframe is:", nearby_venues6.shape)
nearby_venues6
```

    The size of the dataframe is: (4, 4)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>categories</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Calvary Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Tremaur Medical Centre</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>St Vincents Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Launceston General Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Victoria hospital data

vic_data = city_data[city_data['State'].str.contains('VIC')].reset_index(drop = True)
print("The shape of the dataframe:", vic_data.shape)
vic_data[0:10]
```

    The shape of the dataframe: (4, 4)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>State</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Melbourne</td>
      <td>-37.840935</td>
      <td>144.946457</td>
      <td>VIC</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Mildura</td>
      <td>-34.206841</td>
      <td>142.136490</td>
      <td>VIC</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Ziyou Today</td>
      <td>-37.649967</td>
      <td>144.880600</td>
      <td>VIC</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Bendigo</td>
      <td>-36.757786</td>
      <td>144.278702</td>
      <td>VIC</td>
    </tr>
  </tbody>
</table>
</div>




```python
address = 'Melbourne'
geolocator = Nominatim(user_agent = 'my_application')
location = geolocator.geocode(address)
latitude = location.latitude
longitude = location.longitude
print("The geographical coordinates of Melbourne are {},{}.".format(latitude, longitude))
```

    The geographical coordinates of Melbourne are -37.8142176,144.9631608.
    


```python
city_latitude = vic_data.loc[0, "Latitude"]
city_longitude = vic_data.loc[0, "Longitude"]
city_name = vic_data.loc[0, "City"]
print("The Latitude and Longitude of {} are {}, {}.".format(city_name, city_latitude, city_longitude)) 
```

    The Latitude and Longitude of Melbourne are -37.840934999999995, 144.946457.
    


```python
radius = 5000
limit = 100
query = "Hospital"
url7 = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&ll={},{}&v={}&radius={}&limit={}&query={}'.format(CLIENT_ID, CLIENT_SECRET, city_latitude, city_longitude, VERSION, radius, limit, query)
url7
```




    'https://api.foursquare.com/v2/venues/explore?&client_id=L351DHR2FSTSDETU0K5X3IZMC5XCTH5GJMOTYKAE0BEVPVO3&client_secret=U0NV33KA4DBR4L5ZDYKM2XXXOO4RFH5DI1XNRFUGJDIF2MVR&ll=-37.840934999999995,144.946457&v=20180605&radius=5000&limit=100&query=Hospital'




```python
results7 = requests.get(url7).json()
results7
```




    {'meta': {'code': 200, 'requestId': '5d73a7a1a879210038e594f2'},
     'response': {'suggestedFilters': {'header': 'Tap to show:',
       'filters': [{'name': 'Open now', 'key': 'openNow'}]},
      'headerLocation': 'Current map view',
      'headerFullLocation': 'Current map view',
      'headerLocationGranularity': 'unknown',
      'query': 'hospital',
      'totalResults': 35,
      'suggestedBounds': {'ne': {'lat': -37.79593495499995,
        'lng': 145.00333310799172},
       'sw': {'lat': -37.88593504500004, 'lng': 144.8895808920083}},
      'groups': [{'type': 'Recommended Places',
        'name': 'recommended',
        'items': [{'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b206db1f964a520273224e3',
           'name': 'Peter MacCallum Cancer Centre',
           'location': {'address': '305 Grattan St',
            'crossStreet': 'Elizabeth St',
            'lat': -37.81192622596572,
            'lng': 144.97763818594302,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.81192622596572,
              'lng': 144.97763818594302}],
            'distance': 4236,
            'postalCode': '3000',
            'cc': 'AU',
            'city': 'Melbourne',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['305 Grattan St (Elizabeth St)',
             'Melbourne VIC 3000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b206db1f964a520273224e3-0'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4c4f637e92ce2d7f0835b32f',
           'name': 'Alfred Hospital Outpatient Clinics',
           'location': {'lat': -37.84578258242091,
            'lng': 144.98210906982422,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.84578258242091,
              'lng': 144.98210906982422}],
            'distance': 3180,
            'cc': 'AU',
            'city': 'Melbourne',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['Melbourne VIC', 'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4c4f637e92ce2d7f0835b32f-1'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b148587f964a52030a423e3',
           'name': 'The Alfred Hospital',
           'location': {'address': '55 Commercial Rd',
            'lat': -37.84591174295989,
            'lng': 144.98220485110528,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.84591174295989,
              'lng': 144.98220485110528}],
            'distance': 3190,
            'postalCode': '3004',
            'cc': 'AU',
            'city': 'Melbourne',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['55 Commercial Rd',
             'Melbourne VIC 3004',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b148587f964a52030a423e3-2'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '5c2577401953f3002cbfc779',
           'name': 'MyClinic Southbank',
           'location': {'address': '63 Power Street',
            'lat': -37.8238559,
            'lng': 144.9632249,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.8238559,
              'lng': 144.9632249}],
            'distance': 2405,
            'postalCode': '3006',
            'cc': 'AU',
            'city': 'Southbank',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['63 Power Street',
             'Southbank VIC 3006',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '556058689'}},
          'referralId': 'e-0-5c2577401953f3002cbfc779-3'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4c88b6fd944e224b1a491e85',
           'name': 'Centre For Clinical Studies',
           'location': {'address': '89 Commercial Road',
            'lat': -37.84607569985713,
            'lng': 144.98427042057878,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.84607569985713,
              'lng': 144.98427042057878}],
            'distance': 3372,
            'cc': 'AU',
            'city': 'Melbourne',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['89 Commercial Road',
             'Melbourne VIC',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4c88b6fd944e224b1a491e85-4'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '5c6c8732e706550025924b40',
           'name': 'MyClinic St Kilda',
           'location': {'address': '13-15 Grey Street',
            'lat': -37.8602413,
            'lng': 144.9776737,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.8602413,
              'lng': 144.9776737}],
            'distance': 3485,
            'postalCode': '3182',
            'cc': 'AU',
            'city': 'Saint Kilda',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['13-15 Grey Street',
             'St Kilda VIC 3182',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '532074332'}},
          'referralId': 'e-0-5c6c8732e706550025924b40-5'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b184808f964a5202ad023e3',
           'name': 'The Royal Melbourne Hospital',
           'location': {'address': 'Grattan Street',
            'crossStreet': 'Royal Parade',
            'lat': -37.800804405406716,
            'lng': 144.95692538464732,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.800804405406716,
              'lng': 144.95692538464732}],
            'distance': 4561,
            'postalCode': '3050',
            'cc': 'AU',
            'city': 'Parkville',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['Grattan Street (Royal Parade)',
             'Parkville VIC 3050',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '48671056'}},
          'referralId': 'e-0-4b184808f964a5202ad023e3-6'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '5c6c83dff4b52500398cbb04',
           'name': 'MyClinic QV',
           'location': {'address': 'hop 051, Corner Swanston & Lonsdale Street, QV Retail Centre',
            'lat': -37.8111834,
            'lng': 144.964977,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.8111834,
              'lng': 144.964977}],
            'distance': 3690,
            'postalCode': '3000',
            'cc': 'AU',
            'city': 'Melbourne',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['hop 051, Corner Swanston & Lonsdale Street, QV Retail Centre',
             'Melbourne VIC 3000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '532074310'}},
          'referralId': 'e-0-5c6c83dff4b52500398cbb04-7'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b4badacf964a520c7a326e3',
           'name': 'Epworth Richmond',
           'location': {'address': '89 Bridge Rd',
            'crossStreet': 'btw Normanby Pl & Leigh Pl',
            'lat': -37.81736919510412,
            'lng': 144.99339390812543,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.81736919510412,
              'lng': 144.99339390812543}],
            'distance': 4890,
            'postalCode': '3121',
            'cc': 'AU',
            'city': 'Richmond',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['89 Bridge Rd (btw Normanby Pl & Leigh Pl)',
             'Richmond VIC 3121',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b4badacf964a520c7a326e3-8'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4da4ddd1a86ed93def2bce93',
           'name': 'St. Vincents & Mercy Private',
           'location': {'address': '77 Victoria Parade',
            'lat': -37.80814973484444,
            'lng': 144.97596851230833,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.80814973484444,
              'lng': 144.97596851230833}],
            'distance': 4478,
            'postalCode': '3065',
            'cc': 'AU',
            'city': 'Fitzroy',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['77 Victoria Parade',
             'Fitzroy VIC 3065',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4da4ddd1a86ed93def2bce93-9'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b6b4308f964a520befc2be3',
           'name': 'The Royal Victorian Eye and Ear Hospital',
           'location': {'address': '32 Gisborne St',
            'crossStreet': 'Cnr Victoria St',
            'lat': -37.809154664653875,
            'lng': 144.9759881977738,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.809154664653875,
              'lng': 144.9759881977738}],
            'distance': 4388,
            'postalCode': '3002',
            'cc': 'AU',
            'city': 'East Melbourne',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['32 Gisborne St (Cnr Victoria St)',
             'East Melbourne VIC 3002',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b6b4308f964a520befc2be3-10'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4c47ce346c37952190d9d4b5',
           'name': 'The Alfred Centre',
           'location': {'address': '99 Commercial Rd',
            'crossStreet': 'at Punt Rd',
            'lat': -37.846212820777666,
            'lng': 144.98425217172294,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.846212820777666,
              'lng': 144.98425217172294}],
            'distance': 3374,
            'postalCode': '3182',
            'cc': 'AU',
            'city': 'Prahran',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['99 Commercial Rd (at Punt Rd)',
             'Prahran VIC 3182',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d1b3941735',
             'name': 'Medical School',
             'pluralName': 'Medical Schools',
             'shortName': 'Medical School',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4c47ce346c37952190d9d4b5-11'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b97733ef964a5202f0435e3',
           'name': 'Frances Perry House',
           'location': {'address': '20 Flemington Rd',
            'crossStreet': 'at Grattan St',
            'lat': -37.79889858351752,
            'lng': 144.95506946477255,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.79889858351752,
              'lng': 144.95506946477255}],
            'distance': 4740,
            'postalCode': '3052',
            'cc': 'AU',
            'city': 'Parkville',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['20 Flemington Rd (at Grattan St)',
             'Parkville VIC 3052',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b97733ef964a5202f0435e3-12'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b569bdef964a5209d1628e3',
           'name': "St Vincent's Hospital",
           'location': {'address': '41 Victoria Pde.',
            'lat': -37.80704181239151,
            'lng': 144.97519969940186,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.80704181239151,
              'lng': 144.97519969940186}],
            'distance': 4541,
            'postalCode': '3065',
            'cc': 'AU',
            'city': 'Fitzroy',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['41 Victoria Pde.',
             'Fitzroy VIC 3065',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b569bdef964a5209d1628e3-13'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '5bcd2c75d48ec1002c969824',
           'name': 'MyClinic Bourke Street Mall',
           'location': {'address': '283/297 Bourke Street Mall',
            'lat': -37.814008,
            'lng': 144.965194,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.814008,
              'lng': 144.965194}],
            'distance': 3420,
            'postalCode': '3000',
            'cc': 'AU',
            'city': 'Melbourne',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['283/297 Bourke Street Mall',
             'Melbourne VIC 3000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '554056774'}},
          'referralId': 'e-0-5bcd2c75d48ec1002c969824-14'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4cc674751e596dcb98a2de67',
           'name': 'Victoria Parade Surgery Centre',
           'location': {'address': '100 Victoria Parade',
            'lat': -37.80849,
            'lng': 144.975,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.80849,
              'lng': 144.975}],
            'distance': 4398,
            'postalCode': '3002',
            'cc': 'AU',
            'city': 'East Melbourne',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['100 Victoria Parade',
             'East Melbourne VIC 3002',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4cc674751e596dcb98a2de67-15'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4babffcbf964a520e7dc3ae3',
           'name': 'Epworth Freemasons Hospital',
           'location': {'address': '166 Clarendon St',
            'lat': -37.810827172265505,
            'lng': 144.98388850157215,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.810827172265505,
              'lng': 144.98388850157215}],
            'distance': 4697,
            'postalCode': '3002',
            'cc': 'AU',
            'city': 'East Melbourne',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['166 Clarendon St',
             'East Melbourne VIC 3002',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4babffcbf964a520e7dc3ae3-16'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '5d4325394478ea0008369822',
           'name': 'MyClinic South Yarra',
           'location': {'address': '300 Toorak Road',
            'lat': -37.83992921,
            'lng': 144.99703551,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.83992921,
              'lng': 144.99703551}],
            'distance': 4447,
            'postalCode': '3141',
            'cc': 'AU',
            'city': 'South Yarra',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['300 Toorak Road',
             'South Yarra VIC 3141',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-5d4325394478ea0008369822-17'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4c314e66452620a1857e200f',
           'name': 'St Vincents - Healy Wing',
           'location': {'address': '41 Victoria Parade',
            'lat': -37.8077,
            'lng': 144.97456,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.8077,
              'lng': 144.97456}],
            'distance': 4449,
            'postalCode': '3065',
            'cc': 'AU',
            'city': 'Fitzroy',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['41 Victoria Parade',
             'Fitzroy VIC 3065',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4c314e66452620a1857e200f-18'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '50a99eb1e4b03f2b099f057c',
           'name': "St Vincent's Daly Wing",
           'location': {'address': '35 Victoria Parade',
            'lat': -37.8075206306119,
            'lng': 144.97467414446768,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.8075206306119,
              'lng': 144.97467414446768}],
            'distance': 4471,
            'cc': 'AU',
            'city': 'Fitzroy',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['35 Victoria Parade',
             'Fitzroy VIC',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-50a99eb1e4b03f2b099f057c-19'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b746ed5f964a5204ddc2de3',
           'name': 'Lansdowne Eye Clinic',
           'location': {'address': '182 Victoria Parade',
            'lat': -37.8089976,
            'lng': 144.9779111,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.8089976,
              'lng': 144.9779111}],
            'distance': 4504,
            'postalCode': '3002',
            'cc': 'AU',
            'city': 'East Melbourne',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['182 Victoria Parade',
             'East Melbourne VIC 3002',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '535515924'}},
          'referralId': 'e-0-4b746ed5f964a5204ddc2de3-20'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b1dfe91f964a520ae1624e3',
           'name': "Royal Women's Hospital",
           'location': {'address': '20 Flemington Rd.',
            'crossStreet': 'at Grattan St.',
            'lat': -37.798775039876034,
            'lng': 144.95460040742643,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.798775039876034,
              'lng': 144.95460040742643}],
            'distance': 4747,
            'postalCode': '3052',
            'cc': 'AU',
            'city': 'Parkville',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['20 Flemington Rd. (at Grattan St.)',
             'Parkville VIC 3052',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b1dfe91f964a520ae1624e3-21'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4ba25a87f964a52062f037e3',
           'name': 'Cliveden Hill Private Hospital',
           'location': {'address': '29 Simpson Street',
            'lat': -37.81567015751566,
            'lng': 144.98749797429542,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.81567015751566,
              'lng': 144.98749797429542}],
            'distance': 4575,
            'cc': 'AU',
            'city': 'East Melbourne',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['29 Simpson Street',
             'East Melbourne VIC',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4ba25a87f964a52062f037e3-22'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4bbbe396e4529521055d55a4',
           'name': "Mercy / St Vincent's Private Hospital",
           'location': {'address': '159 Grey Street',
            'lat': -37.81182136358053,
            'lng': 144.98416665512147,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.81182136358053,
              'lng': 144.98416665512147}],
            'distance': 4636,
            'postalCode': '3002',
            'cc': 'AU',
            'city': 'East Melbourne',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['159 Grey Street',
             'East Melbourne VIC 3002',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4bbbe396e4529521055d55a4-23'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '57b6971c498e475f601070cb',
           'name': 'Peter McCallum Cancer Centre',
           'location': {'address': '305 Grattan Street',
            'lat': -37.799459,
            'lng': 144.956889,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.799459,
              'lng': 144.956889}],
            'distance': 4707,
            'postalCode': '3000',
            'cc': 'AU',
            'city': 'Melbourne',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['305 Grattan Street',
             'Melbourne VIC 3000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-57b6971c498e475f601070cb-24'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '50bfefc4e4b041f0f238d8aa',
           'name': 'Womens Ultrasound Melbourne (WUME)',
           'location': {'address': 'Epworth Freemasons Hospital',
            'lat': -37.80920821878404,
            'lng': 144.9821026497644,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.80920821878404,
              'lng': 144.9821026497644}],
            'distance': 4722,
            'cc': 'AU',
            'city': 'East Melbourne',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['Epworth Freemasons Hospital',
             'East Melbourne VIC',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-50bfefc4e4b041f0f238d8aa-25'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b620dc6f964a520c2322ae3',
           'name': 'Epworth Freemasons Hospital',
           'location': {'address': '320 Victoria Pde',
            'lat': -37.80917841132713,
            'lng': 144.9820976226919,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.80917841132713,
              'lng': 144.9820976226919}],
            'distance': 4724,
            'postalCode': '3002',
            'cc': 'AU',
            'city': 'East Melbourne',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['320 Victoria Pde',
             'East Melbourne VIC 3002',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b620dc6f964a520c2322ae3-26'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4b77e0e1f964a52055ad2ee3',
           'name': 'The Avenue Hospital',
           'location': {'address': '31-33 The Avenue',
            'crossStreet': 'High Street',
            'lat': -37.854841384762636,
            'lng': 144.998441229062,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.854841384762636,
              'lng': 144.998441229062}],
            'distance': 4824,
            'postalCode': '3181',
            'cc': 'AU',
            'city': 'Windsor',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['31-33 The Avenue (High Street)',
             'Windsor VIC 3181',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4b77e0e1f964a52055ad2ee3-27'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4bbd15f9a0a0c9b6573a1b0f',
           'name': 'The Royal Dental Hospital of Melbourne',
           'location': {'address': '720 Swanston Street',
            'lat': -37.79944206972209,
            'lng': 144.9642236581011,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.79944206972209,
              'lng': 144.9642236581011}],
            'distance': 4876,
            'postalCode': '3053',
            'cc': 'AU',
            'city': 'Carlton',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['720 Swanston Street',
             'Carlton VIC 3053',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '33374320'}},
          'referralId': 'e-0-4bbd15f9a0a0c9b6573a1b0f-28'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4ecef1575c5c9528f78bb7d2',
           'name': "St Vincent's Private Radiology (CMMI)",
           'location': {'address': 'Basement Level',
            'crossStreet': '55 Victoria Parade',
            'lat': -37.8080204028495,
            'lng': 144.97580943779127,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.8080204028495,
              'lng': 144.97580943779127}],
            'distance': 4481,
            'postalCode': '3065',
            'cc': 'AU',
            'city': 'Fitzroy',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['Basement Level (55 Victoria Parade)',
             'Fitzroy VIC 3065',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4ecef1575c5c9528f78bb7d2-29'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '52e5da93498e41df760de8ec',
           'name': "St Vincent's Hospital - Emergency Department",
           'location': {'address': 'Princes Street',
            'crossStreet': 'Nicholson Street',
            'lat': -37.80677903622833,
            'lng': 144.97497975826263,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.80677903622833,
              'lng': 144.97497975826263}],
            'distance': 4554,
            'postalCode': '3065',
            'cc': 'AU',
            'city': 'Fitzroy',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['Princes Street (Nicholson Street)',
             'Fitzroy VIC 3065',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-52e5da93498e41df760de8ec-30'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4bb19416f964a5205f9a3ce3',
           'name': 'Melbourne Private Hospital',
           'location': {'address': '1F Royal Parade',
            'lat': -37.7983026,
            'lng': 144.9572802,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.7983026,
              'lng': 144.9572802}],
            'distance': 4840,
            'postalCode': '3052',
            'cc': 'AU',
            'city': 'Parkville',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['1F Royal Parade',
             'Parkville VIC 3052',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []},
           'venuePage': {'id': '555485179'}},
          'referralId': 'e-0-4bb19416f964a5205f9a3ce3-31'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4e13f06fa809548274a7c2f7',
           'name': 'Melbourne Dental School',
           'location': {'address': '720 Swanston Street',
            'lat': -37.79960896638693,
            'lng': 144.96414941357807,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.79960896638693,
              'lng': 144.96414941357807}],
            'distance': 4856,
            'postalCode': '3053',
            'cc': 'AU',
            'city': 'Carlton',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['720 Swanston Street',
             'Carlton VIC 3053',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d196941735',
             'name': 'Hospital',
             'pluralName': 'Hospitals',
             'shortName': 'Hospital',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/medical_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4e13f06fa809548274a7c2f7-32'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '4d75899d08b42d43a2495c50',
           'name': '370 St Kilda Rd',
           'location': {'address': '370 St Kilda Rd',
            'lat': -37.83181,
            'lng': 144.97111,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.83181,
              'lng': 144.97111}],
            'distance': 2393,
            'postalCode': '3004',
            'cc': 'AU',
            'city': 'Melbourne',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['370 St Kilda Rd',
             'Melbourne VIC 3004',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d124941735',
             'name': 'Office',
             'pluralName': 'Offices',
             'shortName': 'Office',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/default_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-4d75899d08b42d43a2495c50-33'},
         {'reasons': {'count': 0,
           'items': [{'summary': 'This spot is popular',
             'type': 'general',
             'reasonName': 'globalInteractionReason'}]},
          'venue': {'id': '523660fd11d2a87029882809',
           'name': 'Bupa Head Office',
           'location': {'address': '33 Exhibition St.',
            'crossStreet': 'Flinders Lane',
            'lat': -37.81545563590194,
            'lng': 144.9710040834635,
            'labeledLatLngs': [{'label': 'display',
              'lat': -37.81545563590194,
              'lng': 144.9710040834635}],
            'distance': 3564,
            'postalCode': '3000',
            'cc': 'AU',
            'city': 'Melbourne',
            'state': 'VIC',
            'country': 'Australia',
            'formattedAddress': ['33 Exhibition St. (Flinders Lane)',
             'Melbourne VIC 3000',
             'Australia']},
           'categories': [{'id': '4bf58dd8d48988d124941735',
             'name': 'Office',
             'pluralName': 'Offices',
             'shortName': 'Office',
             'icon': {'prefix': 'https://ss3.4sqi.net/img/categories_v2/building/default_',
              'suffix': '.png'},
             'primary': True}],
           'photos': {'count': 0, 'groups': []}},
          'referralId': 'e-0-523660fd11d2a87029882809-34'}]}]}}




```python
def get_category_type(row):
    try:
        categories_list7 = row['categories']
    except:
        categories_list7 = row['venue.categories']
    if len(categories_list7) == 0:
        return None
    else:
        return categories_list7[0]['name']
```


```python
venues7 = results7['response']['groups'][0]['items']
nearby_venues7 = json_normalize(venues7)
filtered_columns = ['venue.name', 'venue.categories', 'venue.location.latitude', 'venue.location.longitude']
nearby_venues7 = nearby_venues7.loc[:, filtered_columns]
nearby_venues7['venue.categories'] = nearby_venues7.apply(get_category_type, axis = 1)
nearby_venues7.columns = [col.split(".")[-1] for col in nearby_venues7.columns]
print("The size of the dataframe is:", nearby_venues7.shape)
nearby_venues7
```

    The size of the dataframe is: (35, 4)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>categories</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Peter MacCallum Cancer Centre</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Alfred Hospital Outpatient Clinics</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>The Alfred Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>MyClinic Southbank</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Centre For Clinical Studies</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>5</th>
      <td>MyClinic St Kilda</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>6</th>
      <td>The Royal Melbourne Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>7</th>
      <td>MyClinic QV</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Epworth Richmond</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9</th>
      <td>St. Vincents &amp; Mercy Private</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>10</th>
      <td>The Royal Victorian Eye and Ear Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>11</th>
      <td>The Alfred Centre</td>
      <td>Medical School</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Frances Perry House</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>13</th>
      <td>St Vincent's Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>14</th>
      <td>MyClinic Bourke Street Mall</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Victoria Parade Surgery Centre</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Epworth Freemasons Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>17</th>
      <td>MyClinic South Yarra</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>18</th>
      <td>St Vincents - Healy Wing</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>19</th>
      <td>St Vincent's Daly Wing</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Lansdowne Eye Clinic</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Royal Women's Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Cliveden Hill Private Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Mercy / St Vincent's Private Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Peter McCallum Cancer Centre</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Womens Ultrasound Melbourne (WUME)</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Epworth Freemasons Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>27</th>
      <td>The Avenue Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>28</th>
      <td>The Royal Dental Hospital of Melbourne</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>29</th>
      <td>St Vincent's Private Radiology (CMMI)</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>30</th>
      <td>St Vincent's Hospital - Emergency Department</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Melbourne Private Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Melbourne Dental School</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>33</th>
      <td>370 St Kilda Rd</td>
      <td>Office</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Bupa Head Office</td>
      <td>Office</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Northern Territory Hospital Data

darwin_data = city_data[city_data['State'].str.contains('NT')].reset_index(drop = True)
print("The shape of the dataframe:", darwin_data.shape)
darwin_data[0:10]
```

    The shape of the dataframe: (1, 4)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>State</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Darwin</td>
      <td>-12.462827</td>
      <td>130.841782</td>
      <td>NT</td>
    </tr>
  </tbody>
</table>
</div>




```python
address = 'Darwin'
geolocator = Nominatim(user_agent = 'my_application')
location = geolocator.geocode(address)
latitude = location.latitude
longitude = location.longitude
print("The geographical coordinates of Darwin are {},{}.".format(latitude, longitude))
```

    The geographical coordinates of Darwin are -12.46044,130.8410469.
    


```python
city_latitude = darwin_data.loc[0, "Latitude"]
city_longitude = darwin_data.loc[0, "Longitude"]
city_name = darwin_data.loc[0, "City"]
print("The Latitude and Longitude of {} are {}, {}.".format(city_name, city_latitude, city_longitude)) 
```

    The Latitude and Longitude of Darwin are -12.462827, 130.841782.
    


```python
radius = 15000
limit = 100
query = "hospitals"
url8 = 'https://api.foursquare.com/v2/venues/explore?&client_id={}&client_secret={}&ll={},{}&v={}&radius={}&limit={}&query={}'.format(CLIENT_ID, CLIENT_SECRET, city_latitude, city_longitude, VERSION, radius, limit, query)
url8
```




    'https://api.foursquare.com/v2/venues/explore?&client_id=L351DHR2FSTSDETU0K5X3IZMC5XCTH5GJMOTYKAE0BEVPVO3&client_secret=U0NV33KA4DBR4L5ZDYKM2XXXOO4RFH5DI1XNRFUGJDIF2MVR&ll=-12.462827,130.841782&v=20180605&radius=15000&limit=100&query=hospitals'




```python
results8 = requests.get(url8).json()
```


```python
def get_category_type(row):
    try:
        categories_list8 = row['categories']
    except:
        categories_list8 = row['venue.categories']
    if len(categories_list8) == 0:
        return None
    else:
        return categories_list8[0]['name']
```


```python
venues8 = results8['response']['groups'][0]['items']
nearby_venues8 = json_normalize(venues8)
filtered_columns = ['venue.name', 'venue.categories', 'venue.location.latitude', 'venue.location.longitude']
nearby_venues8 = nearby_venues8.loc[:, filtered_columns]
nearby_venues8['venue.categories'] = nearby_venues8.apply(get_category_type, axis = 1)
nearby_venues8.columns = [col.split(".")[-1] for col in nearby_venues8.columns]
print("The size of the dataframe is:", nearby_venues8.shape)
nearby_venues8
```

    The size of the dataframe is: (4, 4)
    




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>name</th>
      <th>categories</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Integrated Health Solutions – Margaret Rolling...</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>MB Naturopath</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Royal Darwin Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Darwin Private Hospital</td>
      <td>Hospital</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>



# The use of DBSCAN for clustering cities


```python
from sklearn.cluster import DBSCAN
from geopy.distance import great_circle
from sklearn import metrics
from sklearn.datasets.samples_generator import make_blobs
from sklearn.preprocessing import StandardScaler
coords = city_datagp.as_matrix(columns = ['Latitude', 'Longitude'])
```

    C:\Users\c3273214\AppData\Local\Continuum\anaconda3\lib\site-packages\ipykernel_launcher.py:6: FutureWarning:
    
    Method .as_matrix will be removed in a future version. Use .values instead.
    
    


```python
kms_per_radian = 6371.0088
epsilon = 1.5/kms_per_radian
db = DBSCAN(eps = epsilon, min_samples = 1, algorithm = 'ball_tree', metric = 'haversine').fit(np.radians(coords))
cluster_labels = db.labels_
num_clusters = len(set(cluster_labels))
clusters = pd.Series([coords[cluster_labels == n] for n in range(num_clusters)])
print('Number of clusters: {}'.format(num_clusters))
```

    Number of clusters: 28
    

# Data for cities plus other urban areas


```python
# Reading the csv file for the other cities and other urban areas

import pandas as pd
import io
file = open('C:/Users/c3273214/Documents/Auspop2.csv', encoding = 'latin-1')
X = pd.read_csv(file)
X.head()                                                            
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Rank</th>
      <th>Area</th>
      <th>Territory</th>
      <th>June 2018[2]</th>
      <th>2011_cens</th>
      <th>Unnamed: 5</th>
      <th>Unnamed: 6</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1</td>
      <td>Sydney</td>
      <td>New South Wales</td>
      <td>5,230,330</td>
      <td>4,391,674</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2</td>
      <td>Melbourne</td>
      <td>Victoria</td>
      <td>4,936,349</td>
      <td>3,999,982</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3</td>
      <td>Brisbane</td>
      <td>Queensland</td>
      <td>2,462,637</td>
      <td>2,065,996</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4</td>
      <td>Perth</td>
      <td>Western Australia</td>
      <td>2,059,484</td>
      <td>1,728,867</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5</td>
      <td>Adelaide</td>
      <td>South Australia</td>
      <td>1,345,777</td>
      <td>1,262,940</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Grouped data for cities and other urban areas

Y = X.groupby(['Territory', 'Area'], as_index = False).agg(lambda x: ",".join(x))
Y.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Territory</th>
      <th>Area</th>
      <th>2011_cens</th>
      <th>June 2018[2]</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Australian Capital Territory/New South Wales</td>
      <td>Canberra_Queanbeyan</td>
      <td>391,645</td>
      <td>457,563</td>
    </tr>
    <tr>
      <th>1</th>
      <td>New South Wales</td>
      <td>Armidale</td>
      <td>22,464</td>
      <td>24,504</td>
    </tr>
    <tr>
      <th>2</th>
      <td>New South Wales</td>
      <td>Ballina</td>
      <td>23,509</td>
      <td>26,381</td>
    </tr>
    <tr>
      <th>3</th>
      <td>New South Wales</td>
      <td>Batemans Bay</td>
      <td>15,733</td>
      <td>16,485</td>
    </tr>
    <tr>
      <th>4</th>
      <td>New South Wales</td>
      <td>Bathurst</td>
      <td>32,479</td>
      <td>36,801</td>
    </tr>
  </tbody>
</table>
</div>




```python
Y.describe().all
```




    <bound method DataFrame.all of               Territory      Area 2011_cens June 2018[2]
    count                96        96        96           96
    unique               11        96        96           95
    top     New South Wales  Portland    14,043       26,381
    freq                 33         1         1            2>




```python
Y['June 2018[2]'].describe()
```




    count         96
    unique        95
    top       26,381
    freq           2
    Name: June 2018[2], dtype: object




```python
Y.dtypes
```




    2011_cens       object
    June 2018[2]    object
    dtype: object




```python

```
