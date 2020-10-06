# CoastSat_nocs

CoastSat_nocs is an open-source software toolkit written in Python that enables users to obtain time-series of shoreline position at any coastline worldwide from 30+ years (and growing) of Landsat 7, 8 and Sentinel-2. This is a toolkit which has been developed by Vos et al., 2019 originally named Coastsat found here (https://github.com/kvos/CoastSat).

Coastsat_nocs has banched from coastsat to include the following changes:
* Retrieve median composites of satellite data - I.e. ‘['2000-01-01', '2000-12-31']’ creates a single shoreline from all satellite images in the year 2000.
* The user can loop through multiple study areas rather than a single polygon
* Multiple date ranges (+ satellites) can be specified
* A co-registration process from Landsat to Sentinel-2*
* Automated shoreline editing models
* Instructions for shoreline change rate and forecasting (10- and 20-Year) using Digital Shoreline Analysis System (DSAS) - ArcMap plug-in.
* Landsat 5 not available in this code – working on solution

*Coregistration process uses displace() and displacement() from GEE. In some areas there remains a coregistration issue which can be seen when there are large and unexpected distances between shorelines delineated between Landsat and Sentinel images. GEE documents are relatviely unclear on the exact methods of the functions used to coregister images, but we are working on this issue.

The underlying approach of the CoastSat toolkit are described in detail in:
* Vos K., Splinter K.D., Harley M.D., Simmons J.A., Turner I.L. (2019). CoastSat: a Google Earth Engine-enabled Python toolkit to extract shorelines from publicly available satellite imagery. *Environmental Modelling and Software*. 122, 104528. https://doi.org/10.1016/j.envsoft.2019.104528
Example applications and accuracy of the resulting satellite-derived shorelines are discussed in:
* Vos K., Harley M.D., Splinter K.D., Simmons J.A., Turner I.L. (2019). Sub-annual to multi-decadal shoreline variability from publicly available satellite imagery. *Coastal Engineering*. 150, 160–174. https://doi.org/10.1016/j.coastaleng.2019.04.004

Section 2.1 and 2.2 includes direct instructions written by Vos et al. (2019). Section 2.3 includes direct references from the DSAS user guide by Himmelstoss et al. (2018).
**WARNING**. The Coastsat code here has been altered, therefore the latest updates, issues and pull requests on Github may not be relevant to this workflow. The following changes have been made to the Coastsat module:


### Description

Satellite remote sensing can provide low-cost long-term shoreline data capable of resolving the temporal scales of interest to coastal scientists and engineers at sites where no in-situ field measurements are available. CoastSat_noc enables the non-expert user to extract shorelines from Landsat 7, Landsat 8 and Sentinel-2 images.
The shoreline detection algorithm implemented in CoastSat is optimised for sandy beach coastlines.   It combines a sub-pixel border segmentation and an image classification component, which refines the segmentation into four distinct categories such that the shoreline detection is specific to the sand/water interface.

The toolbox has three main functionalities:
- assisted retrieval from Google Earth Engine of all available satellite images spanning the user-defined region of interest and time period
- automated extraction of shorelines from all the selected images using a sub-pixel resolution technique
- intersection of the 2D shorelines with user-defined shore-normal transects


**If you like the repo put a star on it!**

## 1. Installation
To run the examples you will need to install the coastsat environment and activate Google Earth Engine API (instructions in the main [CoastSat toolbox] (https://github.com/kvos/CoastSat)).

## 2. Usage

An example of how to run the software in a Jupyter Notebook is provided in the repository (`StudyArea_shoreline.ipyNote:`). To run this, first activate your `coastsat` environment with `conda activate coastsat` (if not already active), and then type:

```
jupyter notebook
```

A web browser window will open. Point to the directory where you downloaded this repository and click on `StudyArea_shoreline.ipyNote:`.

A Jupyter Notebook combines formatted text and code. To run the code, place your cursor inside one of the code sections and click on the `run cell` button and progress forward.

### 3. Retrieval of the satellite images - Process shoreline mapping tool
To retrieve from the GEE server the available satellite images cropped around the user-defined region of coastline for the particular time period of interest, the following variables are required:

Task time = ~10 mins

1.  Open Jupyter Notebook (following instructions in ‘Usage’)
    1. Download ‘CoastSat-master_vSC’ and navigate to StudyArea_shoreline, copy, then rename StudyArea_shoreline.ipyNote:. E.g. ‘Tunisia_shoreline_2000_2020’
    2. Edit the following variables:
2. `Coordinate_List`- list of the coordinates of the region of interest (longitude/latitude pairs in WGS84) - see below for an example of how to extract ROI coordinates
3. `All_dates` - dates over which the images will be retrieved (e.g., `dates = ['2017-12-01', '2018-01-01']`)
4. `All_sats`: satellite missions to consider (e.g., `sat_list = ['L7', 'L8', 'S2']` for Landsat 7, 8 and Sentinel-2 collections)
5. `Sitename`: name of the site (this is the name of the subfolder where the images and other accompanying files will be stored)
6. `Settings`
    1. `Output_epsg` = Country-specific coordinate system (see https://epsg.io/)

### 3.1 Example of how to create a coordinate list at study site
This section demonstrates a simple way to create a coordinate list of a study area needed for the code above. It creates boxes around the coastline which are used as the limits to download a subset of satellite images. The coastline can be manually delineated if a small study area is here a country-scale analysis 
Task time = ~10 mins
```diff
!Note:: Google earth Engine has a limited image size of ~100km2 which can be downloaded at a single time.
!The use of smaller ROIs also reduces the volume of data downloaded.
```
1. Open ArcGIS map document and save in appropriate directory
2. First, we create a coastline of the study area. (See below if the study area is large – e.g. Country-scale).
3. In the geodatabase, create a feature class (right-click geodatabase) and select a line feature.
4. In the Edit window, select ‘create features’ and draw a coastline in the region of interest.
```diff
! **Note**: If the study site is large, you can convert administrative boundary polygons into lines
!           from the Humanitarian Data Exchange (https://data.humdata.org/).
! 1. Download the top-level (0) boundary.
! 2. Extract into directory with map document. Import into map document geodatabase.
! 4. Check line. Does it fit the shoreline roughly (within ~800m)?
!    If not, retrieve boundary from different source or draw a rough shoreline.
! 4. Convert the boundary polygon to a polyline. 
!     1. Feature to line
!     2. Input = Top-level admin boundary
!     3. Output = Geodatabase
! 5. Use split tool to remove inland lines and save single coastal line]
```
5. Create regions of interest (ROI) boxes along coast.

    1. Strip Map Index Features
    2. Length along line = 11km
    3. Perpendicular to the line = 2
    4. Overlap = 0

6. Zoom to individual ROIs to ensure that all possible shorelines are contained within the box.
    1. Edit ROIs using ‘edit vertices’ or ‘reshape’ tools.

### 3.2 Extract Coordinates
Once the ROIs have been established, we need to extract the coordinates to a list in order to run the modified coastsat script. The first of four ArcGIS models is used. These models combine multiple ArcGIS functions in a clear chain structure that can be viewed in the edit window (Right click model in toolbox > Edit). The model can be run by double clicking the nuame in the catalog pane as well as in the edit window which can be more reliable if a process fails in the geoprocessing pane. A breakdown of the processes in the models is given in the below for clarity, understanding and scrutiny, with the hope to make this process full open sourced in the future.

<details>
           <summary>Model Breakdown (**Extract Coordinates**)</summary>
           <p>

1.	Extract coordinates of outer boundaries of the ROIs
    1. Feature vertices to points
    2. Input = ROIs
    3. Point type  = All vertices
2. Find XY position
    1. Add fields (XY) (x2)
    2. Input = ROI points
    3. Field type = Double
    4.Calculate geometry (right click field name)
        1. x = x-point coordinate
        2. y = y-point coordinate
        3. Use decimal degree format
3. Find max/min XY coordinates
    1. Dissolve
    2. Dissolve Field = Pg_no
    3. Statistics fields
        1. max x
        2. min x
        3. max y
        4. min y
    4. Create multipart features – checked
    5. Unsplit lines – unchecked
4.	Combine coordinates
    1. Add field (‘xytext’)
    2. Calculate field
    3. Field name = xytext
    4. xytext = "([[" + str(!MAX_x!) + "," + str(!MAX_y!) + "],[" + str(!MAX_x!) + "," + str(!MIN_y!) + "],[" + str(!MIN_x!) + "," + str(!MAX_y!) + "],["  + str(!MIN_x!) + "," + str(!MIN_y!) + "]]),"
5.	Export attribute table
    1. Right click layer in contents panel
    2. Export table
    3. Save in directory as .csv
    4. Output fields = xytext
</p>
         </details>

1. In map document, in Catalog window. Under toolboxes > right click > Add Toolbox > navigate to CoastSat-master_vSC > ShorelineChangeTools > ShorelineChange.tbx
2. Double click ‘Extract Coordinates’ to open processor
3. Input ROIs and the output location folder (e.g. ‘CoastSat-master_vSC’)
4. Run.

Once the table is saved the coordinates are in a table format, but we need a list…
5. Open in excel. Delete Column OBJECTID and top row, then click save
6. Re-open the saved file in a text editor (notepad/notepad++)
7. Find and Replace.
8. Find ‘ ” ’. Replace ‘ ‘.
9. Find ‘ ]]), ’. Replace ‘ ]]),\ ‘.
10. Remove \ symbol on the last coordinate

### 3.3	Begin processing
1. Open Jupyter Notebook (following instructions in ‘Usage’) if not already open and navigate to `StudyArea_shoreline.ipyNote:`
2. Hit run on the initial settings and the edited code after “1. Retrieval of the images from GEE”
3. After a few minutes of processing, navigate to the data folder in Coastsat_master. Find the output. geojson file and export to software to ensure the correct output is viewed.

```diff
! Note: The following shows a common error:
!   ‘HTTPError: HTTP Error 500: Internal Server Error’.
! This is due to a network error with Google Earth Engine which is caused by a timeout using
! the getDownloadURL function – looking into this issue. This error can be frequent, and occurs
! less frequently during evenings (GMT). If this occurs:
!   i.	Find number of folders in C:\Coastsat_master\data. E.g. Tunisia_ 27.
!   ii.	Remove last folder (as shorelines haven’t been created for this folder). I.e. Delete Tunisia_27
!   iii. Cut and paste the first 26 ROIs, and place them elsewhere (I keep them in code and comment them out)
!   iv.	In the For loop, change sitename to “Sitename = counter + 27 “
!   v.	Re-run model. The model will continue to process shorelines from the previous point.
!   vi.	Repeat this process if it occurs again. Maintain continuous file number to prevent overwrite. 
```


## Potential Errors / Solutions

    **EEException: Image.select: Parameter 'input' is required.**

This error is caused by a lack of images in the Landsat/Sentinel collection for the Co-registration process, after refining the cloud cover and date (i.e. within a 2 month window) . Simply change the date into another year, or raise the maximum cloud cover % in the SDS_download file in the Coastsat folder. Change this under the function ‘Landsat_coregistration’ for the variable ‘dated’. For example, change “L8_reference = .filterDate('2017-01-01', '2018-01-01')” to “L8_reference = .filterDate('2018-01-01', '2019-01-01')” AND do the same for Sentinel-2 9 lines below.

Still having a problem? Post an issue in the [Issues page](https://github.com/sac3g15/coastsat_noc/issues) (please do not email).


## References

1. Vos K., Harley M.D., Splinter K.D., Simmons J.A., Turner I.L. (2019). Sub-annual to multi-decadal shoreline variability from publicly available satellite imagery. *Coastal Engineering*. 150, 160–174. https://doi.org/10.1016/j.coastaleng.2019.04.004

2. Vos K., Splinter K.D.,Harley M.D., Simmons J.A., Turner I.L. (2019). CoastSat: a Google Earth Engine-enabled Python toolkit to extract shorelines from publicly available satellite imagery. *Environmental Modelling and Software*. 122, 104528. https://doi.org/10.1016/j.envsoft.2019.104528

3. Training dataset used for pixel classification in CoastSat: https://doi.org/10.5281/zenodo.3334147
