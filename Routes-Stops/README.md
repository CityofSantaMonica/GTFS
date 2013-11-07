# Routes and Stops

- The stops file is derived from stops.txt in the GTFS data.
- The routes file is derived from a different system. However, it has been attached to a copy of routes.txt in the GTFS data.
- There are some inaccuracies in the route vectors (they do not follow street center lines precisely).

## How you can help 

1. Fork this repository
2. Clean up the data
  1. Load stops layer (accurate stop locations)
  2. Load a base map (e.g. OpenLayers plugin in QGIS for Google Streets or OpenStreetMap)
  3. Load CAMS (optional) for street centerlines
  4. Load routes layer and save in editable format (e.g. Shapefile)
  5. Correct route lines where they divert from street centerline
  6. Save corrected routes layer in GeoJSON or Shapefile format
3. Submit a pull request

## Additional Resources

- [QGIS](http://www.qgis.org/) - open source GIS system
- [CAMS](http://egis3.lacounty.gov/dataportal/2013/09/26/2011-la-county-street-centerline-street-address-file/) - Los Angeles County street centerline geodatabase (if using QGIS you will need to install the gdal-filegdb library)
