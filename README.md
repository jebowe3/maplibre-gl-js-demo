# MapLibre GL JS and Big Vector Data
How to map large geojson data sources on the web using vector tiles and MapLibre GL JS, an open-source version of Mapbox GL JS.

## Contents

- [Introduction](#introduction)
- [Download the California Wildfires Data](#download-the-california-wildfires-data)
- [Prepare the HTML Boilerplate](#prepare-the-html-boilerplate)
- [Add a Title Bar and Time Slider](#add-a-title-bar-and-time-slider)
- [Add Vector Base Tiles](#add-vector-base-tiles)

## Introduction
Occasionally you have a large geojson file you want to include in a web map application, but it slows down your map even after implementing various file simplification techniques. Instead of trying all forms of file resizing to fit your data in a Leaflet map, you can convert your geojson file into vector tiles and build an interactive map using the MapLibre GL JS library with all the same functionality. The following is a step-by-step walkthrough of two methods for incorporating large geojson files as vector tiles in MapLibre GL JS using [more than a century of California wildfire data](https://services.gis.ca.gov/arcgis/rest/services/Environment/Wildfires/MapServer). You will see how to process these data as vector tiles and then filter them by year using a time slider. You will also learn how to add popup content to allow users to query the wildfire polygons.

## Download the California Wildfires Data
First, [download this repository](https://github.com/jebowe3/maplibre-gl-js-demo/archive/refs/heads/main.zip). Inside is a folder called "map". All of the data and code are within, and you can simply look at this for reference or build your own map from the ground up using the instructions that follow.

Create a folder structure as follows for your project, creating an index.html file at the root and a subdirectory called "data" to hold the geojson file.

![Initial Project Folder Structure](images/initial-proj-structure.PNG)  
**Figure 01**. How to Organize Your Project.

Inside the data folder within the map folder, grab the calWildfires.json file and add this to your own project folder in your "data" subdirectory. You will notice that this file is a little more than 50 MB and quite a bit larger than the maximum file size recommended for Leaflet maps.

## Prepare the HTML Boilerplate

Now, open your blank index.html file and set it up to handle vector tiles with MapLibre.

```HTML
<!DOCTYPE html>
<html>

<head>
  <meta charset='utf-8' />
  <title>California Wildfires</title>
  <meta name='viewport' content='initial-scale=1,maximum-scale=1,user-scalable=no' />

  <!-- maplibre-gl.css here -->
  <link href='https://unpkg.com/maplibre-gl@2.1.7/dist/maplibre-gl.css' rel='stylesheet' />
  <!-- Rubik font -->
  <link href="https://fonts.googleapis.com/css?family=Rubik&display=swap" rel="stylesheet">

  <style>
    /* basic css to style the viewer goes here */
    body {
      margin: 0;
      padding: 0;
    }

    #map {
      position: absolute;
      top: 0;
      bottom: 0;
      width: 100%;
    }

  </style>

</head>

<body>

  <!-- id the map div -->
  <div id='map'></div>

  <!-- maplibre-gl.js library -->
  <script src='https://unpkg.com/maplibre-gl@2.1.7/dist/maplibre-gl.js'></script>

  <script>
    // javascript goes here
  </script>

</body>  
```

The code above references our necessary MapLibre css and javascript libraries, adds Rubik font from Google, defines the map div, and styles the map and body with some basic css.

## Add a Title Bar and Time Slider

Before going too much further, we should add a title and time slider element so that our map is prepared to handle this filtration later. Within the body tags, and just beneath the code identifying the map div, add and id another div for the title and time slider.

```html
  <!-- id the map div -->
  <div id='map'></div>

  <!-- title and time slider -->
  <div class='session' id='sliderbar'>
    <!-- the shown title of the map -->
    <h1>California Wildfires, 1878 - 2010</h1>
    <div class='container'>
      <!-- the selected year -->
      <h2>Year: <label id='active-year'>2010</label></h2>
    </div>
    <!-- the slider has a range from 1878 to 2010, intervals of 1 year, and an initial value of 2010 -->
    <input id='slider' class='row' type='range' min='1878' max='2010' step='1' value='2010' />
  </div>
```

Now we need to style this content with some css within the style tags.

```HTML
  <style>
    /* basic css to style the viewer goes here */
    body {
      margin: 0;
      padding: 0;
    }

    #map {
      position: absolute;
      top: 0;
      bottom: 0;
      width: 100%;
    }

    /* set styles for the container holding the title and slider */
    .session {
      position: absolute;
      z-index: 1;
      background-color: rgba(255, 255, 255, 0.5);
      border-radius: 3px;
      box-shadow: 0px 0px 0px 1px rgba(0, 0, 0, 0.3);
      top: 10px;
      left: 10px;
      padding: 5px;
    }

    /* set styles for the container holding the year legend */
    .container {
      display: table;
      width: 100%;
      margin: 0;
    }

    /* set font styles for the title */
    h1 {
      font-size: 20px;
      font-family: 'Rubik', sans-serif;
      padding-bottom: 0px;
      padding-top: 0px;
      font-weight: normal;
    }

    /* set font styles for the identified year */
    h2 {
      cursor: pointer;
      font-size: 14px;
      font-family: 'Rubik', sans-serif;
      padding-bottom: 0px;
      padding-top: 0px;
      font-weight: normal;
      width: 50%;
      text-align: left;
      display: table-cell;
    }

    /* define the slider width and change the cursor to a pointer on hover */
    #slider {
      cursor: pointer;
      width: 275px;
    }

  </style>
```

Upon refresh, the result should look like this:

![The Title and Time Slider](images/title.PNG)  
**Figure 02**. Title and Time Slider.

## Add Vector Base Tiles

Now that we have a title, we should add a base map to our blank map screen for some spatial context. Within the script tags, insert the following javascript.

```html
  <script>
    // javascript goes here

    // your maptiler key
    const key = 'X79IRuovndFj8moKWjAt'

    // define the style.json for the vector tiles here
    const style = 'https://api.maptiler.com/maps/darkmatter/style.json?key=' + key

    // define the map
    const map = new maplibregl.Map({
      container: 'map',
      bounds: [[-124.48,32.53],[-114.13,42.01]], // the coordinate boundaries of California
      maxZoom: 16,
      style: style
    });
  </script>
```

Unlike raster tile providers, most vector tiles currently sit behind a paywall or require a personal api key to use. Fortunately, Maptiler offers open-source vector base tiles, but you will need to [sign up for a free account](https://cloud.maptiler.com/maps/). Once you do, you can find your api key by clicking your "Account" tab. The nice thing about this service is that you can add new keys that only work for certain domains. This way, no one else can steal your key from GitHub to use for a map hosted elsewhere.

In the code above, we have added maptiler's darkmatter style. If you are successful, the map should now look like this:

![Dark Matter Vector Tiles](images/cal-dm.PNG)  
**Figure 03**. Dark Matter Vector Tiles.
