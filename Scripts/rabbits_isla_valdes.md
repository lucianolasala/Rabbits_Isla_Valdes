// App title
var header1 = ui.Label("Change in Soil Adjusted Vegetation Index (SAVI) at Isla Valdés, Chubut, after European rabbit (Oryctolagus cuniculus) eradication.", 
            {fontSize: '16px', fontWeight: 'bold', color: '4A997E'});
            
// App summary
var text1 = ui.Label(
"This tool allows exploring the main results from remote sensing analyses presented in the publication " +
"\"Community-wide effects of rabbit eradication in Atlantic islands of Patagonia, Argentina. "
, {fontSize: '14px'});

//----------------------------------------------------------------------    
// Create a panel 1 to hold the above text
//----------------------------------------------------------------------

var panel1 = ui.Panel({
  widgets:[header1, text1], // Adds header and text
  style:{width: '350px', position:'top-right', backgroundColor: 'white'}});


//---------------------------------------------------------------------- 
// Create panel 2 to house a line separator and instructions for the user
//---------------------------------------------------------------------- 

var panel2 = ui.Panel([
  ui.Label({
    value: '___________________________________________________________',
    style: {fontWeight: 'bold',  color: '4A997E'},
  })]);

var panel2 = ui.Panel([
  ui.Label({
    value:'Select layer of interst:',
    style: {fontSize: '14px', fontWeight: 'bold', fontStyle: 'italic', color: '#e74c3c', backgroundColor: 'white', textDecoration: 'underline'}
  }),
  ]);
  
// Add our main panel to the root of our GUI
ui.root.insert(1, panel1)

// Add panel 3 with title
var layerSelect = ui.Select({
  items: ['None', 'Vegetation types', 'Min. difference', 'Threshold 0.5%', 'Threshold 1%'],
  onChange: function(selected) {
    // Remove all layers from the map
    Map.layers().reset();

    if (selected === 'None') {
      Map.setOptions('SATELLITE'); // Reset to Google Satellite view
    } else if (selected === 'Vegetation types') {
      Map.addLayer(maskedImage, visParams_1, 'Vegetation types');
    } else if (selected === 'Min. difference') {
      Map.addLayer(savi_diff_min, visParams_2, 'Min. difference');
    } else if (selected === 'Threshold 0.5%') {
      Map.addLayer(savi_05_diff, visParams_2, 'Threshold 0.5%');
    } else if (selected === 'Threshold 1%') {
      Map.addLayer(savi_1_diff, visParams_2, 'Threshold 1%');
    }

    // Re-add cameras
    Map.addLayer(styledPoints.style({styleProperty: 'style'}), {}, 'Rabbits Presence');

    // **Only add transects if checkbox is checked**
    if (toggleTransectCheckbox.getValue()) {
      Map.addLayer(transects, {color: 'black'}, "Transects");
    }
  },
  placeholder: 'Select a raster layer'
});

// Prevent triggering onChange when setting the initial value
layerSelect.setValue('None', false);  // The second argument (false) prevents calling onChange

// Add the dropdown menu to the UI
var panel3 = ui.Panel({
  widgets: [layerSelect],
  style: {position: 'top-right'}
});

//----------------------------------------------------------------
// Checkbox for activating/deactivating the farms layer
//----------------------------------------------------------------

// Create checkbox to toggle the visibility of the transects layer
var toggleTransectCheckbox = ui.Checkbox({
  label: 'Show/hide transects',
  value: false,  // Default to unchecked (layer off)
  onChange: function(checked) {
    if (checked) {
      // Add the transects layer if the checkbox is checked
      Map.addLayer(transects, {color: 'black'}, 'Transects');
    } else {
      // Remove the transects layer if the checkbox is unchecked
      // Use Map.layers() to get the list of layers and remove the transects layer
      var layers = Map.layers();
      var layerToRemove = null;

      // Find the transects layer
      layers.forEach(function(layer) {
        if (layer.getName() === 'Transects') {
          layerToRemove = layer;
        }
      });

      if (layerToRemove) {
        Map.layers().remove(layerToRemove);
      }
    }
  },
  style: {fontWeight: 'bold', fontSize: '14px', margin: '10px'}
});

// Create a panel for checkboxes and add to the map
var checkboxPanel = ui.Panel({
  widgets: [toggleTransectCheckbox],
  layout: ui.Panel.Layout.Flow('vertical'),
  style: {
    position: 'top-left',
    padding: '10px',
  }
});

// Add to the main panel
panel1.add(panel2).add(panel3).add(toggleTransectCheckbox);


// Set the default basemap to satellite
Map.setOptions('SATELLITE');

// Visualization parameters for raster of vegetation types
var palette_1 = [
  '0000FF', // Category 1: Blue. Sea. Masked
  '#00de00', // Category 2: Green. Restinga
  '#aa4500', // Category 3: Red. Rocky dessert
  '#ff7c00', // Category 4: Yellow. Senecio steppe
  '#4ce8c8', // Category 5: Cyan. Grindelia rocky plains
  '#f20027', // Category 6: Magenta. Grassy steppe
  '#ff35ff', // Category 7: Maroon. Atriplex steppe
  '#cbcbcb', // Category 8: White. Beach
  '#3636f3'  // Category 9: Blue. Colliguaja shrubland. 
];

// Create a mask for blue (assuming blue is category 1)
var blueMask = utmImage.neq(1); // Mask where the value is not 1 (blue)

// Apply the mask to the image
var maskedImage = utmImage.updateMask(blueMask);

var visParams_1 = {
  min: 1, // Minimum value for visualization
  max: 9, // Maximum value for visualization
  palette: palette_1
};


// Visualization parameters for raster comparisons between periods
var palette_2 = ['#ca6f1e','#58d68d'];

var visParams_2 = {
  min: 0, // Minimum value for visualization
  max: 1, // Maximum value for visualization
  palette: palette_2
};

Map.centerObject(island, 14); 

// Add cameras
// Define a styling function
var stylePoints = function(feature) {
  var rabbitValue = feature.get('Rabbits');
  
  // Define the color based on the "Rabbits" value
  var color = ee.Algorithms.If(
    ee.String(rabbitValue).equals('Yes'),
    '#2a2524', // Color if 'Yes'
    '#ffffff' // Color if 'No'
  );
  
  return feature.set({style: {color: color, pointSize: 5}});
};

// Apply the styling function
var styledPoints = cams.map(stylePoints);

// Add the styled points to the map
Map.addLayer(styledPoints.style({styleProperty: 'style'}), {}, 'Rabbits Presence');


//--------------------------------------------------------------------
// Define the palette and labels for vegetation types
//--------------------------------------------------------------------

var palette_b = [
  '#cbcbcb', // Category 8: White. Beach
  '#ff35ff', // Category 7: Maroon. Atriplex steppe
   '#3636f3',  // Category 9: Blue. Colliguaja shrubland. 
  '#f20027', // Category 6: Magenta. Grassy steppe
  '#4ce8c8', // Category 5: Cyan. Grindelia rocky plains
  '#00de00', // Category 2: Green. Restinga
  '#aa4500', // Category 3: Red. Rocky dessert
  '#ff7c00', // Category 4: Yellow. Senecio steppe
];

var labels = [
  'Beech',
  'Atriplex steppe',
  'Colliguaja shrubland',
  'Grassy steppe',
  'Grindelia rocky plains',
  'Restinga',
  'Rocky desert',
  'Senecio steppe'
];

// Create the legend panel
var legend = ui.Panel({
  style: {
    position: 'bottom-left',
    padding: '8px 15px'
  }
});

// Add the title to the legend
legend.add(ui.Label({
  value: 'Vegetation types',
  style: {
    fontWeight: 'bold',
    fontSize: '14px',
    margin: '0 0 8px 0',
    padding: '0'
  }
}));

// Create and style the legend items
for (var i = 0; i < palette_b.length; i++) {
  // Create the color box
  var colorBox = ui.Label({
    style: {
      backgroundColor: '' + palette_b[i],
      padding: '8px',
      margin: '0 0 4px 0'
    }
  });

  // Create the label
  var description = ui.Label({
    value: labels[i],
    style: {margin: '0 0 4px 6px', fontSize: '12px'},
  });

  // Add the color box and label to the legend
  var legendItem = ui.Panel({
    widgets: [colorBox, description],
    layout: ui.Panel.Layout.Flow('horizontal')
  });

  legend.add(legendItem);
}

// Add the legend to the map
Map.add(legend);

//--------------------------------------------------------------------
// Add circle labels for cameras using Unicode characters
//--------------------------------------------------------------------

// Define the colors and labels for the camera points
var cameraIcons = ['\u26AB', '\u26AA'];  // Black and White circle icons
var cameraLabels = ['With detection', 'Without detection']

// Add the title to the legend
legend.add(ui.Label({
  value: 'Camera traps',
  style: {
    fontWeight: 'bold',
    fontSize: '14px',
    margin: '12px 0 8px 0',
    padding: '0'
  }
}));

// Create and style the camera legend items
for (var j = 0; j < cameraIcons.length; j++) {
  // Create a label with a Unicode circle
  var circleLabel = ui.Label({
    value: cameraIcons[j] + "  " + cameraLabels[j],  // Add space for readability
    style: {
      fontSize: '12px',
      margin: '0 0 4px 0',
    }
  });

  legend.add(circleLabel);
}


//--------------------------------------------------------------------
// Add the black line for transects and its label
//--------------------------------------------------------------------

// Create a black line legend item
var lineBox = ui.Label({
  style: {
    backgroundColor: 'black',
    padding: '1.5px 8px',
    margin: '5px 0 0 0'
  }
});

var lineDescription = ui.Label({
  value: 'Transects',
  style: {margin: '0 0 4px 6px', fontSize: '12px'}
});

// Add the black line and label to the legend
var transectLegendItem = ui.Panel({
  widgets: [lineBox, lineDescription],
  layout: ui.Panel.Layout.Flow('horizontal')
});

legend.add(transectLegendItem);


// Create a panel to hold the scale bar
var scaleBarPanel = ui.Panel({
  style: {
    position: 'bottom-right',
    padding: '8px',
    backgroundColor: 'white'
  }
});

// Function to calculate the scale bar length in meters
function calculateScaleBar(map) {
  var zoomLevel = map.getZoom();
  var scale = Math.pow(2, 21 - zoomLevel); // Adjust the scale factor as needed

  var scaleBarLength = scale * 100; // 100 pixels length
  var scaleBarText = ui.Label({
    value: scaleBarLength.toFixed(0) + ' meters',
    style: { fontSize: '14px' }
  });

  // Update the scale bar panel
  scaleBarPanel.clear();
  scaleBarPanel.add(scaleBarText);
}

// Add the scale bar to the map and update it when the zoom changes
Map.onChangeZoom(function() {
  calculateScaleBar(Map);
});

// Initialize the scale bar for the current zoom level
calculateScaleBar(Map);

// Add the scale bar panel to the map
Map.add(scaleBarPanel);

