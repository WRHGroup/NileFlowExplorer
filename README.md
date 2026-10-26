# NileFlowExplorer

NileFlowExplorer is a browser-based tool for exploring simulated naturalized streamflow across the Nile Basin. It shows daily flows simulated with HEC-HMS for 250 sub-basins and 341 river reaches, driven by satellite-based historical climate data (CHIRPS and CHIRTS, 1983–2014) and by downscaled, bias-corrected projections from 19 CMIP6 global climate models under SSP1-2.6, SSP2-4.5, and SSP5-8.5.

**Open the explorer:** https://wrhgroup.github.io/NileFlowExplorer/

**Download the data:** https://doi.org/10.5281/zenodo.23491530

The full methodology and results will be published in a forthcoming paper. Please read the [disclaimer](#disclaimer) before using the data.

## What the explorer does

- Shows the model sub-basins and reaches on an interactive map with selectable basemaps.
- Plots streamflow for any sub-basin or reach as a centred moving average (window of 1–60 years), calendar-year means, or raw daily values.
- Summarises each scenario as the ensemble median with a 10th–90th percentile band across climate models, and reports the number of models in each ensemble (n = X models). Individual model runs can also be shown.
- Converts units between million m³ day⁻¹ (the native unit of the data), m³ s⁻¹, and km³ yr⁻¹ (billion m³ per year).
- Exports the plotted series to CSV, following the selected view, averaging window, and units.

Everything runs in the browser. The HDF5 file is read locally with [h5wasm](https://github.com/usnistgov/h5wasm) in a background thread, and only the selected element is loaded, so the file is never uploaded and large files remain responsive.

## Using the explorer

1. Download the naturalized flow file (`.h5`) from [Zenodo](https://doi.org/10.5281/zenodo.23491530).
2. Open the [explorer](https://wrhgroup.github.io/NileFlowExplorer/), read the disclaimer, and select **I understand**.
3. Load the `.h5` file with the **HDF5 flow file** button.
4. Click a sub-basin or reach on the map, or choose one from the list.

Elements shown in grey on the map have no series in the loaded file.

## Reading the data without the explorer

The `.h5` file is a pandas HDF5 store with one key per model element (for example `Reach-80` or `Subbasin-78`). Each key holds a pandas DataFrame with a daily `DatetimeIndex` and one column per simulation, in million m³ day⁻¹. Projection columns are named `<GCM>_<scenario>`, for example `MIROC6_ssp245`. The explorer treats columns ending in `_historical` or `_hist`, and columns without a scenario suffix, as historical simulations.

```python
import pandas as pd  # reading HDF5 stores also requires PyTables: pip install pandas tables

path = "nile_naturalized_flow.h5"  # use the name of the file downloaded from Zenodo

# List the available sub-basins and reaches
with pd.HDFStore(path, mode="r") as store:
    elements = sorted(key.lstrip("/") for key in store.keys())

# Daily flows for one reach (million m3/day)
flow = pd.read_hdf(path, key="Reach-80")

# Calendar-year means, and the SSP2-4.5 ensemble median
annual = flow.groupby(flow.index.year).mean()
ssp245_median = annual.filter(regex="_ssp245$").median(axis=1)

# Convert million m3/day to km3/year
annual_km3 = annual * 365.25 / 1000
```

## Methods (summary)

### Hydrological model

Daily streamflow is simulated with the Hydrologic Modeling System of the US Army Corps of Engineers Hydrologic Engineering Center (HEC-HMS). The Nile Basin is divided into 250 sub-basins connected by 341 reaches. Catchment delineations are based on existing and planned infrastructure, such as dams and irrigation schemes, so that flows are available at locations relevant to water resources planning. Physiographic inputs include topography from the Shuttle Radar Topography Mission (SRTM) at a 30 meter resolution (Jarvis et al., 2008), lake bathymetry from GLOBathy (Khazaei et al., 2022), land cover from the Copernicus Global Land Service, and hydrologic soil groups from HYSOGs250m (Ross et al., 2018). The flows are naturalized. The representation of reservoirs, wetlands, and water abstractions, and the calibration and validation of the model against observed flow records, will be described in the forthcoming paper.

### Climate forcing

Historical simulations (1983–2014) are driven by daily precipitation from CHIRPS v2 (Funk et al., 2015) and daily maximum and minimum temperatures from CHIRTS (Verdin et al., 2020). These products address the limitations of the sparse and unevenly distributed station network in Africa (Dinku, 2019).

Future simulations are driven by projections from 19 CMIP6 global climate models (GCMs) (Table 1) under three Shared Socioeconomic Pathways (SSPs): SSP1-2.6 (sustainability, low emissions), SSP2-4.5 (middle-of-the-road, medium emissions), and SSP5-8.5 (fossil-fuelled development, very high emissions) (O'Neill et al., 2016; Riahi et al., 2017). We screened GCM projections based on the finest available spatial resolution, daily temporal completeness for the historical and late-century periods, availability of precipitation and temperature variables, and one model per institution, with the aim of capturing a wide range of future precipitation and temperature conditions across the Nile Basin. The explorer reports the number of models (n) in each scenario ensemble for the selected element.

### Bias correction and downscaling

Raw GCM outputs were processed in five steps following guidance on post-processing climate model output for regional impact studies (e.g., Werner and Cannon, 2016). (1) Daily precipitation, minimum temperature, and maximum temperature from the observations and the GCMs were harmonized across units, calendars, time periods, and spatial domains. (2) The observations were coarsened to each GCM grid. (3) The coarse GCM fields were bias-corrected against the coarse observations using quantile mapping, including Detrended Quantile Mapping (DQM), Empirical Quantile Mapping (EQM), and Quantile Delta Mapping (QDM). (4) The bias-corrected fields were downscaled using constructed analogues; for each simulated day, the 30 most similar observed patterns were selected, weighted using ridge regression, and combined to reconstruct the high-resolution field. (5) A second bias correction was applied to the downscaled fields to reduce residual errors, including the drizzle bias in precipitation. The result is a set of daily 0.25° fields of precipitation and maximum and minimum temperature for the historical period (1983–2014) and the three SSPs (2015–2100).

**Table 1.** Downscaled and bias-corrected CMIP6 global climate models.

| # | Institution | Model | Variant | Nominal resolution (km) |
|---|---|---|---|---|
| 1 | AS-RCEC (Taiwan) | TaiESM1 | r1i1p1f1 | 100 |
| 2 | AWI (Germany) | AWI-ESM-1-REcoM | r1i1p1f1 | 250 |
| 3 | BCC (China) | BCC-CSM2-MR | r1i1p1f1 | 100 |
| 4 | CAS (China) | FGOALS-g3 | r1i1p1f1 | 250 |
| 5 | CMCC (Italy) | CMCC-ESM2 | r1i1p1f1 | 100 |
| 6 | CNRM-CERFACS (France) | CNRM-CM6-1 | r1i1p1f2 | 250 |
| 7 | CSIRO (Australia) | ACCESS-ESM1-5 | r1i1p1f1 | 250 |
| 8 | CSIRO-ARCCSS (Australia) | ACCESS-CM2 | r1i1p1f1 | 250 |
| 9 | DKRZ (Germany) | MPI-ESM1-2-HR | r1i1p1f1 | 100 |
| 10 | EC-Earth-Consortium (Europe) | EC-Earth3 | r1i1p1f1 | 100 |
| 11 | INM (Russia) | INM-CM4-8 | r1i1p1f1 | 100 |
| 12 | IPSL (France) | IPSL-CM6A-LR | r1i1p1f1 | 250 |
| 13 | KIOST (South Korea) | KIOST-ESM | r1i1p1f1 | 250 |
| 14 | MIROC (Japan) | MIROC6 | r1i1p1f1 | 250 |
| 15 | MRI (Japan) | MRI-ESM2-0 | r1i1p1f1 | 100 |
| 16 | NCC (Norway) | NorESM2-LM | r1i1p1f1 | 250 |
| 17 | CCCma (Canada) | CanESM5 | r1i1p1f1 | 500 |
| 18 | NOAA (USA) | GFDL-ESM4 | r1i1p1f1 | 100 |
| 19 | NUIST (China) | NESM3 | r1i1p1f1 | 250 |

### What is shown in the explorer

For each element, the explorer computes calendar-year means from the daily series and, optionally, a centred moving average over a full window (years without a complete window are left blank). The ensemble median and the 10th and 90th percentiles are computed across GCMs for each SSP and each year. With small ensembles, the percentiles are interpolated between a few members and should be read as an approximate range rather than a probability statement.

## How to cite

The accompanying paper is in preparation. Until it is published, please cite the dataset on Zenodo (https://doi.org/10.5281/zenodo.23491530) using the citation given on its record page, and this repository using [`CITATION.cff`](CITATION.cff) (GitHub shows this as **Cite this repository** in the sidebar).

## Disclaimer

NileFlowExplorer and the data it displays are provided for research and information purposes only. The same notice is shown as a pop-up each time the website is opened.

- The streamflow series are naturalized flows simulated with a hydrological model. They are not measured flows, and they do not represent the regulated flows that actually occur at any location.
- Future flows are based on climate scenarios. They describe plausible outcomes under particular emission pathways, not predictions, and they carry substantial uncertainty from the climate models, the scenarios, the downscaling, and the hydrological model.
- The website and data are provided "as is", without warranty of any kind, express or implied, including warranties of accuracy, completeness, timeliness, merchantability, or fitness for a particular purpose.
- The authors, their institutions, and funders accept no liability for any loss or damage of any kind, whether direct, indirect, incidental, or consequential, arising from the use of, or reliance on, this website or the data. Users are solely responsible for how they interpret and apply them.
- The data are not a substitute for site-specific analysis, professional engineering judgement, or official information from national agencies and basin organizations, and should not be the sole basis for design, operational, investment, legal, or policy decisions.
- The boundaries, names, and designations on the maps do not imply any opinion on the legal status of any territory or the delimitation of any boundary. Nothing in this repository represents a position on the allocation or use of the waters of the Nile.

## Repository structure

```
NileFlowExplorer/
├── docs/                      Website served by GitHub Pages
│   ├── index.html             The explorer (single page, no build step)
│   ├── .nojekyll              Serve files as they are (no Jekyll processing)
│   └── shapefiles/            Sub-basin and reach layers (WGS 84 / UTM zone 36N)
├── CITATION.cff
├── LICENSE
└── README.md
```

The `.h5` flow file is not stored in this repository; it is distributed through Zenodo.

## Running locally and deploying

The page loads the shapefiles over HTTP, so it must be served rather than opened as a local file:

```bash
cd docs
python -m http.server 8000
# then open http://localhost:8000
```

To publish with GitHub Pages, go to **Settings → Pages** and set the source to the `main` branch and the `/docs` folder. Do not track the shapefiles with Git LFS, because GitHub Pages does not serve LFS files.

## Acknowledgements and third-party components

The explorer uses [Leaflet](https://leafletjs.com/), [Chart.js](https://www.chartjs.org/), [h5wasm](https://github.com/usnistgov/h5wasm), and [shpjs](https://github.com/calvinmetcalf/shapefile-js). Basemaps are provided by Esri, [OpenStreetMap](https://www.openstreetmap.org/copyright) contributors, and [OpenTopoMap](https://opentopomap.org/), under their respective terms.

## License

The code in this repository is released under the GNU General Public License v3.0 (see [`LICENSE`](LICENSE)). The flow data are distributed under the license stated on the Zenodo record.

## References

Dinku, T. (2019). Challenges with availability and quality of climate data in Africa. In *Extreme Hydrology and Climate Variability* (pp. 71–80). Elsevier.

Funk, C., Peterson, P., Landsfeld, M., Pedreros, D., Verdin, J., Shukla, S., Husak, G., Rowland, J., Harrison, L., Hoell, A., & Michaelsen, J. (2015). The climate hazards infrared precipitation with stations: a new environmental record for monitoring extremes. *Scientific Data*, 2, 150066. https://doi.org/10.1038/sdata.2015.66

Jarvis, A., Reuter, H. I., Nelson, A., & Guevara, E. (2008). *Hole-filled SRTM for the globe, Version 4*. CGIAR-CSI SRTM Database. https://srtm.csi.cgiar.org

Khazaei, B., Read, L. K., Casali, M., Sampson, K. M., & Yates, D. N. (2022). GLOBathy, the global lakes bathymetry dataset. *Scientific Data*, 9, 36. https://doi.org/10.1038/s41597-022-01132-9

O'Neill, B. C., Tebaldi, C., van Vuuren, D. P., Eyring, V., Friedlingstein, P., Hurtt, G., Knutti, R., Kriegler, E., Lamarque, J.-F., Lowe, J., Meehl, G. A., Moss, R., Riahi, K., & Sanderson, B. M. (2016). The Scenario Model Intercomparison Project (ScenarioMIP) for CMIP6. *Geoscientific Model Development*, 9, 3461–3482. https://doi.org/10.5194/gmd-9-3461-2016

Riahi, K., van Vuuren, D. P., Kriegler, E., et al. (2017). The Shared Socioeconomic Pathways and their energy, land use, and greenhouse gas emissions implications: an overview. *Global Environmental Change*, 42, 153–168. https://doi.org/10.1016/j.gloenvcha.2016.05.009

Ross, C. W., Prihodko, L., Anchang, J., Kumar, S., Ji, W., & Hanan, N. P. (2018). HYSOGs250m, global gridded hydrologic soil groups for curve-number-based runoff modeling. *Scientific Data*, 5, 180091. https://doi.org/10.1038/sdata.2018.91

US Army Corps of Engineers Hydrologic Engineering Center. *HEC-HMS: Hydrologic Modeling System*. https://www.hec.usace.army.mil/software/hec-hms/

Verdin, A., Funk, C., Peterson, P., Landsfeld, M., Tuholske, C., & Grace, K. (2020). Development and validation of the CHIRTS-daily quasi-global high-resolution daily temperature data set. *Scientific Data*, 7, 303. https://doi.org/10.1038/s41597-020-00643-7

Werner, A. T., & Cannon, A. J. (2016). Hydrologic extremes: an intercomparison of multiple gridded statistical downscaling methods. *Hydrology and Earth System Sciences*, 20, 1483–1508. https://doi.org/10.5194/hess-20-1483-2016
