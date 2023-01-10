[zarr_api_addr]: https://api.dclimate.net/

# dClimate-Zarr-API-Manual

## Quickstart

Enter a valid dClimate API auth token
``` python
import requests
import json
​
token = ... # insert your dClimate API auth token
headers = {'Authorization': token}
```

View all available zarr datasets
``` python
dataset_list = requests.get(
    'https://api.dclimate.net/apiv4/datasets', headers=headers).json()
```

Get most recent IPFS CIDs/hash by dataset
``` python
dataset_hash_dict = requests.get(
    'https://api.dclimate.net/apiv4/get_heads', headers=headers).json()
```

View metadata for a given dataset
``` python
dataset = 'prism-precip-daily'
metadata = requests.get(
    f'https://api.dclimate.net/apiv4/metadata/{dataset}', headers=headers).json()
```

Access data via python requests as json with included shapefile
``` python
# ensure zipfile is read as binary (set mode="rb")
with open("/path/to/shapefile/shape.zip", mode="rb") as f:
    files = {
     'shape_file': ("shape.zip", f, 'application/octet'),
     'json': (
        "data.json",
        json.dumps(
            {
                "spatial_agg_params": {"agg_method": "mean"},
                "polygon_params": {"epsg_crs": 4326}
            }),
        "application/json")
    }
    r1 = requests.post(
        "https://api.dclimate.net/apiv4/geo_temporal_query/prism-precip-daily?output_format=array",
        files=files,
        headers=headers)
    data_dict = r1.json()
```

Access data via python requests as netcdf (no shapefile needed)
``` python
import xarray as xr
r = requests.post(
        "https://api.dclimate.net/apiv4/geo_temporal_query/prism-precip-daily?output_format=netcdf",
        json={"circle_params": {"radius": 50, "center_lat": 43, "center_lon": -123}},
        headers=headers
    )
ds = xr.open_dataset(r.content)
```

Access data in the format of the v3 API:
``` python
r = requests.get(
    "https://api.dclimate.net/apiv4/grid-history/prism-precip-daily/40_-120",
    params={"desired_units": "mm"},
    headers=headers
)
data = r.json()["data"]
```

## Account Setup

Before you can make requests, you must get a free authorization token by [registering for an account](https://api.dclimate.net/register). Upon registering, your authorization token will be emailed to you along with a unique verification link.

When making requests via the [API documentation page](https://api.dclimate.net/apiv3), you should insert this token in the top field labeled `Authorization`. When making requests programmatically, pass this token in the request headers under the key `Authorization`.


## Structure of geospatial requests
​
This API is capable of querying N-dimensional data from dClimate. The API works by parsing the request and passing it within the *body* of a POST request to a separate dClimate client. This client packages the data on the server side according to the passed requests and returns data within a JSON.

Below is a short description of how to structure requests to the API and specify parameters correctly.

Note that as the API is in beta **it is incumbent on users to structure requests and inputs carefully** — error detection and validation is not fully built out and incorrectly projected shapefiles, badly formatted datetimes, etc. will likely fail in unpredictable ways.

### Request format

#### Header
​
The request header should be in the following format

`"https://api.dclimate.net/apiv4/geo_temporal_query/<dataset_name>?output_format=<desired_format>"`

Valid output formats are `'array', 'netcdf'`. The former returns a numpy array of values and the latter a NetCDF file.
​

#### Body
​
Requests in the body of a POST can take two forms:
* A simple query in JSON format listing the various request parameters under the key `json`.
* A two-key dict containing request parameters under `json` and zipped shapefile data under `shape_file`

In the latter case the values of each item in the dict should take the form of `(title, object, format)`. The JSON's title should be `data.json` and its format `application/json` while the shapefile should be `shape_file.zip` and `application/octet` (binary format), respectively. The object will contain the corresponding request parameters or zipped shapefile, respectively
​
#### Shapefile inputs 
​
All shapefiles uploaded must be **zipped** and in **EPSG:4326 (WGS84)** projection.
​
### Parameters
​
Queries can take a number of input parameters for some combination of spatial and temporal filters and aggregations to perform during the request. These are listed and then described below.

**Spatial parameters**
``` python
point_params : (lat: float, lon: float)
circle_params : (center_lat: float, center_lon: float, radius: float) # radius in KM
rectangle_params : (min_lat: float, min_lon: float, max_lat: float, max_lon: float)
polygon_params : (epsg_crs: int)
multiple_points_params : (epsg_crs: int)
spatial_agg_params : (agg_method: str)
```
​
**Temporal parameters**
```python
time_range : [start_time: str, end_time: str] # in ISO Format structured as list with two elements
temporal_agg_params : (time_period: str, agg_method: str, time_unit: int)
rolling_agg_params : (window_size: int, agg_method: str)
```

All spatial units are in degrees of latitude/longitude unless otherwise specified.

Valid aggregation methods for all `agg` requests are `'min'`, `'max'`, `'median'`, `'mean'`, `'std'`, `'sum'`. Only one may be specified in a given set of parameters.

Valid time periods for `temporal_agg_params` are `'hour'`, `'day'`, `'week'`, `'month'`, `'quarter'`, `'year'`, `'all'`. Only one may be specified in a given set of parameters.

The `epsg_crs` for `polygon_params` and `multiple_points_params` refers to the Coordinate Reference System (projection) of the dataset. It defaults to 4326 (WGS84), or recommended projection. We cannot guarantee performance with shapefiles uploaded in other projections, particularly projected (metric) projections).

The `time_unit` for `temporal_agg_params` refers to the number of time periods to aggregate by. It defaults to 1. 

The `window_size` for `rolling_agg_params` refers to the number of units of time used to construct the rolling aggregation. The specific unit depends on the dataset.
​
### Validation and errors
​
Both the API and client perform basic validation checks on the provided requests and will reject invalid requests. 
​

Types of invalid requests include:
* Specifying too many points or too large areas
* Specifying multiple geographic filters (e.g. polygon AND circle)
* Specifying temporal _and_ rolling time-based aggregations
* Providing shapefiles with invalid geometries or projections
* Selecting an area/time period with all null data
* Requesting invalid output formats (or just misspelling them)
​
## Structure of single point requests
This API is capable of querying gridded datasets from dClimate using the same structure as `apiv3/` requests. The `apiv4/grid-history/` endpoint will attempt to query a gridded zarr dataset. If the dataset cannot be found as a zarr, the API will attempt to find a non-zarr version.

Note: Datasets that cannot be found as a zarr will be much slower to retrieve.

#### Path arguments

``` python
dataset_name : str
latitude : float
longitude : float
```
​
The request should be a GET in the following format

`"https://api.dclimate.net/apiv4/grid-history/<dataset_name>/<latitude>_<longitude>"`

#### Query arguments

``` python
use_imperial_units : bool # default true
desired_units : str # default none
as_of : str # default none, iso 8601 formatted date
```

**Note**: The `as_of` parameter will show you a dataset's history as of a certain date in the past. If this is a zarr dataset but the as_of date is before the zarr dataset's creation, the API will attempt to look for this dataset's history as a non-zarr. The `apiv4/grid-history/` endpoint will return an error if `as_of` is before the creation of both the zarr and non-zarr version of the dataset.
