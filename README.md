# MapLibre GL JS and Big Vector Data
How to map large geojson data sources on the web using vector tiles and MapLibre GL JS, an open-source version of Mapbox GL JS.

## Contents

- [Introduction](#introduction)

## Introduction
Occasionally you have a large geojson file you want to include in a web map application, but it slows down your map even after implementing various file simplification techniques. Instead of trying all forms of file resizing to fit your data in a Leaflet map, you can convert your geojson file into vector tiles and build an interactive map using the MapLibre GL JS library with all the same functionality. The following is a step-by-step walkthrough of two methods for incorporating large geojson files as vector tiles in MapLibre GL JS using [more than a century of California wildfire data](https://services.gis.ca.gov/arcgis/rest/services/Environment/Wildfires/MapServer). You will see how to process these data as vector tiles and then filter them by year using a time slider. You will also learn how to add popup content to allow users to query the wildfire polygons.

## Download the California Wildfires Data
First, [download this repository](https://github.com/jebowe3/maplibre-gl-js-demo/archive/refs/heads/main.zip). Inside is a folder called "map". All of the data and code are within, and you can simply look at this for reference or build your own map from the ground up using the instructions that follow.

Create a folder structure as follows for your project, creating an index.html file at the root and a subdirectory called "data" to hold the geojson file.

![Initial Project Folder Structure](images/initial-proj-structure.PNG)  
**Figure 01**. How to Organize Your Project.

Inside the data folder within the map folder, grab the calWildfires.json file and add this to your own project folder in your "data" subdirectory. You will notice that this file is a little more than 50 MB and quite a bit larger than the maximum file size recommended for Leaflet maps.

Now, open your blank index.html file and set it up to handle vector tiles with MapLibre.
