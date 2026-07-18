# Sri Lanka Climate Dataset

District-level daily and monthly climate data (precipitation, temperature, dew point, and derived relative humidity) for all 25 districts of Sri Lanka, extracted for use in climate-health research (specifically, a leptospirosis incidence study).

## Overview

This repository contains scripts and data for extracting, processing, and aggregating climate variables at the district administrative level across Sri Lanka, covering the period **2006-12-23 to present**.

| Variable | Source Dataset | Provider | Native Resolution |
|---|---|---|---|
| Precipitation | CHIRPS Daily | UCSB Climate Hazards Group | ~4.8 km (0.05°) |
| Temperature (2m) | ERA5 Daily | ECMWF / Copernicus | ~25 km (0.25°) |
| Dew Point Temperature (2m) | ERA5 Daily | ECMWF / Copernicus | ~25 km (0.25°) |
| Relative Humidity | *Derived* from temperature + dew point (Magnus formula) | — | ~25 km (0.25°) |

All climate variables were extracted using [Climate Engine](https://app.climateengine.org), a free web platform built on Google Earth Engine.

## Data Sources & Boundaries

- **District boundaries**: Sri Lanka Subnational Administrative Boundaries (COD-AB), sourced from the **Survey Department of Sri Lanka**, distributed via UN OCHA's [Humanitarian Data Exchange (HDX)](https://data.humdata.org/dataset/cod-ab-lka). Admin level 2 (25 districts) was used.
- **Climate data**: [Climate Engine](https://app.climateengine.org) (CHIRPS, ERA5)

## Repository Structure

```
Sri_Lanka_Climate_Dataset/
├── README.md
├── shapefiles/
│   └── lka_admin2.zip              # District boundary shapefile (HDX COD-AB, Admin 2)
├── data_raw/
│   ├── precipitation/              # 25 CSVs, one per district (CHIRPS daily, mm)
│   └── temperature_dewpoint/       # 25 CSVs, one per district (ERA5 daily, °C)
├── data_processed/
│   ├── daily_climate_combined.csv  # All districts merged, daily resolution
│   └── monthly_climate_combined.csv # Aggregated to monthly resolution
└── scripts/
    ├── 01_combine_raw_csvs.R           # Merges the 50 raw district CSVs into one dataset
    ├── 02_calculate_relative_humidity.R # Derives RH from temperature + dew point
    └── 03_aggregate_to_monthly.R       # Aggregates daily data to monthly
```

## Methodology

1. **Boundary extraction**: District polygons uploaded to Climate Engine as a custom shapefile (Region → Make Graph → Custom Polygon from Shape File).
2. **Daily extraction**: For each of the 25 districts, daily mean precipitation (CHIRPS) and daily mean temperature + dew point temperature (ERA5) were extracted and exported as individual CSV files.
3. **Relative humidity estimation**: Since RH is not directly available from CHIRPS/ERA5, it was estimated from daily mean air temperature (T) and daily mean dew point temperature (Td) using the Magnus formula:

   ```
   RH = 100 × exp[ (17.625 × Td)/(243.04 + Td) − (17.625 × T)/(243.04 + T) ]
   ```

   This approach is widely used in climatological and epidemiological studies (Alduchov & Eskridge, 1996).

4. **Monthly aggregation**: Daily values were aggregated to monthly resolution:
   - Precipitation → **sum** of daily values within the month
   - Temperature and relative humidity → **mean** of daily values within the month

## Known Issues / Data Quality Notes

- A small number of districts have complex, multi-part boundary geometry (e.g. Jaffna, Kilinochchi, Mannar — due to small offshore islands). These were tested individually in Climate Engine before the full extraction; all extracted successfully with the HDX shapefile used here (no manual polygon redrawing was required).
- Any missing daily values (if present) are documented in `data_processed/` alongside the merge script's output log, following standard practice: gaps under ~0.1% of total observations, uniformly distributed across districts, were left as missing rather than imputed.

## Requirements

R (≥ 4.0) with the following packages:

```r
install.packages(c("tidyverse", "lubridate"))
```

