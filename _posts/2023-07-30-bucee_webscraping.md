---
layout: post
title: "Cataloging every Buc'ees Location"
subtitle: "Using Python to webscrape and organize a list of all 46 Buc'ees locations"
---
### Loading in the required packages


```python
from bs4 import BeautifulSoup as bs
import requests
import re
import pandas as pd
from geopy.geocoders import ArcGIS
import os
```

### Using the requests and beautifulsoup packages to load in the url and convert the webpage to 'soup'


```python
url = 'https://buc-ees.com/locations/'
page = requests.get(url)
soup = bs(page.text, 'html')
```

### Searching for address and location info by using specific html tags from the webpage


```python
locations = soup.find_all('div', class_='bucees-location-address')
cities = soup.find_all('h4')
```

### Looping through specified div and class tags on the webpage to extract and clean location data


```python
cities_list = [city.text.strip() for city in cities]
#print(cities_list)

address = [location.text for location in locations]
address_clean = []
for x in address:
    street_address = re.sub('(Ethanol-Free:\d+\s\w+\s\d+\sOctane|Ethanol-Free:\d+\sOctane)', '', x)
    address_clean.append(street_address)
```

### Further cleaning the data and stripping whitespace from the entries to isolate city and state values for each location


```python
address_clean

city_state_df = pd.DataFrame(cities_list)
city_state_df.reset_index(inplace=True)
city_state_df.rename( columns={0:'raw', 'index':'#'}, inplace=True )
city_state_df['state'] = city_state_df['raw'].str.extract(r'(\w+$)')
city_state_df['city'] = city_state_df['raw'].str.extract(r'([A-Za-z]+,|[A-Za-z]+\s\w+,)')
city_state_df['city'] = city_state_df['city'].str.replace(',', '')
#city_state_df 
city_state = city_state_df[['#', 'city', 'state']]
city_state.head().style    
```




<style type="text/css">
</style>
<table id="T_3c1ef">
  <thead>
    <tr>
      <th class="blank level0" >&nbsp;</th>
      <th id="T_3c1ef_level0_col0" class="col_heading level0 col0" >#</th>
      <th id="T_3c1ef_level0_col1" class="col_heading level0 col1" >city</th>
      <th id="T_3c1ef_level0_col2" class="col_heading level0 col2" >state</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th id="T_3c1ef_level0_row0" class="row_heading level0 row0" >0</th>
      <td id="T_3c1ef_row0_col0" class="data row0 col0" >0</td>
      <td id="T_3c1ef_row0_col1" class="data row0 col1" >Athens</td>
      <td id="T_3c1ef_row0_col2" class="data row0 col2" >AL</td>
    </tr>
    <tr>
      <th id="T_3c1ef_level0_row1" class="row_heading level0 row1" >1</th>
      <td id="T_3c1ef_row1_col0" class="data row1 col0" >1</td>
      <td id="T_3c1ef_row1_col1" class="data row1 col1" >Auburn</td>
      <td id="T_3c1ef_row1_col2" class="data row1 col2" >AL</td>
    </tr>
    <tr>
      <th id="T_3c1ef_level0_row2" class="row_heading level0 row2" >2</th>
      <td id="T_3c1ef_row2_col0" class="data row2 col0" >2</td>
      <td id="T_3c1ef_row2_col1" class="data row2 col1" >Leeds</td>
      <td id="T_3c1ef_row2_col2" class="data row2 col2" >AL</td>
    </tr>
    <tr>
      <th id="T_3c1ef_level0_row3" class="row_heading level0 row3" >3</th>
      <td id="T_3c1ef_row3_col0" class="data row3 col0" >3</td>
      <td id="T_3c1ef_row3_col1" class="data row3 col1" >Loxley</td>
      <td id="T_3c1ef_row3_col2" class="data row3 col2" >AL</td>
    </tr>
    <tr>
      <th id="T_3c1ef_level0_row4" class="row_heading level0 row4" >4</th>
      <td id="T_3c1ef_row4_col0" class="data row4 col0" >4</td>
      <td id="T_3c1ef_row4_col1" class="data row4 col1" >Daytona Beach</td>
      <td id="T_3c1ef_row4_col2" class="data row4 col2" >FL</td>
    </tr>
  </tbody>
</table>




### Extracting zip codes and street numbers for each location using the regex package


```python
table = pd.DataFrame(columns = ['raw', 'city', 'state', 'zip', 'street_address'])
df = pd.DataFrame(address_clean)
#df['num'] = len(address_clean) - row
df.reset_index(inplace=True)
df.rename( columns={0:'raw','index':'#'}, inplace=True)
df['zip_code'] = df['raw'].str.extract(r'(\d+$)')
df['state'] = df['raw'].str.extract(r'(,\s\D+)')
df['state'] = df['state'].str.extract(r'(\w+\s\w+|\w+)')
df['street_num'] = df['raw'].str.extract(r'(\d+)')
df['city'] = df['raw'].str.extract(r'(\w+,|\w+\s\w+,)')

zip_df = df[['#', 'zip_code', 'street_num']]
zip_df.head()
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
      <th>#</th>
      <th>zip_code</th>
      <th>street_num</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>35613</td>
      <td>2328</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>36832</td>
      <td>2500</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>35094</td>
      <td>6900</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>36567</td>
      <td>20403</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>32117</td>
      <td>2330</td>
    </tr>
  </tbody>
</table>
</div>



### Combining street numbers with street names to form full addresses for each location


```python
address_list = []

for i in locations:
    local = re.search(r'(>(.*?)<)', str(i))
    address_list.append(local[0])
    
add_list_clean = []

for row in address_list:
    
    street = re.sub('>|<', '', str(row))
    add_list_clean.append(street)
    
add_list_clean

addy = pd.DataFrame(add_list_clean)
addy.reset_index(inplace=True)
addy.rename( columns={0:'street_address', 'index':'#'}, inplace=True )
addy.head()
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
      <th>#</th>
      <th>street_address</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>2328 Lindsay Lane South</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>2500 Buc-ee’s Blvd</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>6900 Buc-ee’s Blvd.</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>20403 County Rd. 68</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>2330 Gateway North Drive</td>
    </tr>
  </tbody>
</table>
</div>



### Merging all 3 dataframes together to create a full dataset


```python
half_bucee = addy.merge(zip_df, how='right')
full_bucee = city_state.merge(half_bucee, how='right')
full_bucee['street_name'] = full_bucee['street_address'].replace(r'(^\d+\s)','', regex=True)
full_bucee['full_addy'] = full_bucee['street_address']+', '+full_bucee['city']+', '+full_bucee['state']+' '+full_bucee['zip_code']
full_bucee[['#', 'full_addy', 'street_num', 'street_name', 'city', 'state', 'zip_code']]
full_bucee['store_code'] = full_bucee['#'].apply(str) + '_' + full_bucee['city']
full_bucee.head()
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
      <th>#</th>
      <th>city</th>
      <th>state</th>
      <th>street_address</th>
      <th>zip_code</th>
      <th>street_num</th>
      <th>street_name</th>
      <th>full_addy</th>
      <th>store_code</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>Athens</td>
      <td>AL</td>
      <td>2328 Lindsay Lane South</td>
      <td>35613</td>
      <td>2328</td>
      <td>Lindsay Lane South</td>
      <td>2328 Lindsay Lane South, Athens, AL 35613</td>
      <td>0_Athens</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>Auburn</td>
      <td>AL</td>
      <td>2500 Buc-ee’s Blvd</td>
      <td>36832</td>
      <td>2500</td>
      <td>Buc-ee’s Blvd</td>
      <td>2500 Buc-ee’s Blvd, Auburn, AL 36832</td>
      <td>1_Auburn</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>Leeds</td>
      <td>AL</td>
      <td>6900 Buc-ee’s Blvd.</td>
      <td>35094</td>
      <td>6900</td>
      <td>Buc-ee’s Blvd.</td>
      <td>6900 Buc-ee’s Blvd., Leeds, AL 35094</td>
      <td>2_Leeds</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>Loxley</td>
      <td>AL</td>
      <td>20403 County Rd. 68</td>
      <td>36567</td>
      <td>20403</td>
      <td>County Rd. 68</td>
      <td>20403 County Rd. 68, Loxley, AL 36567</td>
      <td>3_Loxley</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>Daytona Beach</td>
      <td>FL</td>
      <td>2330 Gateway North Drive</td>
      <td>32117</td>
      <td>2330</td>
      <td>Gateway North Drive</td>
      <td>2330 Gateway North Drive, Daytona Beach, FL 32117</td>
      <td>4_Daytona Beach</td>
    </tr>
  </tbody>
</table>
</div>



### Assigning the 'nom' variable to the ArcGIS function to create latitude/longitude data for each Buc'ees location


```python
nom=ArcGIS()
```

### Creating a dummy column in the dataset to hold GIS data


```python
full_bucee['coord'] = full_bucee['full_addy'].apply(nom.geocode)
```

### Splitting the 'coord' column into seperate latitude and longitude data


```python
full_bucee['lat'] = full_bucee['coord'].apply(lambda x: x.latitude if x != None else None)
full_bucee['lon'] = full_bucee['coord'].apply(lambda x: x.longitude if x != None else None)
clean_beaver = full_bucee[['city', 'state', 'store_code','full_addy', 'lat', 'lon']]
clean_beaver.head()
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
      <th>city</th>
      <th>state</th>
      <th>store_code</th>
      <th>full_addy</th>
      <th>lat</th>
      <th>lon</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Athens</td>
      <td>AL</td>
      <td>0_Athens</td>
      <td>2328 Lindsay Lane South, Athens, AL 35613</td>
      <td>34.730718</td>
      <td>-86.930749</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Auburn</td>
      <td>AL</td>
      <td>1_Auburn</td>
      <td>2500 Buc-ee’s Blvd, Auburn, AL 36832</td>
      <td>32.610800</td>
      <td>-85.494635</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Leeds</td>
      <td>AL</td>
      <td>2_Leeds</td>
      <td>6900 Buc-ee’s Blvd., Leeds, AL 35094</td>
      <td>33.543646</td>
      <td>-86.587598</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Loxley</td>
      <td>AL</td>
      <td>3_Loxley</td>
      <td>20403 County Rd. 68, Loxley, AL 36567</td>
      <td>30.634522</td>
      <td>-87.676438</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Daytona Beach</td>
      <td>FL</td>
      <td>4_Daytona Beach</td>
      <td>2330 Gateway North Drive, Daytona Beach, FL 32117</td>
      <td>29.224078</td>
      <td>-81.099183</td>
    </tr>
  </tbody>
</table>
</div>



### Exporting dataframe to csv and saving to local drive


```python
os.makedirs('/User/rieke/R FILES', exist_ok=True)
clean_beaver.to_csv('/Users/rieke/R FILES/bucees.csv')
```
