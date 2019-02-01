[![PyPI](https://img.shields.io/badge/python-3.6-yellow.svg)]()
[![license](https://img.shields.io/github/license/khider/simplewg.svg)]()
[![Version](https://img.shields.io/github/release/khider/simplewg.svg)]()

# simplewg

**Python code to generate weather from location climatology**

This code is written for the MINT project. In short, the aim is to analyze the current climatology, select a year based on a criteria (e.g., 60% more rain than average), and use a weather generator to create possible rendition of the weather conditions.

**Table of contents**

* [What is it?](#what)
* [Version Information](#version)
* [Quickstart Guide](#quickstart)
* [Requirements](#req)
* [Files in this repository](#files)
* [Cite](#cite)
* [Contact](#contact)
* [License](#license)

## <a name = "what">What is it?</a>

This Python routine creates data transformation wrappers around [GWGEN](https://github.com/ARVE-Research/gwgen): A global weather generator for daily data.

Specifically, the package analyzes monthly climatology to determine a baseline for a user-defined change in one of the climate variables (e.g., precipitation) and generates weather for a season similar to baseline climatology.

## <a name = "version">Version Information</a>
- v0.0.3: Update metadata for lat/lon  
- v0.0.2: Fix bug in lats/lons due to np.arange spacing.
- v0.0.1: Support built for FLDAS datasets. Cloud cover is assumed to be 0.5 (50%) since it is not return by FLDAS.

## <a name = "quickstart">Quickstart Guide</a>

The routine is implemented from the command line in three steps (components):
1. Establish climatology from FLDAS_to_WGEN.py -> Python
2. Run the weather generator -> Fortran. Instructions to install the Fortran code can be found [here](https://github.com/ARVE-Research/gwgen).
3. Infer daily data and repackage according to the FLDAS format using WGEN_to_FLDAS.py

The three steps are detailed below:

__Step 1:__

`python FLDAS_to_WGEN.py path_monthly path_daily flagP min_lon max_lon min_lat max_lat min_month max_month level`

where:
* path_monthly (str): The path to the directory containing the monthly netCDF files  
* path_daily (str): The path to the directory containing the daily netCDF files  
* flagP (str): Name of the variable of interest as declared in the file  
* min_lon (float): Minimum longitude for the bounding box  
* max_lon (float): Maximum longitude for the bounding box  
* min_lat (float): Minimum latitude for the bounding box  
* max_lat (float): Maximum latitude for the bounding box  
* min_month (int): The first month of the season of interest  
* max_month (int): The last month of the season of interest  
* level: Event to be considered (e.g., +2sigma is 97.5% of the underlying distribution). Should be between 0 and 1.

Returns a .csv file for use to the WGEN.

*Example:*

`python FLDAS_to_WGEN.py '/Users/deborahkhider/Documents/MINT/Climate/MonthlyDatasets' '/Users/deborahkhider/Documents/MINT/Climate/Scenarios/FLDAS_NOAH01_A_EA_D.001' Rainf_f_tavg 32 33 13 14 6 8 0.6`

__Step 2:__

This requires a Fortran compiler:

`./weathergen <input-file.csv> <output-file.csv>`

Where:
* input-file.csv is generated from Step 1
* output-file.csv doesn't have to be an existing file in the directory.

__Step 3:__

This steps moves the data into a FLDAS-like netCDF file (same variable name).

Two caveats:
1. not all variables are generated and are ignored for the purpose of the weather generator.
2. Only data corresponding to the subset of the geographical region is stored in the file.

`python WGEN_to_FLDAS.py wgen_out wgen_in path_out file_prefix`

where:
* wgen_out: The csv file output from the weather generator
* wgen_in: The csv file input to the weather generator
* path_out: an existing directory in which to write out the netCDF files
* file_prefix: A prefix for the netcdf files

*Example:*

`python WGEN_to_FLDAS.py '/Users/deborahkhider/Documents/GitHub/simplewg/WGEN_out.csv' '/Users/deborahkhider/Documents/GitHub/simplewg/FLDAS_WGEN.csv' '/Users/deborahkhider/Documents/MINT/Climate/Scenarios/FLDAS_WGEN' 'FLDAS_NOAH01_A_EA_D.A'`

## <a name="req">Requirements</a>

### Software requirements

#### Python

Current version tested under Python 3.6 with the following dependencies.

- xarray v0.11.0
- numpy v1.15.0
- pandas v0.23.4
- glob2 v0.6

#### Fortran
To run wgen, a Fortran compiler needs to be installed.

### Data Requirements

This routine assumes that the data is stored as per FLDAS netCDF format and file organization. This makes use of **both** monthly and daily data.

**Caution**: Make sure to use the same source for the monthly and daily data. For instance, pair the monthly FLDAS-NOAH01 monthly with the corresponding daily.

## <a name = "cite"> Cite </a>

When using this package, please cite:

- Richardson, C. W.: Stochastic simulation of daily precipitation, temperature, and solar radiation, Water Resources Research, 17, 182–190, [doi:10.1029/WR017i001p00182](https://agupubs.onlinelibrary.wiley.com/doi/abs/10.1029/WR017i001p00182), 1981.
- T. G.: Global Historical Climatology Network - Daily (GHCN-Daily), Version 3.22, [doi:10.7289/V5D21VHZ, doi:10.7289/V5D21VHZ](https://doi.org/10.7289/V5D21VHZ), 2012
- Hahn, C. and Warren, S.: Extended Edited Synoptic Cloud Reports from Ships and Land Stations Over the Globe, 1952-1996 (with Ship data updated through 2008), [doi:10.3334/CDIAC/cli.ndp026c](https://doi.org/10.3334/CDIAC/cli.ndp026c), 1999

## <a name = "contact"> Contact </a>

Please report issues to <khider@usc.edu>

## <a name ="license"> License </a>

The project is licensed under the GNU Public License. Please refer to the file call license.
