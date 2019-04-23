
# WeatherPy
----

### Analysis
* Weather becomes significantly warmer as one approaches the equator. There appears more of these hot cities in the northern hemisphere, this may be due to the tilt of the earth and climate derived from constant and direct sunlight.
* There is no strong relationship between latitude and cloudiness. However, it is interesting to see that a strong band of cities sits at 0, 80, and 100% cloudiness.
* There is no strong relationship between latitude and wind speed. Between 0 and 50 Latitude there is a cluster of cities with lower wind speed, this could have something to do with distance from the ocean or some other factor as cities are founded where people need to live.
* Humidity does not seem to correlate strongly with distance to equator, it may have something to do with distance to the ocean, or other geographic factors.


```python
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
import requests
import time
api_key = "your key here"
from citipy import citipy
output_data_file = "output_data/cities.csv"
lat_range = (-90, 90)
lng_range = (-180, 180)
#dont forget to : pip install citipy
#mac users may need to: pip install pyopenssl ndg-httpsclient pyasn1
#otherwise api calls may refuse connection at some point
```

## Generate Cities List


```python
lat_lngs = []
cities = []
lats = np.random.uniform(low = -90.000, high = 90.000, size = 1500)
lngs = np.random.uniform(low = -180.000, high = 180.000, size = 1500)
lat_lngs = zip(lats, lngs)
for lat_lng in lat_lngs:
    city = citipy.nearest_city(lat_lng[0], lat_lng[1]).city_name
    if city not in cities:
        cities.append(city)
len(cities)
```




    619



### Perform API Calls
* Perform a weather check on each city using a series of successive API calls.
* Include a print log of each city as it'sbeing processed (with the city number and city name).



```python
i = 1
citylen = len(cities)
city_name = []
cloudiness = []
country = []
date = []
humidity = []
lat = []
lng = []
max_temp = []
wind_speed = []
for city in cities:
    if i % 60 == 0:
        print("api mandated rest (key will get banned otherwise)")
        time.sleep(60)
    print(f"record {i} of {citylen}, city: {city}")
    target_url = ('http://api.openweathermap.org/data/2.5/weather?q='
                  '{0}&appid={1}').format(city, api_key)
    weather_data = requests.get(target_url).json()  
    try:
        city_name.append(weather_data['name'])
        cloudiness.append(weather_data['clouds']['all'])
        country.append(weather_data['sys']['country'])
        date.append(weather_data['dt'])
        humidity.append(weather_data['main']['humidity'])
        lat.append(weather_data["coord"]["lon"]) 
        lng.append(weather_data['coord']['lon'])
        max_temp.append(weather_data['main']['temp_max'])
        wind_speed.append(weather_data['wind']['speed'])
    except:
        try:
            print(f"record {i} no city with that name, next record.")
        except:
            print(f"record {i} no city with that name, next record.")
    i = i + 1
    # if you don't want to wait 60 seconds you can use the below
    #time.sleep(1)
```

### Convert Raw Data to DataFrame
* Export the city data into a .csv.
* Display the DataFrame


```python
# create a data frame from cities, lat, and temp
weather_dict = {
    "City": city_name,
    "Cloudiness": cloudiness,
    "Country": country,
    "Date": date,
    "Humidity": humidity,
    "Lat": lat,
    "Lng": lng,
    "Max Temp": max_temp,
    "Wind Speed": wind_speed,
    }
weather_data = pd.DataFrame(weather_dict)
weather_data.to_csv("weather.csv", encoding='utf-8')
weather_data.count()
```


```python
weather_data.head()
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
      <th>Cloudiness</th>
      <th>Country</th>
      <th>Date</th>
      <th>Humidity</th>
      <th>Lat</th>
      <th>Lng</th>
      <th>Max Temp</th>
      <th>Wind Speed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Ilulissat</td>
      <td>75</td>
      <td>GL</td>
      <td>1552575000</td>
      <td>65</td>
      <td>-51.10</td>
      <td>-51.10</td>
      <td>258.150</td>
      <td>3.10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Tezu</td>
      <td>88</td>
      <td>IN</td>
      <td>1552578871</td>
      <td>88</td>
      <td>96.16</td>
      <td>96.16</td>
      <td>287.796</td>
      <td>0.76</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Nome</td>
      <td>40</td>
      <td>US</td>
      <td>1552578830</td>
      <td>56</td>
      <td>-94.42</td>
      <td>-94.42</td>
      <td>296.480</td>
      <td>3.10</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Benguela</td>
      <td>8</td>
      <td>AO</td>
      <td>1552578873</td>
      <td>88</td>
      <td>13.40</td>
      <td>13.40</td>
      <td>302.396</td>
      <td>4.01</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Busselton</td>
      <td>0</td>
      <td>AU</td>
      <td>1552578745</td>
      <td>42</td>
      <td>115.35</td>
      <td>115.35</td>
      <td>290.370</td>
      <td>5.26</td>
    </tr>
  </tbody>
</table>
</div>



### Plotting the Data
* Use proper labeling of the plots using plot titles (including date of analysis) and axes labels.
* Save the plotted figures as .pngs.

#### Latitude vs. Temperature Plot


```python
x_axis = weather_data["Lat"]
max_temps = weather_data["Max Temp"]

plt.title("City Latitude vs Max tempurature")
plt.xlabel("Latitude")
plt.ylabel("Tempurature")
plt.grid(True)
plt.scatter(x_axis, max_temps, marker="o", color="blue")
plt.savefig('lat_v_temp.png')
plt.show()
```


![png](output_11_0.png)


#### Latitude vs. Humidity Plot


```python
x_axis = weather_data["Lat"]
max_temps = weather_data["Humidity"]

plt.title("City Latitude vs Max Humidity")
plt.xlabel("Latitude")
plt.ylabel("Humidity")
plt.grid(True)
plt.scatter(x_axis, max_temps, marker="o", color="blue")
plt.savefig('lat_v_humidity.png')
plt.show()
```


![png](output_13_0.png)


#### Latitude vs. Cloudiness Plot


```python
x_axis = weather_data["Lat"]
max_temps = weather_data["Cloudiness"]

plt.title("City Latitude vs Max Cloudiness")
plt.xlabel("Latitude")
plt.ylabel("Cloudiness")
plt.grid(True)
plt.scatter(x_axis, max_temps, marker="o", color="blue")
plt.savefig('lat_v_cloud.png')
plt.show()
```


![png](output_15_0.png)


#### Latitude vs. Wind Speed Plot


```python
x_axis = weather_data["Lat"]
max_temps = weather_data["Wind Speed"]

plt.title("City Latitude vs Wind Speed")
plt.xlabel("Latitude")
plt.ylabel("Wind Speed")
plt.grid(True)
plt.scatter(x_axis, max_temps, marker = "o", color="blue")
plt.savefig('lat_v_wind.png')
plt.show()
```


![png](output_17_0.png)

