> **Note:** This tutorial relies on **CMR-STAC**, which will continue to evolve in order to align with the STAC spec over time. As these changes are released, we will update this tutorial as necessary. If you are encountering an issue with the tutorial, please check out the [CHANGELOG.md](https://git.earthdata.nasa.gov/projects/LPDUR/repos/hls-tutorial/browse/CHANGELOG.md) and make sure that you have copied/cloned/downloaded the latest release of this tutorial.        
---  
# Getting Started with Cloud-Native HLS Data in Python:  
## Extracting an EVI Time Series from Harmonized Landsat and Sentinel-2 (HLS) data in the Cloud using CMR's SpatioTemporal Asset Catalog (CMR-STAC)  
---
# Objective:
NASA's Land Processes Distributed Active Archive Center (LP DAAC) archives and distributes HLS products in the LP DAAC Cumulus cloud archive as Cloud Optimized GeoTIFFs (COG). This tutorial will demonstrate how to query and subset HLS data using the NASA Common Metadata Repository (CMR) SpatioTemporal Asset Catalog (STAC). Because these data are stored as COGs, this tutorial will teach users how to load subsets of individual files into memory for just the bands you are interested in--a paradigm shift from the more common workflow where you would need to download a .zip/HDF file containing every band over the entire scene/tile. This tutorial covers how to process HLS data (quality filtering and EVI calculation), visualize, and "stack" the scenes over a region of interest into an [xarray](http://xarray.pydata.org/en/stable/) data array, calculate statistics for an EVI time series, and export as a comma-separated values (CSV) file--providing you with all of the information you need for your area of interest without having to download the source data file.       
## Use Case Example:  
This tutorial was developed using an example use case for crop monitoring over a single large farm field in northern California. **The goal of the project is to observe HLS-derived mean EVI over a farm field in northern California without downloading the entirety of the HLS source data.**  

## Products Used:
**1. PROVISIONAL daily 30 meter (m) global HLS Sentinel-2 Multi-spectral Instrument Surface Reflectance - [HLSS30.015](https://doi.org/10.5067/HLS/HLSS30.015)**    
  - **Science Dataset (SDS) layers:**    
    - B8A (NIR Narrow)    
    - B04 (Red)    
    - B02 (Blue)    
    - Fmask (Quality)  

**2. PROVISIONAL daily 30 meter (m) global HLS Landsat-8 OLI Surface Reflectance - [HLSL30.015](https://doi.org/10.5067/HLS/HLSL30.015)**  
 - **Science Dataset (SDS) layers:**  
    - B05 (NIR)  
    - B04 (Red)  
    - B02 (Blue)  
    - Fmask (Quality)       

<div class="alert alert-block alert-warning" >
<b>Disclaimer:</b> This tutorial uses the <b>PROVISIONAL</b> Version 1.5 daily 30 meter (m) global Harmonized Landsat Sentinel-2 (HLS) Sentinel-2 Multi-spectral Instrument Surface Reflectance (HLSS30) product and the <b>PROVISIONAL</b> Version 1.5 daily 30 meter (m) global Harmonized Landsat Sentinel-2 (HLS) Landsat-8 OLI Surface Reflectance (HLSL30) data. </div>   

---
# Prerequisites:
*Disclaimer: This tutorial has been tested on Windows and MacOS using the specifications identified below.*  
+ #### Python Version 3.7  
  + `xarray`  
  + `shapely`  
  + `geopandas`  
  + `pandas`  
  + `geoviews`  
  + `holoviews`    
  + `gdal`  
  + `rasterio`  
  + `xarray`  
  + `matplotlib`  
  + `cartopy`  
  + `scikit-image`  
  + `hvplot`  
  + `pyepsg`  
---  

# Procedures:     
## Getting Started:      
#### 1. This tutorial uses data from HLS S30 and L30 V1.5. The data needed to execute this tutorial will be accessed directly from NASA's Cumulus cloud archive in Section 4.
 - Ancillary Files Needed:  
    - [Field_Boundary.geojson](https://git.earthdata.nasa.gov/projects/LPDUR/repos/hls-tutorial/browse/Field_Boundary.geojson)  

The `Field_Boundary.json` file will need to be downloaded into the same directory as the tutorial in order to execute the tutorial.    
#### 2.	Copy/clone/download the [HLS Tutorial repo](https://git.earthdata.nasa.gov/rest/api/latest/projects/LPDUR/repos/hls-tutorial/archive?format=zip), or the desired tutorial from the LP DAAC Data User Resources Repository:   
 -  [Getting Started with Cloud-Native HLS Data in Python Jupyter Notebook](https://git.earthdata.nasa.gov/projects/LPDUR/repos/HLS-tutorial/browse/HLS_Tutorial.ipynb)   
## Python Environment Setup
> #### 1. It is recommended to use [Conda](https://conda.io/docs/), an environment manager, to set up a compatible Python environment. Download Conda for your OS [here](https://www.anaconda.com/download/). Once you have Conda installed, Follow the instructions below to successfully setup a Python environment on Windows, MacOS, or Linux.
> #### 2. Setup  
> - Using your preferred command line interface (command prompt, terminal, cmder, etc.) type the following to successfully create a compatible python environment:
>   - `conda create -n hlstutorial -c conda-forge --yes python=3.7 gdal rasterio shapely geopandas geoviews holoviews xarray matplotlib cartopy scikit-image hvplot pyepsg`   
> `conda install jupyter notebook --yes`       
>   - `conda activate hlstutorial`  
>   - `jupyter notebook`  
  TIP: Having trouble activating your environment, or loading specific packages once you have activated your environment? Try the following:
  > Type: `conda update conda` or `conda update --all`     

If you prefer to not install Conda, the same setup and dependencies can be achieved by using another package manager such as pip and the [requirements.txt file](https://git.earthdata.nasa.gov/projects/LPDUR/repos/hls-tutorial/browse/requirements.txt) listed above.  
[Additional information](https://conda.io/docs/user-guide/tasks/manage-environments.html) on setting up and managing Conda environments.    
#### Still having trouble getting a compatible Python environment set up? Contact [LP DAAC User Services](https://lpdaac.usgs.gov/lpdaac-contact-us/).      
## Setting up a netrc File:  
You will need a netrc file containing your NASA Earthdata Login credentials in order to execute this tutorial. If you want to manually create your own netrc file, download the [.netrc file template](https://git.earthdata.nasa.gov/projects/LPDUR/repos/daac_data_download_python/browse/.netrc), add your credentials, and save to your home directory. If you want to use the python script to set up a netrc file but do not need to download any files, copy/clone/download the [EarthdataLoginSetup.py](https://git.earthdata.nasa.gov/projects/LPDUR/repos/daac_data_download_python/browse/EarthdataLoginSetup.py) script and execute it: `python EarthdataLoginSetup.py`. You will be prompted for your NASA Earthdata Login Username and Password, hit enter once you have submitted your credentials.    
---
# Contact Information:
#### Author: Cole Krehbiel¹   
**Contact:** LPDAAC@usgs.gov  
**Voice:** +1-866-573-3222  
**Organization:** Land Processes Distributed Active Archive Center (LP DAAC)  
**Website:** https://lpdaac.usgs.gov/  
**Date last modified:** 03-15-2021  

¹KBR, Inc., contractor to the U.S. Geological Survey, Earth Resources Observation and Science (EROS) Center,  
 Sioux Falls, South Dakota, USA. Work performed under USGS contract G15PD00467 for LP DAAC².  
²LP DAAC Work performed under NASA contract NNG14HH33I.
