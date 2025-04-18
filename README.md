# Geospatial interpolation of yield maps Jolanda Di Savoia plains
For this assignment, I have decided to use NDVI (Normalized Difference Vegetation Index) to investigate the state of vegetation in the field. It is one of the most important and widely used indices for farmers to forecast and monitor yield.
## Data Frame:
The Dataframe is in GPKG format. GPKG is an open format for transferring geospatial information. It is an independent, self-describing, compact format. The GeoPandas library is used to read and write GPKG format files.
The prefix 'r' is used before a string in the code to indicate that it’s a raw string. Raw strings clearly differentiate backslashes () and escape characters (used for things like \n for a new line). This practice is helpful to avoid issues when using Windows file paths, as they rely heavily on backslashes.
## Data 
•	Longitudin/Latitudine geo-references of the acquisitions, CSR is in EPSG:4326

•	RESAKG the amount of the product harvested acquired, in kg.

•	VELOCITA, the speed of the harvester, in km/h.

•	AREA, the area harvested since the last acquisition, in mq.

•	UMIDITA, the average relative humidity of the product.
## Missing Values
The column UMIDITA had 37 missing values.
## Steps for KNN Imputation to replace missing values
•	Defining the target column, which is UMIDITA. 

•	Finding missing values before applying the procedure. 

•	Extracting X and Y coordinates from the geometry column, which has a projected coordinate system. 

•	Defining predictor columns. 

•	Defining the columns that will be used for imputation, containing both the target and predictor columns. 

•	Creating a temporary Dataframe with these columns. 

•	Standardizing all columns involved because KNN relies on distance between points. Scaling prevents features with inherently large numerical values (like coordinates in meters) from unfairly dominating the definition of "similarity" in distance-based algorithms like KNN, ensuring all chosen features play a balanced role in finding the true nearest neighbours.

•	Applying the imputation algorithm using the KNNImputer class from scikit-learn's imputation module (sklearn.impute). The fit_transform method of scikit-learn imputers (like KNNImputer) returns a NumPy array. 

•	Reversing the scaling after imputation using inverse_transform to get the final, imputed dataset (imputed_df) back into its original, interpretable units.

•	Updating the original Dataframe.
## Spatial Interpolation Yield rate
•	Calculating yield rate.

•	Defining the target variable ('yield_rate') for spatial interpolation from the GeoDataFrame (gdf). 

•	Extracting the X and Y coordinates from the GeoDataFrame's geometry into a NumPy array (points). 

•	Extracting the corresponding data values for the target variable into a NumPy array (values). 

•	Calculating the overall spatial extent (minimum/maximum X and Y coordinates, i.e., bounding box) of the input data points (gdf.total_bounds). 

•	Defining the desired spatial resolution (pixel size, e.g., 2.0 meters) for the output interpolated grid. 

•	Generating 1D NumPy arrays of regularly spaced X and Y coordinates (grid_x_coords, grid_y_coords) covering the data extent at the specified resolution. 

•	Creating 2D NumPy arrays (grid_x, grid_y) representing the coordinate matrices for every node in the output grid using np.meshgrid. 

•	Importing the griddata function from the scipy.interpolate library. 

•	Calling the griddata function, passing it the points, values, grid_x, grid_y variables, and specifying method='linear'. 

•	Assigning the result of the griddata function (the 2D array of interpolated values) to the variable grid_z. 

** Exporting Interpolated Data to Raster File**

•	Importing necessary libraries (rasterio and numpy). To gain tools for working with raster files (GeoTIFFs) and numerical arrays.

•	Defining the file path (output_raster_path) for the output raster. To specify where the final map image will be saved.

•	Calculating the affine transformation matrix (transform). To georeference the raster by defining how pixel coordinates (row, column) map to real-world geographic coordinates (X, Y), using the grid's origin and resolution.

•	Defining the Coordinate Reference System (output_crs, e.g., "EPSG:32633"). To specify the map projection for the output raster, ensuring it aligns correctly with other geospatial data.

•	Opening a new file at the specified output_raster_path in write mode using rasterio.open. To create the GeoTIFF file and prepare it for writing data and metadata.

•	Configuring the output GeoTIFF's structure and metadata within rasterio.open. To define the essential properties of the raster file, including: 

o	Setting the driver to 'GTiff' for GeoTIFF format.

o	Defining raster height and width based on the interpolated data array (grid_z).

o	Setting the number of bands (count=1) for the single variable (yield rate).

o	Specifying the data type (dtype) for storing pixel values.

o	Assigning the Coordinate Reference System (crs).

o	Assigning the georeferencing transform.

o	Defining a nodata value (np.nan) to represent missing data pixels.

•	Writing the 2D NumPy array of interpolated data (grid_z) to the first band of the new GeoTIFF file. To store the calculated yield rate values as the actual pixel data in the map.

•	Reads the specified GeoTIFF raster file (interpolated_yield_map.tif), loads its data while handling NoData values, and then visualizes this data as a properly georeferenced map using Matplotlib.

## Average Vegetation Index Map 
•	Importing a wide range of libraries essential for geospatial data handling (geopandas, xarray, rioxarray, rasterio), cloud data access (pystac_client, planetary_computer), data stacking (stackstac), parallel processing (dask), and general data manipulation (numpy, pandas, matplotlib). 

•	Defining core parameters for the analysis: The Area of Interest (AOI) using bounding box coordinates in a projected CRS (EPSG:32633).

o	The target time period (e.g., '2022-04-01' to '2022-08-31').

o	The desired output spatial resolution (e.g., 2.0 meters).

o	The specific vegetation index to calculate ('NDVI').

•	Converting the projected AOI bounds to geographic coordinates (EPSG:4326), as required for searching the STAC catalog. 

•	Attempting to initialize and start a Dask local cluster and client to enable parallel processing for potentially faster computation. 

•	Connecting to the Microsoft Planetary Computer's STAC (SpatioTemporal Asset Catalog) API to search for satellite imagery. 

•	Searching the STAC catalog for Sentinel-2 L2A satellite data based on the defined AOI, time period, and a filter for low cloud cover (< 25%). 

•	Loading the required spectral bands (Red - B04, Near-Infrared - B08) from the discovered satellite scenes into an xarray data cube using stackstac. This process aligns the data to the target CRS and resolution, initially keeping the raw integer pixel values without rescaling. 

•	Selecting the raw integer data for the Red and NIR bands from the data cube.

•	Manually scaling the raw integer values of the Red and NIR bands by the appropriate factor (0.0001) to convert them into surface reflectance values (floating-point numbers between 0 and 1). 

•	Calculating the Normalized Difference Vegetation Index (NDVI) for each pixel at each time step using the standard formula (NIR - Red) / (NIR + Red) on the scaled reflectance values. 

•	Clipping the calculated NDVI values to ensure they fall within the valid theoretical range of -1 to 1. 

•	Calculating the temporal median NDVI value for each pixel across all the time steps in the data cube. This creates a single representative NDVI map for the period, and. compute () triggers the actual calculation (potentially using Dask). 

•	Defining the file path for the output GeoTIFF raster that will store the average NDVI map. 

•	Ensuring the resulting average NDVI xarray DataArray has the correct Coordinate Reference System metadata attached using rioxarray.

•	Saving the calculated average NDVI map as a GeoTIFF file using rioxarray, applying compression and defining how NoData values should be handled.

•	Implementing error handling (try...except) to manage potential issues like missing libraries or runtime errors during processing. 

•	Ensuring the Dask client and cluster are properly closed (finally block) after the script finishes or encounters an error, freeing up resources.

## Correlation Analysis between Yield Map and Average NDVI Map
•	Compare Raster Properties: Open the interpolated yield map and the average NDVI map, then print and compare their key geospatial metadata (CRS, Bounds, Dimensions, Resolution, Transform) to check if they align. 

•	Align Rasters: If the rasters don't align (or to ensure alignment), use the yield map as a reference to resample/reproject the average NDVI map onto the exact same grid, saving the result as a new, aligned NDVI map file (avg_ndvi_map_aligned.tif). 

•	Read Aligned Data: Load the pixel data from both the yield map and the newly aligned average NDVI map, using masked arrays to handle NoData values appropriately. 

•	Extract Common Valid Data: Identify the pixels where both maps have valid (non-NoData) data points and extract these corresponding pairs of yield and NDVI values into flattened arrays. 

•	Calculate Correlation: Compute the Pearson correlation coefficient and its p-value between the extracted pairs of valid yield and NDVI pixel values to quantify the linear relationship. 

•	Analyze and Report Results: Print the calculated correlation coefficient and p-value, along with a basic interpretation of the correlation strength and its statistical significance.  

•	Visualize Relationship: Create a scatter plot to visually represent the relationship between the aligned NDVI values (x-axis) and the yield values (y-axis) for the common valid pixels, including a linear regression line fit to the data.

## Interpretation of Correlation Results

The results show statistically significant (meaning unlikely due to chance) but very weak negative linear relationship between the average NDVI and the estimated yield rate for the pixels analyzed. The weakness of this linear relationship (r close to 0) is why the linear regression line appears horizontal on the scatter plot, indicating that NDVI is a poor linear predictor of yield rate in this specific dataset and context.









