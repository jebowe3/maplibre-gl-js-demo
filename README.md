# MapLibre GL JS and Big Vector Data
How to map large geojson data sources on the web using vector tiles and MapLibre GL JS, an open-source version of Mapbox GL JS.

## Contents

- [Introduction](#introduction)
- [Download the California Wildfires Data](#download-the-california-wildfires-data)
- [Prepare the HTML Boilerplate](#prepare-the-html-boilerplate)
- [Add a Title Bar and Time Slider](#add-a-title-bar-and-time-slider)
- [Add Vector Base Tiles](#add-vector-base-tiles)
- [Adding GeoJSON Files as Vector Tiles On the Fly](#adding-geojson-files-as-vector-tiles-on-the-fly)
- [Display Popup on Mouse Hover](#display-popup-on-mouse-hover)
- [Filter Geojson Data with the Time Slider](#filter-geojson-data-with-the-time-slider)

## Introduction
Occasionally you have a large geojson file you want to include in a web map application, but it slows down your map even after implementing various file simplification techniques. Instead of trying all forms of file resizing to fit your data in a Leaflet map, you can convert your geojson file into vector tiles and build an interactive map using the MapLibre GL JS library with all the same functionality. The following is a step-by-step walkthrough of two methods for incorporating large geojson files as vector tiles in MapLibre GL JS using [more than a century of California wildfire data](https://services.gis.ca.gov/arcgis/rest/services/Environment/Wildfires/MapServer). You will see how to process these data as vector tiles and then filter them by year using a time slider. You will also learn how to add popup content to allow users to query the wildfire polygons.

## Download the California Wildfires Data
First, [download this repository](https://github.com/jebowe3/maplibre-gl-js-demo/archive/refs/heads/main.zip). Inside is a folder called "map". All of the data and code are within, and you can simply look at this for reference or build your own map from the ground up using the instructions that follow.

Create a folder structure as follows for your project, creating an index.html file at the root and a subdirectory called "data" to hold the geojson file.

![Initial Project Folder Structure](images/initial-proj-structure.PNG)  
**Figure 01**. How to Organize Your Project.

Inside the data folder within the map folder, grab the calWildfires.geojson file and add this to your own project folder in your "data" subdirectory. You will notice that this file is a little more than 40 MB and quite a bit larger than the maximum file size recommended for Leaflet maps.

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

We can build on this a little and add a few preliminary interactive controls.

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

    // add zoom and rotation controls to the map.
    map.addControl(new maplibregl.NavigationControl());

    // add a scale bar
    map.addControl(new maplibregl.ScaleControl({
      position: 'bottom-left',
      unit: 'imperial' // you can change from imperial to metric, if desired
    }));

  </script>
```

After saving and refreshing the map, you should now notice a scale bar in the bottom left corner and a zoom and rotation control in the upper right corner.

## Adding GeoJSON Files as Vector Tiles On the Fly

The first method to add geojson as vector tiles uses MapLibre GL JS to handle the processing on the fly. This is the easiest method, but is slower and less responsive in the browser than the second method I will demonstrate. Still, it is pretty good at handling large files.

First, let's add D3.js to our available scripting libraries underneath the address for the MapLibre GL JS library. This will help simplify access to our data.

```HTML
  <!-- maplibre-gl.js library -->
  <script src='https://unpkg.com/maplibre-gl@2.1.7/dist/maplibre-gl.js'></script>

  <!-- d3 js -->
  <script src="https://d3js.org/d3.v6.min.js"></script>
```

Next, just after the javascript adding the scale bar, signal to the application to call a function when the map loads to bring in our geojson data.

```js
    // add a scale bar
    map.addControl(new maplibregl.ScaleControl({
      position: 'bottom-left',
      unit: 'imperial' // you can change from imperial to metric, if desired
    }));

    // when the map loads, call a function
    map.on('load', () => {

      // define incoming geojson data
      const calwf = d3.json('./data/calWildfires.geojson');

      // promise to wait until geojson files are loaded
      Promise.all([calwf]).then((data) => {

        // define the data according to its order in object array
        const wildfires = data[0];

        // add the source to the map
        map.addSource('wildfires', {
          type: 'geojson',
          data: wildfires // use our data as the data source
        });
        // add the GeoJSON data as a mapbox gl layer
        map.addLayer({
          'id': 'wildfires',
          'type': 'fill',
          'source': 'wildfires', // refers to source above
          'layout': {},
          "paint": {
            "fill-color":"orange",
            "fill-opacity": 0.5
          }
        });

      });

    });
```

Upon saving and refreshing, your map should now have all the wildfire polygons in semi-transparent orange displayed on top of the dark matter vector tiles. Maplibre GL JS is creating vector tiles from the geojson data on the fly. You will know this because the complexity of the polygons increases as you zoom in and decreases as you zoom out. The result should look something like this:

![Wildfires over Dark Matter](images/wildfires-all.PNG)  
**Figure 04**. Wildfires over Dark Matter.

## Display Popup on Mouse Hover

Before tackling the more complicated time slider filtration interactivity, let's add and display popup content when the user hovers over each polygon. For this, we will need to write a function that parses the geojson attribute data and returns the information we want to the popup. First, we need to write some code to display an empty popup. Just after the method adding the scale bar and just before the function called after the map loads, we will define a popup and define the initial hover state id.

```js
    // add a scale bar
    map.addControl(new maplibregl.ScaleControl({
      position: 'bottom-left',
      unit: 'imperial' // you can change from imperial to metric, if desired
    }));

    // create a popup without adding to map
    let popup = new maplibregl.Popup({
      closeButton: false,
      closeOnClick: false
    });

    // define initial hover state
    let hoveredStateId = null;
```

Next, below the entire block function called on map loading (the very bottom of our code outside this function and just before the closing script tags), we add a function to fire upon moving the mouse over the wildfires layer. For this, we will reference the wildfires layer with the id we gave it when you added it to the map. We will also add some code that will change the cursor to a pointer when we hover over the layer, followed by some code to change the hover state id. This lets the layer know when to change its opacity value. We will also need to define the feature properties ("props" here) in order to access this data for the popup defined in the following code.

```js
    // fire a function on a mousemove and feed it the geojson we gave an id of 'wildfires'
    map.on('mousemove','wildfires', (e) => {

      // change the cursor style as a UI indicator.
      map.getCanvas().style.cursor = 'pointer';

      // reset the hover state id on mouse move so the layer knows when to change opacity
      if (e.features.length > 0) {
        if (hoveredStateId !== null) {
          map.setFeatureState(
            { source: 'wildfires', id: hoveredStateId },
            { hover: false }
          );
        }
        hoveredStateId = e.features[0].id;
        map.setFeatureState(
          { source: 'wildfires', id: hoveredStateId },
          { hover: true }
        );
      }

      // define the layer properties
      // with "e.features[0].properties," you have access to all of the properties from the initial geojson layer
      let props = e.features[0].properties;

      // get the popup defined above
      popup
        .setLngLat(e.lngLat) // the location of the cursor
        .setHTML(/*We will put the popup content here in HTML code*/) // feed the popup the prop info
        .addTo(map) // add the popup to the map

    });
```

After saving and refreshing your map, you should now see a popup with the word "undefined" appear when you hover over the wildfire polygons. Three issues exist: 1.) we need to add information to the popup, 2.) we need to change the opacity value back to the original value, and 3.) we need to add some code to close the popup after the mouse leaves the wildfires layer. Just beneath the ```map.on('mousemove','wildfires', (e) => {});``` code block, let's add another code block to reset opacity by reverting to the original hover state id and to remove the popup on mouse leave.

```js
    // when you mouse off the layer...
    map.on('mouseleave', 'wildfires', () => {

      // change the cursor style back to initial setting
      map.getCanvas().style.cursor = '';

      // reset the hover state id
      if (hoveredStateId !== null) {
        map.setFeatureState(
          { source: 'wildfires', id: hoveredStateId },
          { hover: false }
        );
      }
      hoveredStateId = null;

      // and remove the popup
      popup.remove();

    });
```

Now, the popup disappears when your cursor leaves the layer. However, we still need to add relevant feature properties to the popup. If you open the geojson file, you can see that there are properties for "YEAR_", "FIRE_NAME", and "GIS_ACRES". These all look like good content for the popup. Return to the ```.setHTML()``` section of the popup code and build it out as follows:

```js
      // get the popup defined above
      popup
        .setLngLat(e.lngLat) // the location of the cursor
        .setHTML('<b>' + props.FIRE_NAME + '</b><br>Year: ' + props.YEAR_ + '<br>Acres burned: ' + props.GIS_ACRES.toFixed(2)) // feed the popup the prop info
        .addTo(map) // add the popup to the map
```

Now, your map should display the fire name, year, and acres burned when you hover over a wildfire polygon. This is nice, but we also want to change the layer style on hover so that we know which shape corresponds to the fire for which we see popup information. Fortunately, this is pretty easy in Maplibre GL JS. To do this, all we need to do is return to the ```map.addLayer({});``` method and edit the layer options. To change the polygon opacity from 0.5 to 1.0 on hover, all we need to do is to edit the code as follows:

```js
        // add the GeoJSON data as a mapbox gl layer
        map.addLayer({
          'id': 'wildfires',
          'type': 'fill',
          'source': 'wildfires', // refers to source above
          'layout': {},
          "paint": {
            "fill-color":"orange",
            "fill-opacity": [
              'case',
              ['boolean', ['feature-state', 'hover'], false],
              1.0,
              0.5
            ]
          }
        });
```

There is one caveat to all of this; it only works if your geojson has a unique feature id (not a property id) for each feature in this format: ```"type": "Feature", "id": 1```. Take a look at [this stackoverflow forum](https://stackoverflow.com/questions/34314066/adding-a-unique-feature-id-to-geojson-in-qgis-or-python) for how to add this using QGIS if your data does not have this id.

The result should look like the image below.

![Popup and Shading on Hover](images/popup.PNG)  
**Figure 05**. Popup and Shading on Hover.

## Filter Geojson Data with the Time Slider

One of the issues with the current visualization is that all of the wildfires appear at once and overlap. The presentation is confusing and many fires are hidden behind later fires. Using the existing time slider, we are going to filter the polygons so that only the fires for the selected year appear on the screen.

First, at the very bottom of the script after the function fired by the mouseleave event, we will write a new function that will be fired upon an input change in the time slider.

```js
    // listen for a slider input
    document.getElementById('slider').addEventListener('input', (event) => {

      // define the newly-selected year
      const year = event.target.value;

      // update text in the UI
      document.getElementById('active-year').innerText = year;

      // set a filter on the wildfires layer that tests for a match between each feature's year and the year identified by the time slider
      map.setFilter('wildfires', ['==', ['string', ['get', 'YEAR_']], year]);

    });
```

With this block of code, the slider interacts with the year legend and the geojson data. Now you should see both the year change in the legend above the slider as well as the wildfire polygons filtering as you drag the slider. The image below shows all fires filtered to show only those that occurred in 1924.

![Temporal Filtration](images/slider-filter.PNG)  
**Figure 06**. Temporal Filtration.

This works, but all of the wildfire polygons still display when the map loads, so we need to go back and add the filter to the layer when it is first added to the map. At the very top of your script, just above ```const key```, add the following code to identify the initial year selected with the time slider and to change the year legend to reflect this year.

```js
// define the initial slider-selected year
const initYear = document.getElementById('slider').value;

// update text in the UI
document.getElementById('active-year').innerText = initYear;

// your maptiler key
const key = 'X79IRuovndFj8moKWjAt'
```

Next, filter your data on initial loading by adding the following filter option to ```map.addLayer({});```.

```js
        // add the GeoJSON data as a mapbox gl layer
        map.addLayer({
          'id': 'wildfires',
          'type': 'fill',
          'source': 'wildfires', // refers to source above
          'layout': {},
          "paint": {
            "fill-color":"orange",
            "fill-opacity": [
              'case',
              ['boolean', ['feature-state', 'hover'], false],
              1.0,
              0.5
            ]
          },
          "filter": ['==', ['string', ['get', 'YEAR_']], initYear]
        });
```  

You should now have [a web map application that generates vector tiles from heavy geojson data on the fly using MapLibre GL JS](https://jebowe3.github.io/maplibre-gl-js-demo/map/index.html). In case you mixed up anything, the following consists of the entire code used to generate this map. If you would like to see another, slightly more complicated, technique that speeds up your map by converting your geojson data to vector tiles before loading, skip the following code and follow the instructions in the next section.

```html
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

</head>

<body>

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

  <!-- maplibre-gl.js library -->
  <script src='https://unpkg.com/maplibre-gl@2.1.7/dist/maplibre-gl.js'></script>

  <!-- d3 js -->
  <script src="https://d3js.org/d3.v6.min.js"></script>

  <script>
    // javascript goes here

    // define the initial slider-selected year
    const initYear = document.getElementById('slider').value;

    // update text in the UI
    document.getElementById('active-year').innerText = initYear;

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

    // add zoom and rotation controls to the map.
    map.addControl(new maplibregl.NavigationControl());

    // add a scale bar
    map.addControl(new maplibregl.ScaleControl({
      position: 'bottom-left',
      unit: 'imperial' // you can change from imperial to metric, if desired
    }));

    // create a popup without adding to map
    let popup = new maplibregl.Popup({
      closeButton: false,
      closeOnClick: false
    });

    // define initial hover state
    let hoveredStateId = null;

    // when the map loads, call a function
    map.on('load', () => {

      // define incoming geojson data
      const calwf = d3.json('./data/calWildfires.geojson');

      // promise to wait until geojson files are loaded
      Promise.all([calwf]).then((data) => {

        // define the data according to its order in object array
        const wildfires = data[0];

        // add the source to the map
        map.addSource('wildfires', {
          type: 'geojson',
          data: wildfires // use our data as the data source
        });
        // add the GeoJSON data as a mapbox gl layer
        map.addLayer({
          'id': 'wildfires',
          'type': 'fill',
          'source': 'wildfires', // refers to source above
          'layout': {},
          "paint": {
            "fill-color":"orange",
            "fill-opacity": [
              'case',
              ['boolean', ['feature-state', 'hover'], false],
              1.0,
              0.5
            ]
          },
          "filter": ['==', ['string', ['get', 'YEAR_']], initYear]
        });

      });

    });

    // fire a function on a mousemove and feed it the geojson we gave an id of 'wildfires'
    map.on('mousemove','wildfires', (e) => {

      // change the cursor style as a UI indicator.
      map.getCanvas().style.cursor = 'pointer';

      // reset the hover state id on mousemove so the layer knows when to change opacity
      if (e.features.length > 0) {
        if (hoveredStateId !== null) {
          map.setFeatureState(
            { source: 'wildfires', id: hoveredStateId },
            { hover: false }
          );
        }
        hoveredStateId = e.features[0].id;
        map.setFeatureState(
          { source: 'wildfires', id: hoveredStateId },
          { hover: true }
        );
      }

      // define the layer properties
      // with "e.features[0].properties," you have access to all of the properties from the initial geojson layer
      let props = e.features[0].properties;

      // get the popup defined above
      popup
        .setLngLat(e.lngLat) // the location of the cursor
        .setHTML('<b>' + props.FIRE_NAME + '</b><br>Year: ' + props.YEAR_ + '<br>Acres burned: ' + props.GIS_ACRES.toFixed(2)) // feed the popup the prop info
        .addTo(map) // add the popup to the map

    });

    // when you mouse off the layer...
    map.on('mouseleave', 'wildfires', () => {

      // change the cursor style back to initial setting
      map.getCanvas().style.cursor = '';

      // reset the hover state id
      if (hoveredStateId !== null) {
        map.setFeatureState(
          { source: 'wildfires', id: hoveredStateId },
          { hover: false }
        );
      }
      hoveredStateId = null;

      // and remove the popup
      popup.remove();

    });


    // listen for a slider input
    document.getElementById('slider').addEventListener('input', (event) => {

      // define the newly-selected year
      const year = event.target.value;

      // update text in the UI
      document.getElementById('active-year').innerText = year;

      map.setFilter('wildfires', ['==', ['string', ['get', 'YEAR_']], year]);

    });

  </script>

</body>
```
