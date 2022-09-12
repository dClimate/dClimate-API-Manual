# dClimate-API-Manual

## Description

The dClimate REST API enables programmatic access to a decentralized network of climate data in a simple yet comprehensive manner.

## Account Setup

Before you can make requests, you must get a free authorization token by [registering for an account](https://api.dclimate.net/register). Upon registering, your authorization token will be emailed to you along with a unique verification link. You must click the link to verify your email address before your token becomes active. It's also recommended you save your authorization token somewhere accessible as you will need to use this token with each request. When making requests via the [API documentation page](https://api.dclimate.net), you should insert this token in the top field labeled `Authorization`. When making requests programmatically, pass this token in the request headers under the key `Authorization`.

## Intro to dClimate Data

The [API documentation page](https://api.dclimate.net) shows available endpoints categorized by type and index method. For example, `Grid File Dataset History` contains datasets spatially indexed in a format that consists of a matrix of cells, or pixels, organized into rows and columns where each cell contains a value for each grid point across a two dimensional surface. On the contrary, the `GHCN Dataset History`, `CME Station History`, and other station endpoints contain data provided by ground based station and are indexed by a station ID. Other datasets include forecast data, hurricane and tropical storm data, as well as agricultural yield data and more. A list of all available datasets can be found under the `Dataset Information` tab. 

## Pulling Data

Now that you have your authorization token you can make your first request. The easiest way to pull data is by using the [API documentation page](https://api.dclimate.net) and requires no technical skills. To begin click on a tab containing the dataset you would like to pull. The ERA5 dataset is a good example to use as it is a globally spanning gridded dataset containing over 30 years of hourly historical data across multiple climate catagories such as precipitation, temperature, and wind. <!--  We'll talk more about dataset selection below.  -->

To pull temperature data from ERA5 first we need to click on the `Grid File Dataset History` tab, click on the `GET` request endpoint and click `"Try it out"` on the top right. Next, paste your authorization token on the top field. Now you will set the parameters of our request. The first important parameter is the units you want the data to be displayed in. For temperature datasets, set the `use_imperial_units` parameter to `true` to get data in degrees fahrenheit. Alternatively you can attempt to get data in a different unit by setting the `desired_units` to an applicable unit scale such as `celsius` or `kelvin` for temperature. The next important parameter for hourly datasets in what timezone you want the data to be indexed in. For UTC time set the `convert_to_local_time` to `False`. If left untouched, this parameter will default to data indexed by the grid's local time. When pulling data programmatically these values should be passed via the request parameters.

Now set the required path parameters, dataset and location. For ERA5 temperature set the `dataset` to `era5_land_2m_temp-hourly`. In this example we will use New York, NY which has an approximate latitude of `40.7128° N` and longitude of `74.0060° W`. When passing the coordinates via the api use a positive value for north and east and a negative value for south and west. Therefore, set `lat` to `40.7128` and `lon` to `-74.0060`. Finally, click `Execute` and if your request was successful a link to download your data as a csv should appear at the bottom.

## Pulling Data Programmatically (Python 3)

Making this same request using Python 3 is simple using the built in `requests` library. You can load data into a Python dictionary using the following code:

```python
import requests as r

key = ... # dClimate API authorization token

dataset = "era5_land_2m_temp-hourly"

lat, lon = 40.7128, -74.0060

params = {
    "use_imperial_units": True,
    "convert_to_local_time": False
    }

headers = {
    "Authorization": key,
    "accept": "application/json" 
    }

response = requests.get(
    f"https://api.dclimate.net/apiv3/grid-history/{dataset}/{lat}_{lon}",
    params=params,
    headers=headers)

print(response.json())

```

<!-- ## Understanding Metadata -->

