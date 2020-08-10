---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - Python3
  - shell

search: true

code_clipboard: true
---

# Introduction

WattTime technology—based on real-time grid data, cutting-edge algorithms, and machine learning—provides first-of-its-kind insight into your local electricity grid’s marginal emissions rate. 

The WattTime API provides access to real-time, forecast, and historical marginal emissions data for electric grids around the world. For a deeper understanding of these Marginal Operating Emissions Rates (MOERs) please see [What is AER] (https://www.watttime.org/aer/what-is-aer/) and [How AER works] (https://www.watttime.org/aer/how-aer-works/).
  
You can access the API by sending standard HTTP requests to the endpoints listed below. 

The `/data`, `/historical`, and `/forecast` endpoints are only available to subscribers. ANALYST and PRO data plans can be found [here] (https://www.watttime.org/get-the-data/data-plans/).

For (Python3) example code that can be used to register, log in, and query data, please see our [example code] (https://github.com/WattTime/apiv2-example/blob/master/query_apiv2.py).

### Restrictions
There are strict limits on the number of connections and rates at which you may query the API. From any single IP address you may make a maximum of 10 connections, and may have up to 100 outstanding queries that will be rate limited to 10 requests per second. If there are more than 100 requests outstanding, or more than 10 connections, those requests may be dropped and an `HTTP 429` error code returned.


# Authentication

To start using the API, first register for an account by using the `/register` endpoint. Then use the `/login` endpoint to obtain an access token. You can then use your token to access the remainder of our endpoints. WattTime expects a token to be included in all API requests to the server. Your access token will expire after 30 minutes and you'll need to sign in again to obtain a fresh one.

## Create New User Account

> To register, use this code:

```python
import requests
register_url = 'https://api2.watttime.org/v2/register'
params = {'username': 'USERNAME',
         'password': 'my_special_password',
         'email': 'some_email@gmail.com',
         'org': 'some org name'}
rsp = requests.post(register_url, json=params)
print(rsp.text)
```

```shell
curl -H "Content-type: application/json" --data-binary '{"username":"freddo", "password":"the_frog", "email":"freddo@frog.org", "org":"freds world"}' https://api2.watttime.org/v2/register
```

> The above command returns JSON structured like this:

```json
{
  "user": "freddo",
  "ok": "User created"
}
```

### `/register`

Provide basic information to self register for an account. Higher level access is via subscription and can be requested at `support@watttime.org`.

### HTTP Request

`POST https://api2.watttime.org/v2/register`

### URL Query Parameters

Parameter | Description | Example | Type
--------- | ----------- | ------- | ----
username (required) | name of user that will be used in subsequent calls | freddo | string 
password (required) | user password | the_frog | string
email (required) | valid email address | freddo@frog.org | string
organization | organization name | freds world | string

<aside class="success">
Make sure to only register once! 
</aside>

## Obtain Access Token

> To register, use this code:

```python
import requests
from requests.auth import HTTPBasicAuth
login_url = 'https://api2.watttime.org/v2/login'
rsp = requests.get(login_url, auth=HTTPBasicAuth('USERNAME', 'PASSWORD'))
print(rsp.json())
```

```shell
curl --basic -u USERNAME:PASSWORD https://api2.watttime.org/v2/login
```

> The above command returns your personal token, in JSON structure like this:

```json
{
  "token": "abcdef0123456789fedcabc"
}

```
### `/login`

Use HTTP basic auth to exchange username and password for an access token. Tokens are used in all subsequent data calls. 

### HTTP Request

`GET https://api2.watttime.org/v2/login/`

### URL Query Parameters

Parameter | Description
--------- | -----------
none | uses basic authentication 

### Response

Attribute | Description
--------- | -----------
token | abcdef0123456789fedcabc 

<aside class="success">
Token expires after 30 minutes. If a data call returns 401, you will need to login again to receive a new token.

</aside>

## Password Reset

> To reset your password, use this code:

```python
import requests
password_url = 'https://api2.watttime.org/v2/password/?username=USERNAME'
rsp = requests.get(password_url)
print(rsp.json())
```

```shell
curl "https://api2.watttime.org/v2/password/?username=USERNAME"
```

> Sample Response:

```json
{
  "ok": "Please check your email for the password reset link"
}

```

### `/password`

Provide your username to request an email be sent to you with password reset instructions.  

### HTTP Request

`GET https://api2.watttime.org/v2/password/?username=freddo`

### URL Query Parameters

Parameter | Example | Type
--------- | ------- | ----
username | freddo | string 

### Response

Attribute | Description | Type
--------- | ----------- | ----
"ok" | Message confirming reset link has been sent | string 


# Grid Emissions Information

## Determine Grid Region

> Make sure to add in your registered USERNAME and PASSWORD to this code. You should not add in your token here. The code automatically generates a new token each time you run it.

```python
import requests
from requests.auth import HTTPBasicAuth

login_url = 'https://api2.watttime.org/v2/login'
token = requests.get(login_url, auth=HTTPBasicAuth('USERNAME', 'PASSWORD')).json()['token']

region_url = 'https://api2.watttime.org/v2/ba-from-loc'
headers = {'Authorization': 'Bearer {}'.format(token)}
params = {'latitude': '42.372', 'longitude': '-72.519'}
rsp=requests.get(region_url, headers=headers, params=params)
print(rsp.text)
```

```shell
curl -H "Authorization: Bearer abcdef0123456789fedcabc" https://api2.watttime.org/v2/ba-from-loc?latitude=42.372&longitude=-72.519
```

> Sample Response

```json
{
"id":169,
"abbrev":"ISONE_WCMA",
"name":"ISONE Western/Central Massachusetts"
}
```
### `/ba-from-loc`

This endpoint, provided a latitude and longitude parameter, returns the enclosing balancing authority details if known, or an "Invalid Coordinates" error if the point lies outside of known BAs. For more information on what a balancing authority is, see [this explanation] (https://www.eia.gov/todayinenergy/detail.php?id=27152) from the EIA.

    
See [map coverage] (https://www.watttime.org/explorer/#3/41.23/-97.64) for available grid regions.

### HTTP Request

`GET https://api2.watttime.org/v2/ba-from-loc/?latitude=42.372&longitude=-72.519`

### URL Query Parameters

Parameter | Description | Example | Type
--------- | ----------- | ------- | ----
latitude | Latitude of location | 42.372 | float 
longitude | Longitude of location | -72.519 | float

<aside class="warning">
Locations not associated with a known balancing authority will return a <code>HTTP code 400</code>.
</aside>

### Response

Parameter | Description | Example | Type
--------- | ----------- | ------- | ----
ba | Balancing authority abbreviation | NYISO | string 
id | Unique id for the region | 8 | integer
name | Human readable name/description for the region | New York ISO | string


## Realtime Emissions

```python
import requests
from requests.auth import HTTPBasicAuth

login_url = 'https://api2.watttime.org/v2/login'
token = requests.get(login_url, auth=HTTPBasicAuth('USERNAME', 'PASSWORD')).json()['token']

index_url = 'https://api2.watttime.org/index'
headers = {'Authorization': 'Bearer {}'.format(token)}
params = {'ba': 'CAISO_ZP26'}
rsp=requests.get(index_url, headers=headers, params=params)
print(rsp.text)
```

```shell
curl -H "Authorization: Bearer abcdef0123456789fedcabc" https://api2.watttime.org/v2/index?ba=CAISO_ZP26&style=all
```

> Sample Response

```json
{
    "freq": "300",   
    "ba": "MISO_MI",
    "percent": "53",
    "moer": "982.65",
    "point_time": "2019-01-29T14:55:00.00Z",
}
```
### `/index`

Provides a real-time signal indicating the carbon intensity on the local grid in that moment (typically updated every 5 minutes). The current emissions rate of the grid is returned as a raw Marginal Operating Emissions Rate (MOER) value (available only to users with PRO subscriptions), or as an index value (percent), which is available to all users. This value is returned along with a ‘freq’ value, indicating how long the value is in effect. The type of value returned is set by the 'style' query parameter.  

The *MOER* value is the lbs of CO2 per MWh produced on the grid in the specified region. The reported index *percent* is where the current MOER is between the min and max from the last two weeks. 

<aside class="success">
Please use this endpoint for all real-time emissions queries and do not use the <code>/data</code> endpoint for this purpose unless required.  
</aside>

<aside class="success">
For retrieving historical emissions rates, we recommend using raw MOER values from the <code>/data</code> or <code>/historical</code> endpoints.
</aside>

### HTTP Request

`GET https://api2.watttime.org/v2/index`

### URL Query Parameters

Parameter | Description | Example | Type
--------- | ----------- | ------- | ----
ba | Balancing authority abbreviation. Optional - provide ba OR provide latitude+longitude, not all three | CAISO_ZP26 | string 
latitude | Latitude of location | 42.372 | float 
longitude | Longitude of location | -72.519 | float
style | Units in which to provide realtime marginal emissions. **Choices are 'percent', 'moer' or 'all'**. If you have PRO access you may use 'moer' to get the latest raw MOER value. Defaults to 'all' if not provided | all | string

### Response

Parameter | Description | Example | Type
--------- | ----------- | ------- | ----
freq | Duration from point_time for when new value is expected (in seconds) | 300 | string 
ba | Balancing authority abbreviation | CAISO_ZP26 | string 
percent | An integer between 0 (minimum MOER in the last two weeks i.e. clean) and 100 (maximum MOER in the last two weeks
i.e. dirty) representing the relative realtime marginal emissions intensity. | 53 | integer 
moer | Marginal Operating Emissions Rate (MOER) measured in lbs CO2/MWh. This is only available for PRO subscriptions | 850.743 | float
point_time | ISO8601 UTC date/time format indicating when this data became valid | 2019-01-29T14:55:00.00Z | string

## Grid Data

```python
import requests
from requests.auth import HTTPBasicAuth

login_url = 'https://api2.watttime.org/v2/login'
token = requests.get(login_url, auth=HTTPBasicAuth('USERNAME', 'PASSWORD')).json()['token']

data_url = 'https://api2.watttime.org/v2/data'
headers = {'Authorization': 'Bearer {}'.format(token)}
params = {'ba': 'CAISO_ZP26', 
          'starttime': '2019-02-20T16:00:00-0800', 
          'endtime': '2019-02-20T16:15:00-0800'}
rsp = requests.get(data_url, headers=headers, params=params)
print(rsp.text)
```

```shell
curl -H "Authorization: Bearer abcdef0123456789fedcabc" https://api2.watttime.org/v2/data?ba=CAISO_NP15&starttime=2019-02-20T16:00:00-0800&endtime=2019-02-20T16:15:00-0800
```

> Sample Response

```json
[
{"point_time": "2019-02-21T00:15:00.000Z", "value": 844, "frequency": 300, "market": "RTM", "ba": "CAISO_ZP26", "datatype": "MOER", "version": "2.1.1"}, 
{"point_time": "2019-02-21T00:10:00.000Z", "value": 834, "frequency": 300, "market": "RTM", "ba": "CAISO_ZP26", "datatype": "MOER", "version": "2.1.1"}, 
{"point_time": "2019-02-21T00:05:00.000Z", "value": 834, "frequency": 300, "market": "RTM", "ba": "CAISO_ZP26", "datatype": "MOER", "version": "2.1.1"}, 
{"point_time": "2019-02-21T00:00:00.000Z", "value": 844, "frequency": 300, "market": "RTM", "ba": "CAISO_ZP26", "datatype": "MOER", "version": "2.1.1"}
]
```
### `/data`

Obtain historical marginal emissions for a given area (balancing authority abbreviated code) or location (latitude/longitude pair).  
  
Access to this endpoint is restricted to customers with ANALYST or PRO subscriptions. However, you can preview the data provided by using grid region `CAISO_ZP26` for your requests.

<aside class="success">
Individual queries are limited to 30 days of data. If you need to download multiple months or years of data, consider using the <code>/historical</code> endpoint or request multiple days of data for each query (do not query every 5-minute datapoint individually).
</aside>

<aside class="success">
Provide <code>ba</code> OR provide <code>latitude+longitude</code>, not all three. If all are supplied, only the <code>ba</code> value is used. 
</aside>

<aside class="success">
Starttime/endtime values are in <code>ISO8601</code> format or <code>RFC2822</code> format. Generally <code>ISO8601</code> formats are preferred.
</aside>

### HTTP Request

`GET https://api2.watttime.org/v2/data/?ba=CAISO_ZP26&latitude=35.0&longitude=120.0&starttime=2019-02-20T16:00:00-0800&endtime=2019-02-20T16:15:00-0800&style=all&moerversion=2.1.1`

### URL Query Parameters

Parameter | Description | Example | Type
--------- | ----------- | ------- | ----
ba | Balancing authority abbreviation | CAISO_ZP26 | string 
latitude | Latitude of location | 42.372 | float 
longitude | Longitude of location | -72.519 | float
starttime | ISO 8601 timestamp (inclusive) - may be omitted if endtime is also omitted. If both are omitted, returns the most recent data point. | 2019-02-20T16:00:00-0800 | string
endtime | ISO 8601 timestamp (inclusive) - if endtime is omitted, endtime is equal to  | 2019-02-20T16:15:00-0800 | string
style | Style of data to provide. Optional - can be all or moer. Defaults to all if omitted. | all | string
moerversion | MOERversion. Defaults to the latest version for a given region if omitted. Omit this field unless you need to request a particular version. | 2.1.1 | string


### Response

Parameter | Description | Example | Type
--------- | ----------- | ------- | ----
ba | Balancing authority abbreviation | CAISO_ZP26 | string 
datatype | Type of data | MOER | string 
frequency | Duration in seconds for which the data is valid from point_time | 300 | integer 
market | Market type, only useful for grid data | RTM | string
point_time | ISO8601 UTC date/time format indicating when this data became valid | 2019-01-29T14:55:00.00Z | string
value | Number value of data (corresponding to datatype above) | 909.06 | float 
version | MOER version (Not present and not applicable for other datatypes) | 2.0 | string 

## Historical Emissions

```python
from os import path
import requests
from requests.auth import HTTPBasicAuth

login_url = 'https://api2.watttime.org/v2/login'
token = requests.get(login_url, auth=HTTPBasicAuth('USERNAME', 'PASSWORD')).json()['token']

historical_url = 'https://api2.watttime.org/v2/historical'
headers = {'Authorization': 'Bearer {}'.format(token)}
ba = 'CAISO_ZP26'
params = {'ba': ba}
rsp = requests.get(historical_url, headers=headers, params=params)
cur_dir = path.dirname(path.realpath('__file__'))
file_path = path.join(cur_dir, '{}_historical.zip'.format(ba))
with open(file_path, 'wb') as fp:
    fp.write(rsp.content)

print('Wrote historical data for {} to {}'.format(ba, file_path))
```

```shell
curl -H "Authorization: Bearer abcdef0123456789fedcabc" https://api2.watttime.org/v2/historical?ba=CAISO_NP15&version=latest
```

> Sample Response

```json
Content-Type:application/zip
```
### `/historical`

Obtain a zip file containing the MOER values and timestamps for a given region for (up to) the past two years.  

WattTime periodically updates the algorithms used to calculate new MOERs to add new features, incorporate new data sources, or in response to changing grid dynamics. This endpoint provides you with the option to retrieve all the MOER versions published for a given region or to retrieve only the most up-to-date version.  

Historical data will be updated on the 2nd of each month at midnight UTC for the previous month. 
  
Access to this endpoint is restricted to customers with ANALYST or PRO subscriptions.  

<aside class="success">
Please note that this endpoint will return binary data (representing a zip file) which needs to be saved to disk and then unzipped (or unzipped programmatically) in order to obtain human readable csv files containing the MOER values.
</aside>

### HTTP Request

`GET https://api2.watttime.org/v2/historical`

### URL Query Parameters

Parameter | Description | Example | Type
--------- | ----------- | ------- | ----
ba | Balancing authority abbreviation | MISO_MI | string 
version | Optional: MOER version to retrieve. Options are latest for the most up-to-date version or all for all versions. Defaults to latest. | all | string 

### Response

Description  | Type
-----------  | ----
A binary zip file payload containing MOERs for the specified balancing authority for the past two years. Save the body to disk and unzip it. | Binary Data Zip File

## Marginal Emissions Forecast

```python
import requests
from requests.auth import HTTPBasicAuth

login_url = 'https://api2.watttime.org/v2/login'
token = requests.get(login_url, auth=HTTPBasicAuth('USERNAME', 'PASSWORD')).json()['token']

forecast_url = 'https://api2.watttime.org/v2/forecast'
headers = {'Authorization': 'Bearer {}'.format(token)}
params = {'ba': 'CAISO_NP15', 
          'starttime': '2019-02-20T16:45:30-0800', 
          'endtime': '2019-02-20T17:45:30-0800'}
rsp = requests.get(forecast_url, headers=headers, params=params)
print(rsp.text)
```

```shell
curl --include \
     --header "Authorization: Bearer <Access_Token>" \
  'https://api2.watttime.org/v2/forecast/?ba='
```

> Sample Response

```json
[
  {
    "generated_at": "2019-01-29T20:00:00.000Z", 
    "forecast": 
      [
        {
          "point_time": "2019-01-29T20:05:00.000Z",
          "ba": "MISO_IN",
          "value": 1350,
          "version": "0.0"
        },
        {
          "point_time": "2019-01-29T20:10:00.000Z",
          "ba": "MISO_IN",
          "value": 1300,
          "version": "0.0"},
         ..... <24 hours worth of datapoints>
      ]
  }
]
```
### `/forecast`

Obtain MOER forecast data for a given region. Omitting the starttime and endtime parameters will return the most recently generated forecast for a given region. Use the starttime and endtime parameters to obtain historical forecast data.  

Note that starttime and endtime define the time when a forecast was **generated**. Every five minutes, WattTime generates a new forecast which (depending on the region) is between 8 and 24 hours in duration. So, if you make a request to the forecast endpoint with starttime of Jan 1, 1:00 and endtime Jan 1, 1:05, you will receive the forecast (all 288 values in the forecast window) generated at 1:00, and the forecast generated at 1:05 on January 1.  
  
Access to this endpoint is restricted to customers with PRO subscriptions.

<aside class="success">
Historical forecast queries are limited to a maximum time span of 1 day. If you want to query more data than that, please break up your request into multiple queries.
</aside>

### HTTP Request

`GET https://api2.watttime.org/v2/forecast`

### URL Query Parameters

Parameter | Description | Example | Type
--------- | ----------- | ------- | ----
ba | Balancing authority abbreviation | CAISO_ZP26 | string 
starttime | ISO 8601 timestamp (inclusive) - Omit to obtain the real-time forecast. If included, those forecasts generated between start and endtime are returned. | 2019-02-20T16:45:30-0800 | string
endtime | ISO 8601 timestamp (inclusive)- Omit to obtain the real-time forecast. If included, those forecasts generated between start and endtime are returned. | 2019-02-20T17:45:30-0800 | string


### Response

Attribute | Description | Example | Type
--------- | ----------- | ------- | ----
generated_at | ISO 8601 timestamp for when the forecast was generated  | 2019-01-29T14:55:00.000Z | string 
forecast | List of forecasts - see below |  | object

The value of the ‘forecast’ key is a list of dictionaries containing the following keys:

Key | Description | Example | Type
--- | ----------- | ------- | ----
ba | Balancing authority abbreviation | CAISO_ZP26 | string 
point_time | ISO8601 UTC date/time format indicating when this data became valid | 2019-01-29T14:55:00.00Z | string
value | MOER value (corresponding to datatype above) | 909.06 | float 
version | MOER version (Not present and not applicable for other datatypes) | 2.0 | string 
  
    