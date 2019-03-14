## Running WRF and WPS Data Cases

---
### Rough Katrin Guide

#### Getting Katrina data

We want to get the Katrina data into our DATA folder

```
cd /software/ncar-wrf_3.8.1/build/DATA
wget http://www2.mmm.ucar.edu/wrf/TUTORIAL_DATA/Katrina.tar.gz
tar -xf Katrina.tar
```

#### Looking into the our data

They want us to run g1print on /DATA/Katrina/avn_050828_00_00
g1print.exe list the content of a GRIB1 file

#### Link the GFS VTable

```
ln -sf ungrib/Variable_Tables/Vtable.GFS Vtable
```
#### Link in the GRIB data

```
./link_grib.csh ../DATA/Katrina/avn
```
#### Edit namelist.wps

```
sudo nano namelist.wps

max_dom = 1
start_date = '2005-08-28_00:00:00'
end_date = '2005-08-29_00:00:00'
interval_seconds = 21600
prefix = 'FILE'
```
#### Run ungrib

Creates the intermediate files

```
./ungrib.exe >& ungrib_data.log
```
#### Intermediate files:

```
./util/rd_intermediate.exe FILE:2005-08-28_00
./util/plotfmt.exe FILE:2005-08-28_00 ; idt gmeta
```

#### Edit the namelist file

```
cd WPS/
sudo nano namelist.wps

max_dom = 1
parent_id = 1,
parent_grid_ratio = 1,
i_parent_start = 1,
j_parent_start = 1,
e_we = 98,
e_sn = 70,
geog_data_res = '10m',
dx = 30000,
dy = 30000,
map_proj = 'mercator',
ref_lat = 25.00,
ref_lon = -89.00,
truelat1 = 0.0,
truelat2 = 0.0,
stand_lon = -89.0,
geog_data_path = 'Your WPS_GEOG data location'
```

#### Running geogrid.exe

```
ncl util/plotgrids.ncl

./geogrid.exe
```
Should see:
Parsed 20 entries in GEOGRID.TBL
Processing domain 1 of 1
Processing XLAT and XLONG
Processing MAPFAC
Processing F and E
Processing ROTANG
Processing LANDUSEF
Calculating landmask from LANDUSEF ( WATER = 16 )
Processing HGT_M
Processing SOILTEMP
Processing SOILCTOP
Processing SCT_DOM
Processing SOILCBOT
Processing SCB_DOM
Processing ALBEDO12M
Processing GREENFRAC
Processing SNOALB
Processing SLOPECAT
Processing SLOPECAT
Processing CON
Processing VAR
Processing OA1
Processing OA2
Processing OA3
Processing OA4
Processing OL1
Processing OL2
Processing OL3
Processing OL4
Processing VAR_SSO
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
! Successful completion of geogrid.         !
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

### References/Information

Start here:
https://wiki.uio.no/mn/geo/geoit/index.php/WRFand_WRF-CHEM

WRF-NMM(online tutorial) - https://dtcenter.org/wrf-nmm/users/OnLineTutorial/NMM/Compilation/wps_compile3.php
SST (Sea Surface Temperature) - https://climatedataguide.ucar.edu/climate-data/sst-data-sets-overview-comparison-table
GFS (Global Forecast System) - http://www.emc.ncep.noaa.gov/index.php?branch=GFS
NCEP (National Centers for Environmental Prediction's) - https://www.emc.ncep.noaa.gov/
AVN/GFS-AVN - https://www.ncdc.noaa.gov/data-access/model-data/model-datasets/global-forcast-system-gfs
g1print - List the contents of a GRIB1 file (https://dtcenter.org/wrf-nmm/users/OnLineTutorial/NMM/Compilation/wps_compile3.php)
GrADS (Grid Analysis and Display System) - http://cola.gmu.edu/grads/grads.php
MSLP (Mean Sea Level Pressure) - http://www.bom.gov.au/australia/charts/Interpreting_MSLP.shtml
VTable - https://ncarrda.blogspot.com/2016/01/wrf-able-datasets.html
GRIB (General Regularly-distributed Information in Binary form) - https://en.wikipedia.org/wiki/GRIB
namelist.wps - http://www2.mmm.ucar.edu/wrf/users/docs/user_guide_V3/users_guide_chap3.htm#_Description_of_the_1
