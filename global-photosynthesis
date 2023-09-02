/**** Start of imports. If edited, may not auto-convert in the playground. ****/
var npp = ee.ImageCollection("MODIS/006/MOD17A3HGF");
/***** End of imports. If edited, may not auto-convert in the playground. *****/
/*
 * Mapping Global Monthly GPP
 */
 

/*
 * Configure GPP Layer
 */
var nppLayer = npp.select(['Npp'])
var img = ee.Image(npp.filterDate('2000-01-01','2001-01-01').first());

var layerProperties = {
  'Global Mean GPP': {
    name: 'Npp',
    visParams: {min: 0, max: 19000, palette: ["e8704f","f0dd60","49ab71","18477c","3b2e85"]},
    legend: [
      {'Low Npp': 'bbe029'}, {'Mean Npp': '0a9501'}, {'High Npp': '074b03'},
      {'No Npp': 'black'}, {'Water or no data': 'grey'}
    ],
    defaultVisibility: true
  }
};

/*
 * Map panel configuration
 */

// Create a map panel.
var mapPanel = ui.Map();

// Take all tools off the map except the zoom and mapTypeControl tools.
mapPanel.setControlVisibility(
    {all: false, zoomControl: true, mapTypeControl: true});

// Add these to the interface.
ui.root.widgets().reset([mapPanel]);
ui.root.setLayout(ui.Panel.Layout.flow('horizontal'));

// Add layers to the map and center it.
for (var key in layerProperties) {
  var layer = layerProperties[key];
  var image = img.select(layer.name).visualize(layer.visParams);
  mapPanel.add(ui.Map.Layer(image, {}, key, layer.defaultVisibility));
}

var inspectorPanel = ui.Panel({style: {width: '30%'}});
ui.root.widgets().add(inspectorPanel);

// Create an intro panel with labels.
var header = ui.Panel([
  ui.Label({
    value: 'Mapping Global Annual Gross Primary Productivity (GPP)',
    style: {fontSize: '24px', fontWeight: 'bold'}
  }),
  ui.Label(
    'Results from analysis of annual GPP datasets characterizing the global carbon uptake.',
    {fontSize: '14px'}),
]);
inspectorPanel.insert(0,header);


// Create a hyperlink to the research paper.
// var link = ui.Label(
//     'Presentation Link', {},
//     'https://docs.google.com/presentation/d/1Kp8oGEjd2ihuHLhgOBeoij9fb02uACW2X3ffojSQVpQ/edit?usp=share_link');
// var linkPanel = ui.Panel(
//     [ui.Label('For more information:', {fontWeight: 'bold'}), link]);
// inspectorPanel.insert(1,linkPanel);

// Create the legend.
var legendPanel = ui.Panel({
  style:
      {fontWeight: 'bold', fontSize: '10px', margin: '0 0 0 8px', padding: '0'}
});
inspectorPanel.insert(2,legendPanel);

// Define an area for the legend key itself.
// This area will be replaced every time the layer pulldown is changed.
var keyPanel = ui.Panel();
legendPanel.add(keyPanel);


function setLegend(legend) {
  // Loop through all the items in a layer's key property,
  // creates the item, and adds it to the key panel.
  keyPanel.clear();
  for (var i = 0; i < legend.length; i++) {
    var item = legend[i];
    var name = Object.keys(item)[0];
    var color = item[name];
    var colorBox = ui.Label('', {
      backgroundColor: color,
      // Use padding to give the box height and width.
      padding: '8px',
      margin: '0'
    });
    // Create the label with the description text.
    var description = ui.Label(name, {margin: '0 0 4px 6px'});
    keyPanel.add(
        ui.Panel([colorBox, description], ui.Panel.Layout.Flow('horizontal')));
  }
}

// Create the year pulldown.
var years = ['2000', '2005', '2010', '2015', '2020'];
var yearSelect = ui.Select({
  items: years,
  value: years[0],
  onChange: function(value) {
    var endNumber = Number(value) + 1;
    var startValue = value+'-01-01';
    var endValue = endNumber+'-01-01';
    img = ee.Image(npp.filterDate(startValue,endValue).first());
    
    // Add layers to the map and center it.
    for (var key in layerProperties) {
      var layer = layerProperties[key];
      var image = img.select(layer.name).visualize(layer.visParams);
      mapPanel.add(ui.Map.Layer(image, {}, key, layer.defaultVisibility));
    }
  }
});

var yearPanel = ui.Panel([
  ui.Label('View A Different Year', {'font-size': '20px', fontWeight: 'bold'}), yearSelect
]);
inspectorPanel.insert(3, yearPanel);

// Create an intro panel with labels.
var intro = ui.Panel([
  ui.Label({
    value: 'GPP - Time Series Inspector',
    style: {fontSize: '20px', fontWeight: 'bold'}
  }),
  ui.Label('Click a location to see its time series of GPP values.')
]);
inspectorPanel.insert(4, intro);

// Create panels to hold lon/lat values.
var lon = ui.Label();
var lat = ui.Label();
inspectorPanel.add(ui.Panel([lon, lat], ui.Panel.Layout.flow('horizontal')));
//inspectorPanel.insert(5, ui.Panel([lon, lat], ui.Panel.Layout.flow('horizontal')));6
/*
 * Chart setup
 */

// Generates a new time series chart of NPP for the given coordinates.
var generateChart = function (coords) {
  // Update the lon/lat panel with values from the click event.
  lon.setValue('longitude: ' + coords.lon.toFixed(2));
  lat.setValue('latitude: ' + coords.lat.toFixed(2));

  // Add a dot for the point clicked on.
  var point = ee.Geometry.Point(coords.lon, coords.lat);
  var dot = ui.Map.Layer(point, {color: 'FF0000'}, 'clicked location');
  // Add the dot as the second layer, so it shows up on top of the composite.
  mapPanel.layers().set(1, dot);

  // Make a chart from the time series.
  var nppChart = ui.Chart.image.series(nppLayer, point, ee.Reducer.mean(), 500, 'system:time_start');

  // Customize the chart.
  nppChart.setOptions({
    title: 'GPP: time series',
    style: {fontSize: '20px', fontWeight: 'bold'},
    vAxis: {title: 'GPP value'},
    hAxis: {title: 'Date', format: 'yyyy-MM', gridlines: {count: 7}},
    series: {
      0: {
        color: 'green',
        lineWidth: 1,
        pointsVisible: true,
        pointSize: 2,
      },
    },
    legend: {position: 'right'},
  });
  // Add the chart at a fixed position, so that new charts overwrite older ones.
  inspectorPanel.widgets().set(5, nppChart);
};

/*
 * Legend setup
 */

// Creates a color bar thumbnail image for use in legend from the given color
// palette.
function makeColorBarParams(palette) {
  return {
    bbox: [0, 0, 1, 0.1],
    dimensions: '100x10',
    format: 'png',
    min: 0,
    max: 1,
    palette: palette,
  };
}

var vis = {min: 0, max: 19000, palette: "e8704f, f0dd60, 49ab71, 18477c, 3b2e85"};
// Create the color bar for the legend.
var colorBar = ui.Thumbnail({
  image: ee.Image.pixelLonLat().select(0),
  params: makeColorBarParams(vis.palette),
  style: {stretch: 'horizontal', margin: '0px 8px', maxHeight: '24px'},
});

// Create a panel with three numbers for the legend.
var legendLabels = ui.Panel({
  widgets: [
    ui.Label(vis.min, {margin: '4px 8px'}),
    ui.Label(
        (vis.max / 2),
        {margin: '4px 8px', textAlign: 'center', stretch: 'horizontal'}),
    ui.Label(vis.max, {margin: '4px 8px'})
  ],
  layout: ui.Panel.Layout.flow('horizontal')
});

var legendTitle = ui.Label({
  value: 'Map Legend: GPP values',
  style: {fontWeight: 'bold'}
});

var legendPanel = ui.Panel([legendTitle, colorBar, legendLabels]);
inspectorPanel.widgets().set(3, legendPanel);

/*
 * Map setup
 */

// Register a callback on the default map to be invoked when the map is clicked.
mapPanel.onClick(generateChart);

// Configure the map.
mapPanel.style().set('cursor', 'crosshair');

// Initialize with a test point.
var initialPoint = ee.Geometry.Point(11.27, 3.39);
mapPanel.centerObject(initialPoint, 2);

/*
 * Initialize the app
 */

// Replace the root with a SplitPanel that contains the inspector and map.
ui.root.clear();
ui.root.add(ui.SplitPanel(inspectorPanel, mapPanel));

generateChart({
  lon: initialPoint.coordinates().get(0).getInfo(),
  lat: initialPoint.coordinates().get(1).getInfo()
});