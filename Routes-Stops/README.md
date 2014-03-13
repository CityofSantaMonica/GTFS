# Routes and Stops

- The stops and route files are derived from the GTFS data using QGIS.
 
## Process

1. Install QGIS from http://www.qgis.org
2. Install plugins
  - QSpatiaLite
  - OpenLayers Plugin (optional - allows viewing Google Maps as basemap underneath
3. extract GTFS zip file into folder
4. Open QGIS
  - NOTE: QGIS and the QspatiaLite database manager run side by side in this process
5. (QGIS) "Plugins" | "OpenLayers Plugin" | "Add Google Streets layer" (optional)
6. Save QGIS project "gtfs.qgs" into same folder where you unzipped GTFS zip content
7. (QGIS) "Layer" | "Add Delimited Text Layer..."
  - you'll do this for the following files in the folder where you unzipped GTFS zip content
    - routes.txt (select "Trim Fields" and "No geometry (attribute only table)")
    - trips.txt (select "Trim Fields" and "No geometry (attribute only table)")
    - stop_times.txt (select "Trim Fields" and "No geometry (attribute only table)")
    - stops.txt (select "Trim Fields" and "Point coordinates")
8. (QGIS) "Database" | "SpatiaLite" | "QSpatiaLite"
  - This opens the QspatiaLite database manager
  - NOTE: The manager interface may look different from version to version so you'll need to mouse over the toolbar at the top to verify actions
9. (QspatiaLite) "Open/Create new database"
  - save the database gtfs.sqlite in the same folder where you unzipped GTFS zip content
10. (QspatiaLite) "Import QGIS Layer"
  - (Select All)
  - this will create four tables in the SpatiaLite database with the same names, "routes", "stop_times", "stops" (a point layer), and "trips"
11. (QspatiaLite) Right+Click on the following four tables in the QspatiaLite manager and select "Load in QGIS"
  1. routes
  2. trips
  3. stop_times
  4. stops
12. (QGIS) Remove the four layers added in stop #7
13. (QSpatiaLite) under the "SQL" tab:
  - select the dropdown option "Create View & Load in QGIS"
  - set the Table Name to "trip_stops"
  - execute the following query:
  ```SQL
SELECT trip_id,
  (SELECT group_concat('stop_times' . 'stop_id')
  FROM 'stop_times'
  WHERE 'stop_times' . 'trip_id' = 'trips' . 'trip_id'
  ORDER BY 'stop_times' . 'stop_sequence') AS 'stop_ids'
FROM 'trips'
  ```
14. (QSpatiaLite) under the "SQL" tab:
  - select the dropdown option "Create View & Load in QGIS"
  - set the Table Name to "unique_trips"
  - execute the following query:
  ```SQL
SELECT MIN('trip_stops' . 'trip_id') AS trip_id, 'trip_stops' . 'stop_ids'
FROM 'trip_stops'
GROUP BY 'trip_stops' . 'stop_ids'
  ```
15. (QSpatiaLite) under the "SQL" tab:
  - select the dropdown option "Create View & Load in QGIS"
  - set the Table Name to "distinct_routes"
  - execute the following query:
  ```SQL
SELECT 'routes' . 'route_short_name', 'routes' . 'route_long_name', 'trips' . 'direction_id', 'trips' . 'trip_headsign', 'routes' . 'route_short_name' || ' - ' || 'routes' . 'route_long_name' || ' - ' || 'trips' . 'direction_id' || ' - ' || coalesce('trips' . 'trip_headsign', ' ') AS headsign, 'unique_trips' . 'trip_id'
FROM 'routes', 'trips', 'unique_trips'
WHERE 'routes' . 'route_id' = 'trips' . 'route_id' AND 'trips' . 'trip_id' = 'unique_trips' . 'trip_id'
ORDER BY 'trips' . 'trip_headsign'
  ```
16.
