# Routes and Stops

- The stops and route files are derived from the GTFS data using QGIS.
 
## Process

1. Install QGIS from http://www.qgis.org
2. Install plugins
  - QSpatiaLite
  - OpenLayers Plugin (optional - allows viewing Google Maps as basemap underneath
3. extract GTFS zip file into folder
4. Open QGIS
5. (menu) "Plugins" | "OpenLayers Plugin" | "Add Google Streets layer" (optional)
6. Save QGIS project "gtfs.qgs" into same folder where you unzipped GTFS zip content
7. (menu) "Layer" | "Add Delimited Text Layer..."
  - you'll do this for the following files in the folder where you unzipped GTFS zip content
    - routes.txt (select "Trim Fields" and "No geometry (attribute only table)")
    - trips.txt (select "Trim Fields" and "No geometry (attribute only table)")
    - stop_times.txt (select "Trim Fields" and "No geometry (attribute only table)")
    - stops.txt (select "Trim Fields" and "Point coordinates")
8. (menu) "Database" | "SpatiaLite" | "QSpatiaLite"
  - This opens the QspatiaLite database manager
  - NOTE: The manager interface may look different from version to version so you'll need to mouse over the toolbar at the top to verify actions
9. "Open/Create new database"
  - save the database gtfs.sqlite in the same folder where you unzipped GTFS zip content
10. "Import QGIS Layer"
  - (Select All)
  - this will create four tables in the SpatiaLite database with the same names, "routes", "stop_times", "stops" (a point layer), and "trips"
11. Right+Click on the following four tables in the QspatiaLite manager and select "Load in QGIS"
  1. routes
  2. trips
  3. stop_times
  4. stops
12. Remove the four layers added in stop #7
