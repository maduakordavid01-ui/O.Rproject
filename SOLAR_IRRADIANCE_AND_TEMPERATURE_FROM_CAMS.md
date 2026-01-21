# Solar Irradiance & Temperature Data from Copernicus CAMS (ADS)

This tutorial shows how to **automatically download historical solar irradiance and related meteorological data** from the **Copernicus Atmosphere Data Store (ADS)** using the `cdsapi` Python client.

Use-case examples:
- Solar PV energy yield estimation
- Performance ratio calculation
- Site selection & feasibility studies
- Comparison between modeled vs measured irradiance
- Input for PVlib or other solar simulation tools

## Why Copernicus CAMS (ADS)?

- Free high-quality reanalysis & near-real-time data
- Global coverage (0.25° × 0.25° grid)
- Multiple irradiance components: GHI, DNI, DHI, DNI
- Long historical archive (2004–present)
- Hourly data (very useful for PV simulations)
- Includes temperature, wind, humidity, cloud cover, etc.

## 1. Prerequisites

- Free Copernicus Atmosphere Data Store account → https://ads.atmosphere.copernicus.eu
- Registered API key (you'll find it in your user profile after registration)
- Python environment (Colab, local Jupyter, etc.)

Install required packages:

```bash
!pip install -q cdsapi netCDF4 pvlib xarray pandas
```
## 2. Get Your API Key & Set Up Authentication
After registering at https://ads.atmosphere.copernicus.eu/user/, copy your URL and key from the profile page.
You should see something like:

```
url: https://ads.atmosphere.copernicus.eu/api
key: 12345:abcdef12-3456-7890-abcd-ef1234567890
```

Recommended safe way — create a .cdsapirc file (never commit it to GitHub!):
```
# In terminal or ! in Colab
echo "url: https://ads.atmosphere.copernicus.eu/api" > ~/.cdsapirc
echo "key: YOUR_KEY_HERE" >> ~/.cdsapirc
chmod 600 ~/.cdsapirc
```

Alternative (less secure – only for quick tests):
```
import cdsapi

client = cdsapi.Client(
    url='https://ads.atmosphere.copernicus.eu/api',
    key='YOUR_LONG_KEY_HERE:YOUR_UUID_HERE',
    verify=False   # only if you get SSL issues (not recommended long-term)
)
```

## 3. Define and Submit a Download Request
Example: June 2020 over a region covering Portugal & parts of Spain (adjust as needed).

```
import cdsapi

dataset = "cams-europe-air-quality-reanalyses"   # or "cams-global-reanalysis-eac4" etc.
# For solar radiation use: "cams-global-atmospheric-composition-reanalysis" or specific radiation datasets

c = cdsapi.Client()

c.retrieve(
    'cams-global-atmospheric-composition-reanalysis',
    {
        'variable': [
            'global_horizontal_irradiance',
            'direct_horizontal_irradiance',
            'direct_normal_irradiance',
            'diffuse_horizontal_irradiance',
            '2m_temperature',
            'surface_solar_radiation_downwards_clear_sky',
        ],
        'date': '2020-06-01/2020-06-30',
        'time': '00:00',               # or list of hours: ['00:00','03:00',...]
        'area': [44.5, -10, 36, 3.5],  # North, West, South, East  → Portugal + surroundings
        'format': 'netcdf',
    },
    'cams_irradiance_portugal_june2020.nc'
)
```

Popular datasets to explore:

```
reanalysis-era5-single-levels (ECMWF – very complete)
cams-global-reanalysis-eac4
cams-europe-air-quality-forecasts
cams-solar-radiation-time-series
```
Check full list & exact variable names here:
https://ads.atmosphere.copernicus.eu/cdsapp#!/dataset/

## 4. Read and Explore the Downloaded NetCDF File

```
import xarray as xr
import pandas as pd
import matplotlib.pyplot as plt

# Open the file
ds = xr.open_dataset('cams_irradiance_portugal_june2020.nc')

print(ds)

# Select a point (example: Lisbon area)
point_data = ds.sel(latitude=38.7, longitude=-9.1, method='nearest')

# Convert to pandas DataFrame
df = point_data.to_dataframe().reset_index()

# Plot GHI over time
plt.figure(figsize=(12, 5))
plt.plot(df['time'], df['ghi'], label='Global Horizontal Irradiance (W/m²)')
plt.title('GHI – Lisbon area – June 2020 (CAMS)')
plt.xlabel('Time')
plt.ylabel('Irradiance (W/m²)')
plt.grid(True)
plt.legend()
plt.show()
```

## 5. Convert to PVlib-friendly Format (optional)

```
import pvlib

# Example: create pvlib compatible DataFrame
pvlib_df = pd.DataFrame({
    'ghi':     df['ghi'],
    'dni':     df['dni'],
    'dhi':     df['dhi'],
    'temp_air': df['t2m'] - 273.15,   # Kelvin → Celsius
    'wind_speed': df.get('10m_u_component_of_wind', 0),  # if available
}, index=df['time'])

# You can now use this with pvlib.pvsystem, ModelChain, etc.
print(pvlib_df.head())
```

## 6. Tips & Best Practices

Large areas / long periods → split requests (API has limits)
Use area in [North, West, South, East] format
Prefer netcdf format (smaller & faster than grib)
Cache downloaded files — don't re-download the same data
For many locations → download once, then extract points with xarray
Combine with PVGIS, NASA POWER or local measurements for validation
Check variable long names & units in the NetCDF attributes

## 7. Useful Links

Copernicus ADS registration & API docs: https://ads.atmosphere.copernicus.eu
CAMS solar radiation documentation: https://climate.copernicus.eu/cams-solar-radiation
pvlib python library: https://pvlib-python.readthedocs.io
xarray tutorial (great for NetCDF): https://docs.xarray.dev

```
