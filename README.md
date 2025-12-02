# ISMIP Antarctic Surface Mass Balance

This repository contains scripts for processing CMIP climate model data to generate surface mass balance (SMB) anomalies for Antarctic ice sheet simulations. The scripts regrid CMIP data onto the ISMIP 8km South Polar Stereographic grid and compute SMB anomalies relative to the 1995-2014 climatology.

## Overview

The workflow consists of two main steps:

1. **`preprocess_smb.py`**: Processes raw CMIP data files to compute SMB on the ISMIP 8km grid
2. **`compute_smb_anomaly.py`**: Computes SMB anomalies relative to the 1995-2014 climatology

## Getting Started

### Clone the Repository

```bash
git clone https://github.com/Jonniebarnsley/ismip_antarctic_smb.git
cd preprocess_ismip_smb
```

### Environment

Create the conda environment using the provided `environment.yml` file:

```bash
conda env create -f environment.yml
conda activate ismip_antarctic_smb
```

### Required Files

- **`ISMIP6_grid.nc`**: The ISMIP 8km (761×761) South Polar Stereographic grid definition (included in repository)

### Environment Variables

Set the `DATA_HOME` environment variable to specify where regridding mapping files should be stored, by adding the following line to your shell configuration file (e.g., `.bashrc` or `.zshrc`):

```bash
export DATA_HOME=path/to/data/directory/
```

If not set, the default is `~/data/`. Mapping files will be saved in `$DATA_HOME/regridding/`.

## Pipeline Execution

### Step 1: Preprocess SMB (`preprocess_smb.py`)

This script processes raw CMIP data to compute surface mass balance using the formula:

```
SMB = pr - evspsbl - mrro
```

where:
- `pr`: precipitation
- `evspsbl`: evaporation and sublimation
- `mrro`: runoff

**Usage:**

```bash
python preprocess_smb.py <input_dir1> <input_dir2> ... <output_dir>
```

**Arguments:**
- `input_dir1, input_dir2, ...`: Directories containing netCDF files with CMIP data for individual variables. Must include at least `pr`, `evspsbl`, and `mrro` directories. Can include additional variables (e.g., `tas`). Order doesn't matter.
- `output_dir`: Directory where output files will be saved (must be the last argument)

**Example:**

```bash
python preprocess_smb.py /path/to/pr /path/to/evspsbl /path/to/mrro /path/to/output
```

**What it does:**
1. Validates that input directories contain required variables (`pr`, `evspsbl`, `mrro`)
2. Computes annual means from monthly CMIP data
3. Regrids all variables from native lat-lon grid to ISMIP 8km grid using bilinear interpolation
4. Calculates SMB from the regridded variables
5. Saves output as: `output_dir/<experiment>/smb_<model>_<experiment>_8km.nc`

### Step 2: Compute SMB Anomaly (`compute_smb_anomaly.py`)

This script computes SMB anomalies relative to the 1995-2014 climatology and converts units to meters per year ice equivalent.

**Usage:**

```bash
python compute_smb_anomaly.py <historical_file> <scenario_file> <anomaly_dir>
```

**Arguments:**
- `historical_file`: Path to netCDF file containing historical SMB data (from Step 1)
- `scenario_file`: Path to netCDF file containing scenario SMB data (from Step 1)
- `anomaly_dir`: Output directory for SMB anomaly files

**Example:**

```bash
python compute_smb_anomaly.py \
    output/historical/smb_CESM2_historical_8km.nc \
    output/ssp585/smb_CESM2_ssp585_8km.nc \
    anomalies/
```

**What it does:**
1. Loads historical and scenario SMB data and concatenates them along the time dimension
2. Trims data to years 1995-2300
3. Converts units from kg m⁻² s⁻¹ to m a⁻¹ (ice equivalent) using ice density of 917 kg m⁻³
4. Computes climatology over 1995-2014
5. Calculates anomaly: `SMB_anomaly = SMB - climatology`
6. Saves one netCDF file per year

## Regridding Details

The regridding process (`regrid_CMIP_to_ISMIP.py`) handles several common issues with CMIP data:

- **Cyclic points**: Adds cyclic longitude point if longitude range is less than 360°
- **Pole holes**: Fills missing pole data by copying the southernmost latitude row
- **NaN handling**: Fills NaN values in runoff (`mrro`) with zeros before regridding
- **Method**: Uses bilinear interpolation by default (conserve method also available)

The regridder can also be used standalone:

```bash
python regrid_CMIP_to_ISMIP.py <input_file> <output_file> [--method bilinear] [--annual]
```

## Expected Input Data Structure

Input directories for `preprocess_smb.py` should contain CMIP netCDF files with:
- Time-varying data in CF-compliant format
- Lat-lon grid coordinates
- Required global attributes: `source_id`, `variable_id`, `experiment_id`

Example directory structure:
```
data/
├── pr/
│   ├── pr_CESM2_historical_185001-200012.nc
│   └── pr_CESM2_historical_200101-201412.nc
├── evspsbl/
│   ├── evspsbl_CESM2_historical_185001-200012.nc
│   └── evspsbl_CESM2_historical_200101-201412.nc
└── mrro/
    ├── mrro_CESM2_historical_185001-200012.nc
    └── mrro_CESM2_historical_200101-201412.nc
```

## Output Data Structure

After running the pipeline:

```
output/
├── historical/
│   └── smb_CESM2_historical_8km.nc
└── ssp585/
    └── smb_CESM2_ssp585_8km.nc

anomalies/
├── historical/
│   └── CESM2/
│       ├── smb_anomaly_CESM2_historical_8km_1995.nc
│       ├── smb_anomaly_CESM2_historical_8km_1996.nc
│       └── ...
└── ssp585/
    └── CESM2/
        ├── smb_anomaly_CESM2_ssp585_8km_2015.nc
        ├── smb_anomaly_CESM2_ssp585_8km_2016.nc
        └── ...
```

## Authors

Jonnie Barnsley  
November 2025
