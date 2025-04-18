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

** Exporting Interpolated Data to Raster File

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





