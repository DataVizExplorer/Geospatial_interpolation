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

