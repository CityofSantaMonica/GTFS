# Routes and Stops

- The stops and route files are derived from the GTFS data using QGIS.
 
## Process

This process allows you to segregate route data by actual stop patterns, regardless of trip_headign issues (e.g. extra spaces, etc.). It uses database features in SpatiaLite to create views and a distinct route signature using SQLITE's "group_concat" clause.

- Install QGIS from http://www.qgis.org
- Install plugins
  - QSpatiaLite
  - OpenLayers Plugin (optional - allows viewing Google Maps as basemap underneath

  
1. extract GTFS zip file into folder
2. Open QGIS
  - NOTE: QGIS and the QspatiaLite database manager run side by side in this process
3. (QGIS) "Plugins" | "OpenLayers Plugin" | "Add Google Streets layer" (optional)
4. Save QGIS project "gtfs.qgs" into same folder where you unzipped GTFS zip content
5. (QGIS) "Layer" | "Add Delimited Text Layer..."
  - you'll do this for the following files in the folder where you unzipped GTFS zip content
    - routes.txt (select "Trim Fields" and "No geometry (attribute only table)")
    - trips.txt (select "Trim Fields" and "No geometry (attribute only table)")
    - stop_times.txt (select "Trim Fields" and "No geometry (attribute only table)")
    - stops.txt (select "Trim Fields" and "Point coordinates" then CRS "4326")
6. (QGIS) "Database" | "SpatiaLite" | "QSpatiaLite"
  - This opens the QspatiaLite database manager
  - NOTE: The manager interface may look different from version to version so you'll need to mouse over the toolbar at the top to verify actions
7. (QspatiaLite) "Open/Create new database"
  - save the database gtfs.sqlite in the same folder where you unzipped GTFS zip content
8. (QspatiaLite) "Import QGIS Layer"
  - (Select All)
  - this will create four tables in the SpatiaLite database with the same names, "routes", "stop_times", "stops" (a point layer), and "trips"
10. (QGIS) Remove the four layers added in stop #7
11. (QSpatiaLite) under the "SQL" tab:
  - execute the following query:
  ```SQL
CREATE TABLE 'trip_stops' AS
SELECT trip_id,
  (SELECT group_concat('stop_times' . 'stop_id')
  FROM 'stop_times'
  WHERE 'stop_times' . 'trip_id' = 'trips' . 'trip_id'
  ORDER BY 'stop_times' . 'stop_sequence') AS 'stop_ids'
FROM 'trips'
  ```
12. (QSpatiaLite) under the "SQL" tab:
  - execute the following query:
  ```SQL
CREATE TABLE 'unique_trips' AS
SELECT MIN('trip_stops' . 'trip_id') AS trip_id, 'trip_stops' . 'stop_ids'
FROM 'trip_stops'
GROUP BY 'trip_stops' . 'stop_ids'
  ```
13. (QSpatiaLite) under the "SQL" tab:
  - execute the following query:
  ```SQL
CREATE TABLE 'distinct_routes' AS
SELECT 'routes' . 'route_short_name', 'routes' . 'route_long_name', 'trips' . 'direction_id', 'trips' . 'trip_headsign', 'routes' . 'route_short_name' || ' - ' || 'routes' . 'route_long_name' || ' - ' || 'trips' . 'direction_id' || ' - ' || coalesce('trips' . 'trip_headsign', ' ') AS headsign, 'unique_trips' . 'trip_id'
FROM 'routes', 'trips', 'unique_trips'
WHERE 'routes' . 'route_id' = 'trips' . 'route_id' AND 'trips' . 'trip_id' = 'unique_trips' . 'trip_id'
ORDER BY 'trips' . 'trip_headsign'
  ```
14. (QSpatiaLite) under the "SQL" tab:
  - select the dropdown option "Create Spatial Table & Load in QGIS"
  - set the Table Name to "distinct_route_stops"
  - set the Geometry field to "Geometry"
  - execute the following query:
  ```SQL
SELECT 'stops' . 'Geometry' AS Geometry, 'distinct_routes' . 'route_short_name', 'distinct_routes' . 'route_long_name', 'distinct_routes' . 'direction_id', 'distinct_routes' . 'trip_headsign', 'distinct_routes' . 'headsign', 'distinct_routes' . 'trip_id', 'stop_times' . 'stop_sequence', 'stops' . 'stop_name'
FROM 'distinct_routes', 'stop_times', 'stops'
WHERE 'distinct_routes' . 'trip_id' = 'stop_times' . 'trip_id' AND 'stop_times' . 'stop_id' = 'stops' . 'stop_id'
ORDER BY 'distinct_routes' . 'headsign', 'stop_times' . 'stop_sequence'
  ```
16. (QGIS) Right+Click on the "distinct_route_stops" layer and select "Properties"
17. (QGIS) Click the "Style" tab and change "Single Symbol" to "Categorized"
18. (QGIS) for "Column" select "headsign" and click the "Classify" button at the bottom
19. (QspatiaLite) Delete Link to Database
