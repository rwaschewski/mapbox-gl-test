<template>
  <div id="app">
      <mapbox
      @map-load="mapLoaded"
      access-token="pk.eyJ1Ijoicndhc2NoZXdza2kiLCJhIjoiY2oxdGZ3MXowMDAydDMybzF6ZHBqdTNtNCJ9.9b4vYuVJfF4T0ndZsoEZ2Q"
      :map-options="{
        style: 'mapbox://styles/mapbox/dark-v9',
        center: [-122.447303, 37.753574],
        zoom: 12
      }"
      :geolocate-control="{
        show: true, 
        position: 'top-left'
      }"
      :scale-control="{
        show: true,
        position: 'top-right'
      }"
      :fullscreen-control="{
        show: true,
        position: 'top-left'
      }"
      @map-init="mapInitialized">
      </mapbox>
  </div>
</template>

<script>
import Mapbox from 'mapbox-gl-vue';
export default {
  name: 'app',
  components: {
    'mapbox': Mapbox
  },
  data () {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
  },
  methods: {
    mapInitialized(map) {
      var directions = new MapboxDirections({
      accessToken: 'pk.eyJ1Ijoicndhc2NoZXdza2kiLCJhIjoiY2oxdGZ3MXowMDAydDMybzF6ZHBqdTNtNCJ9.9b4vYuVJfF4T0ndZsoEZ2Q',
      unit: 'metric',
      profile: 'mapbox/driving-traffic'
      });
      map.addControl(directions);
    },
    mapLoaded(map) {
      map.addLayer({
        'id': 'population',
        'type': 'circle',
        'source': {
            type: 'vector',
            url: 'mapbox://examples.8fgz4egr'
        },
        'source-layer': 'sf2010',
        'paint': {
            // make circles larger as the user zooms from z12 to z22
            'circle-radius': {
                'base': 1.75,
                'stops': [[12, 2], [22, 180]]
            },
            // color circles by ethnicity, using data-driven styles
            'circle-color': {
                property: 'ethnicity',
                type: 'categorical',
                stops: [
                    ['White', '#fbb03b'],
                    ['Black', '#223b53'],
                    ['Hispanic', '#e55e5e'],
                    ['Asian', '#3bb2d0'],
                    ['Other', '#ccc']]
            }
        }
      });
      map.addLayer({
        "id": "route",
        "type": "line",
        "source": {
            "type": "geojson",
            "data": {
                "type": "Feature",
                "properties": {},
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [-122.48369693756104, 37.83381888486939],
                        [-122.48348236083984, 37.83317489144141],
                        [-122.48339653015138, 37.83270036637107],
                        [-122.48356819152832, 37.832056363179625],
                        [-122.48404026031496, 37.83114119107971],
                        [-122.48404026031496, 37.83049717427869],
                        [-122.48348236083984, 37.829920943955045],
                        [-122.48356819152832, 37.82954808664175],
                        [-122.48507022857666, 37.82944639795659],
                        [-122.48610019683838, 37.82880236636284],
                        [-122.48695850372314, 37.82931081282506],
                        [-122.48700141906738, 37.83080223556934],
                        [-122.48751640319824, 37.83168351665737],
                        [-122.48803138732912, 37.832158048267786],
                        [-122.48888969421387, 37.83297152392784],
                        [-122.48987674713133, 37.83263257682617],
                        [-122.49043464660643, 37.832937629287755],
                        [-122.49125003814696, 37.832429207817725],
                        [-122.49163627624512, 37.832564787218985],
                        [-122.49223709106445, 37.83337825839438],
                        [-122.49378204345702, 37.83368330777276],
                        [-122.49421204345702, 37.83484330777276]

                    ]
                }
            }
        },
        "layout": {
            "line-join": "round",
            "line-cap": "round"
        },
        "paint": {
            "line-color": "#F23",
            "line-width": 8
        }
      });
      map.addSource('earthquakes', {
        "type": "geojson",
        "data": "/mapbox-gl-js/assets/earthquakes.geojson"
      });
      map.addLayer({
        "id": "earthquakes-heat",
        "type": "heatmap",
        "source": "earthquakes",
        "maxzoom": 9,
        "paint": {
            //Increase the heatmap weight based on frequency and property magnitude
            "heatmap-weight": {
                "property": "mag",
                "type": "exponential",
                "stops": [
                    [0, 0],
                    [6, 1]
                ]
            },
            //Increase the heatmap color weight weight by zoom level
            //heatmap-ntensity is a multiplier on top of heatmap-weight
            "heatmap-intensity": {
                "stops": [
                    [0, 1],
                    [9, 3]
                ]
            },
            //Color ramp for heatmap.  Domain is 0 (low) to 1 (high).
            //Begin color ramp at 0-stop with a 0-transparancy color
            //to create a blur-like effect.
            "heatmap-color": [
                "interpolate",
                ["linear"],
                ["heatmap-density"],
                0, "rgba(33,102,172,0)",
                0.2, "rgb(103,169,207)",
                0.4, "rgb(209,229,240)",
                0.6, "rgb(253,219,199)",
                0.8, "rgb(239,138,98)",
                1, "rgb(178,24,43)"
            ],
            //Adjust the heatmap radius by zoom level
            "heatmap-radius": {
                "stops": [
                    [0, 2],
                    [9, 20]
                ]
            },
            //Transition from heatmap to circle layer by zoom level
            "heatmap-opacity": {
                "default": 1,
                "stops": [
                    [7, 1],
                    [9, 0]
                ]
            },
        }
    }, 'waterway-label');

    map.addLayer({
        "id": "earthquakes-point",
        "type": "circle",
        "source": "earthquakes",
        "minzoom": 7,
        "paint": {
            //Size circle raidus by earthquake magnitude and zoom level
            "circle-radius": {
                "property": "mag",
                "type": "exponential",
                "stops": [
                    [{ zoom: 7, value: 1 }, 1],
                    [{ zoom: 7, value: 6 }, 4],
                    [{ zoom: 16, value: 1 }, 5],
                    [{ zoom: 16, value: 6 }, 50],
                ]
            },
            //Color circle by earthquake magnitude
            "circle-color": {
                "property": "mag",
                "type": "exponential",
                "stops": [
                    [1, "rgba(33,102,172,0)"],
                    [2, "rgb(103,169,207)"],
                    [3, "rgb(209,229,240)"],
                    [4, "rgb(253,219,199)"],
                    [5, "rgb(239,138,98)"],
                    [6, "rgb(178,24,43)"]
                ]
            },
            "circle-stroke-color": "white",
            "circle-stroke-width": 1,
            //Transition from heatmap to circle layer by zoom level
            "circle-opacity": {
                "stops": [
                    [7, 0],
                    [8, 1]
                ]
            }
        }
      }, 'waterway-label');
    }
  }
}
</script>

<style lang="scss">
body {
  margin: 0em;
}
#map {
  position:absolute; 
  top:0; 
  bottom:0;
  width: 100%;
  min-height: 500px;
  height: 100vh;
}
</style>
