<template>
  <div class="mapview">
    <div class="street-view" id="street-view" v-if="this.showStreetView"></div>
    <video
      v-if="this.showVideo"
      id="filmCut"
      class="filmCut"
      autoplay
      loop
      @mousedown="dragFunc($event,'filmCut')"
    ></video>
    <el-col class="switches" v-if="this.showStreetView||this.showVideo">
      <el-switch @change="streetViewChange($event)" active-text="显示街景" v-model="showStreetView"></el-switch>
      <el-switch
        @change="fimlCutChange($event)"
        active-text="显示视频"
        style="margin-top: 10px"
        v-model="showVideo"
      ></el-switch>
    </el-col>
    <el-col class="newswitch">
      <el-switch @change="AlarmChange()" active-text="显示警报" v-model="alarm"></el-switch>
    </el-col>
    <!-- <el-col class="newswitch1">
      <el-switch  active-text="救援路线规划" v-model="alarm1"></el-switch>
      <el-switch  active-text="运送伤员路线规划" v-model="alarm2"></el-switch>
    </el-col> -->
    <div id="map" class="mapcontainer"></div>
    <div id="distance" class="distance-container"></div>
    <div class="map-overlay" id="changecolor" @mousedown="dragFunc($event,'changecolor')">
      <div class="map-overlay-inner">
        <fieldset>
          <label>Select layer</label>
          <select id="layer" name="layer">
            <option value="water">Water</option>
            <option value="building">Buildings</option>
            <option value="landuse">landuse</option>
            <option value="streets">roads</option>
          </select>
        </fieldset>
        <fieldset>
          <label>Choose a color</label>
          <div id="swatches"></div>
        </fieldset>
      </div>
    </div>
    <div id="menu">
      <input id="streets-v11" type="radio" name="rtoggle" value="streets" checked="checked" />
      <label for="streets">streets</label>
      <input id="light-v10" type="radio" name="rtoggle" value="light" />
      <label for="light">light</label>
      <input id="dark-v10" type="radio" name="rtoggle" value="dark" />
      <label for="dark">dark</label>
      <input id="outdoors-v11" type="radio" name="rtoggle" value="outdoors" />
      <label for="outdoors">outdoors</label>
      <input id="satellite-v9" type="radio" name="rtoggle" value="satellite" />
      <label for="satellite">satellite</label>
    </div>
  </div>
</template>
 
<script>
import { IconLayer } from "@deck.gl/layers";
import { MapboxLayer } from "@deck.gl/mapbox";
import { ScatterplotLayer, LineLayer, PolygonLayer } from "@deck.gl/layers";
import data from "../assets/json/convertcsv";
import "mapbox-gl/dist/mapbox-gl.css";
import MapboxGeocoder from "@mapbox/mapbox-gl-geocoder";
import "@mapbox/mapbox-gl-geocoder/dist/mapbox-gl-geocoder.css";
import "_@turf_turf@6.3.0@@turf/turf/turf.min.js";
// import turf from "_@turf_turf@6.3.0@@turf/turf/turf.min.js";
// import img from "../assets/p2.png"
let mapboxgl = require("mapbox-gl");
mapboxgl.accessToken =
  "pk.eyJ1Ijoid2VpNTMwIiwiYSI6ImNrbWdpMHFtYzBrMTEyd3MwaWNxd3hvdG8ifQ.U1gwQVBBYF1S4isdOdIDfw";
export default {
  name: "mapview",
  data() {
    return {
      showVideo: false,
      showStreetView: false,
      info: null,
      alarm: false,
      alarm1: false,
      alarm2: false,
      alarm3: false,
      alarm4: false,
      imgUrl: require("../assets/img/point1.png"),
      imgUrlEmergence: require("../assets/img/emergence.png"),
      imgUrlKong: require("../assets/img/kong.png"),
      imgUrlBKong: require("../assets/img/bkong.png")
    };
  },
  mounted() {
    let s1 = document.createElement("script");
    s1.type = "text/javascript";
    s1.src =
      "https://cn.bing.com/api/maps/mapcontrol?key=Alp7ogPrXX23_ZkgMTKg0def61NyZeHx_hJrN_dyxN47JTCez3gV53ar2PGoWqdj";
    document.body.appendChild(s1);
    let s2 = document.createElement("script");
    s2.type = "text/javascript";
    s2.src =
      "https://api.mapbox.com/mapbox-gl-js/plugins/mapbox-gl-directions/v4.1.0/mapbox-gl-directions.js";
    document.body.appendChild(s2);
    let l = document.createElement("link");
    l.href =
      "https://api.mapbox.com/mapbox-gl-js/plugins/mapbox-gl-directions/v4.1.0/mapbox-gl-directions.css";
    l.type = "text/css";
    l.rel = "stylesheet";
    document.body.appendChild(l);
    this.init();
  },
  methods: {
    init() {
      let INITIAL_VIEW_STATE = {
        latitude: 42.35906,
        longitude: -71.09228,
        zoom: 14,
        bearing: 0,
        pitch: 30
      };
      mapboxgl.accessToken =
        "pk.eyJ1IjoiaW52aXNpdmxlIiwiYSI6ImNrYTlwOGh5aTBwMjUzMG10c3R0dTltOG4ifQ.ZP3u66pJEeSZX4fJ7lL3SQ"; // eslint-disable-line
      this.map = new mapboxgl.Map({
        container: "map",
        style: "mapbox://styles/mapbox/dark-v10",
        attributionControl: false,
        //自适应
        trackResize: true,
        // Note: deck.gl will be in charge of interaction and event handling
        interactive: true,
        // camera options properties - https://docs.mapbox.com/help/glossary/camera/
        pitch: 60, // pitch in degrees
        bearing: -60, // bearing in degrees
        center: [INITIAL_VIEW_STATE.longitude, INITIAL_VIEW_STATE.latitude],
        zoom: INITIAL_VIEW_STATE.zoom
        // bearing: INITIAL_VIEW_STATE.bearing,
        // pitch: INITIAL_VIEW_STATE.pitch
      });

      //生成动态图标
      // var size = 200;

      // // implementation of CustomLayerInterface to draw a pulsing dot icon on the map
      // // see https://docs.mapbox.com/mapbox-gl-js/api/#customlayerinterface for more info
      // var pulsingDot = {
      //   width: size,
      //   height: size,
      //   data: new Uint8Array(size * size * 4),

      //   // get rendering context for the map canvas when layer is added to the map
      //   onAdd: function() {
      //     var canvas = document.createElement("canvas");
      //     canvas.width = this.width;
      //     canvas.height = this.height;
      //     this.context = canvas.getContext("2d");
      //   },

      //   // called once before every frame where the icon will be used
      //   render: function() {
      //     var duration = 1000;
      //     var t = (performance.now() % duration) / duration;

      //     var radius = (size / 2) * 0.3;
      //     var outerRadius = (size / 2) * 0.7 * t + radius;
      //     var context = this.context;

      //     // draw outer circle
      //     context.clearRect(0, 0, this.width, this.height);
      //     context.beginPath();
      //     context.arc(
      //       this.width / 2,
      //       this.height / 2,
      //       outerRadius,
      //       0,
      //       Math.PI * 2
      //     );
      //     context.fillStyle = "rgba(255, 200, 200," + (1 - t) + ")";
      //     context.fill();

      //     // draw inner circle
      //     context.beginPath();
      //     context.arc(this.width / 2, this.height / 2, radius, 0, Math.PI * 2);
      //     context.fillStyle = "rgba(255, 100, 100, 1)";
      //     context.strokeStyle = "white";
      //     context.lineWidth = 2 + 4 * (1 - t);
      //     context.fill();
      //     context.stroke();

      //     // update this image's data with data from the canvas
      //     this.data = context.getImageData(0, 0, this.width, this.height).data;

      //     // continuously repaint the map, resulting in the smooth animation of the dot
      //     this.map.triggerRepaint();

      //     // return `true` to let the map know that the image was updated
      //     return true;
      //   }
      // };

      //更改图层颜色
      this.changeLayerColor();

      // Create a default Marker and add it to the map.
      // var marker1 = new mapboxgl.Marker()
      //   .setLngLat([114.356574, 30.526436])
      //   .addTo(this.map);

      this.map.setMaxBounds([
        [-180, -90],
        [180, 90]
      ]);

      //切换地图样式
      var layerList = document.getElementById("menu");
      var inputs = layerList.getElementsByTagName("input");
      for (var i = 0; i < inputs.length; i++) {
        inputs[i].onclick = this.switchLayer;
      }

      // let ICON_MAPPING = {
      //   marker: { x: 0, y: 0, width: 200, height: 200, mask: false }
      // };

      // this.map.on('load', () => {
      //               this.map.addLayer(new MapboxLayer({
      //                   id: 'icon-layer',
      //                   type: IconLayer,
      //                   data: data.features,
      //                   pickable: true,
      //                   opacity:0.7,
      //                   // iconAtlas and iconMapping are required
      //                   // getIcon: return a string
      //                   iconAtlas: require('../assets/img/point1.png'),
      //                   iconMapping: ICON_MAPPING,
      //                   getIcon: d => 'marker',
      //                   sizeScale: 10,
      //                   getPosition: d => d.geometry.coordinates,
      //                   getSize: 4,
      //                   getColor: d => [Math.sqrt(d.exits), 140, 0],

      //               }));
      //           });

      this.map.on("load", () => {
        //this.buildings3D();

        this.map.loadImage(this.imgUrl, (error, image) => {
          if (error) throw error;

          //Add the image to the map style.
          this.map.addImage("point", image);
          // this.map.addImage("pulsing-dot", pulsingDot, { pixelRatio: 2 });

          this.map.addSource("points1", {
            type: "geojson",
            data: data
          });
          // this.map.addSource("points", {
          //   type: "geojson",
          //   data: {
          //     type: "FeatureCollection",
          //     features: [
          //       {
          //         type: "Feature",
          //         geometry: {
          //           type: "Point",
          //           coordinates: [114.356521, 30.526124]
          //         }
          //       }
          //     ]
          //   }
          // });

          this.map.addLayer({
            id: "pointsicon",
            type: "symbol",
            source: "points1",
            layout: {
              "icon-image": "point",
              "icon-size": 0.1
            }
          });

          // Create a popup, but don't add it to the map yet.
          // var popup = new mapboxgl.Popup({
          //   closeButton: false,
          //   closeOnClick: false
          // });

          // this.map.on("mouseenter", "pointsicon", function(e) {
          //   // Change the cursor style as a UI indicator.
          //   this.map.getCanvas().style.cursor = "pointer";

          //   var coordinates = e.features[0].geometry.coordinates.slice();
          //   var description = e.features[0].properties.description;

          //   // Ensure that if the map is zoomed out such that multiple
          //   // copies of the feature are visible, the popup appears
          //   // over the copy being pointed to.
          //   while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
          //     coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
          //   }

          //   // Populate the popup and set its coordinates
          //   // based on the feature found.
          //   popup
          //     .setLngLat(coordinates)
          //     .setHTML(name)
          //     .addTo(map);
          // });

          // this.map.on("mouseleave", "places", function() {
          //   this.map.getCanvas().style.cursor = "";
          //   popup.remove();
          // });
          //move point
          // this.map.addLayer({
          //   id: "points",
          //   type: "symbol",
          //   source: "points",
          //   layout: {
          //     "icon-image": "pulsing-dot"
          //   }
          // });
        });
        //kong
        this.loadWatch();
        //bkong
        this.loadblueWatch();
        //emergence
        this.loadAlarm();
        // Add the control to the map.
        this.map.addControl(
          new MapboxGeocoder({
            accessToken: mapboxgl.accessToken,
            mapboxgl: mapboxgl
          }),
          "bottom-right"
        );

        // Add zoom and rotation controls to the map.
        this.map.addControl(new mapboxgl.NavigationControl());
        //View a fullscreen map
        this.map.addControl(new mapboxgl.FullscreenControl());

        //directions
        this.map.addControl(
          new MapboxDirections({
            accessToken: mapboxgl.accessToken
          }),
          "bottom-left"
        );

        this.map.on("click", "emergenceP", info => {
          this.$message({
            type: "success",
            message: "加载成功"
          });
          this.info = info;
          console.log(info);
          //this.loadStreetView(info.lngLat);
          this.loadVideo();
        });

        // When a click event occurs on a feature in the places layer, open a popup at the
        // location of the feature, with description HTML from its properties.
        // this.map.on("click", "pointsicon", info => {
        //   var coordinates = e.features[0].geometry.coordinates.slice();
        //   var description = e.features[0].properties.description;

        //   // Ensure that if the map is zoomed out such that multiple
        //   // copies of the feature are visible, the popup appears
        //   // over the copy being pointed to.
        //   while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
        //     coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
        //   }

        //   new mapboxgl.Popup()
        //     .setLngLat(coordinates)
        //     .setHTML(name)
        //     .addTo(map);
        // });
      });
    },
    //alarm
    loadAlarm() {
      setTimeout(() => {
        this.map.loadImage(
          //引入图片
          this.imgUrlEmergence,
          (error, image) => {
            if (error) throw error;

            //Add the image to the map style.
            this.map.addImage("emergence", image);

            this.map.addSource("pointsemergence", {
              type: "geojson",
              data: {
                type: "FeatureCollection",
                features: [
                  {
                    type: "Feature",
                    geometry: {
                      type: "Point",
                      coordinates: [-71.08967, 42.35131]
                    },
                    properties: {
                      title: "Emergency！",
                      icon: "monument"
                    }
                  }
                ]
              }
            });

            this.map.addLayer({
              id: "emergenceP",
              type: "symbol",
              source: "pointsemergence",
              layout: {
                "icon-image": "emergence",
                "icon-size": 0.3,
                "text-field": "{title}",
                "text-font": ["Open Sans Semibold", "Arial Unicode MS Bold"],
                "text-offset": [0, 0.6],
                "text-anchor": "top"
              }
            });
            this.map.setLayoutProperty("emergenceP", "visibility", "none");
          }
        );
      }, 500);
    },
    //watch
    loadWatch() {
      setTimeout(() => {
        this.map.loadImage(
          //引入图片
          this.imgUrlKong,
          (error, image) => {
            if (error) throw error;

            //Add the image to the map style.
            this.map.addImage("kong", image);

            this.map.addSource("pointkong", {
              type: "geojson",
              data: {
                type: "FeatureCollection",
                features: [
                  {
                    type: "Feature",
                    geometry: {
                      type: "Point",
                      coordinates: [-71.09273, 42.3575]
                    },
                    properties: {
                      title: "monitor3",
                      icon: "monument"
                    }
                  },
                  {
                    type: "Feature",
                    geometry: {
                      type: "Point",
                      coordinates: [-71.09154, 42.35032]
                    },
                    properties: {
                      title: "monitor16",
                      icon: "monument"
                    }
                  },
                  {
                    type: "Feature",
                    geometry: {
                      type: "Point",
                      coordinates: [-71.08864, 42.34946]
                    },
                    properties: {
                      title: "monitor41",
                      icon: "monument"
                    }
                  }
                ]
              }
            });

            this.map.addLayer({
              id: "KongP",
              type: "symbol",
              source: "pointkong",
              layout: {
                "icon-image": "kong",
                "icon-size": 0.3,
                "text-field": "{title}",
                "text-font": ["Open Sans Semibold", "Arial Unicode MS Bold"],
                "text-offset": [0, 0.6],
                "text-anchor": "top"
              }
            });
            this.map.setLayoutProperty("KongP", "visibility", "visible");
          }
        );
      }, 500);
    },
    //bluewatch
    loadblueWatch() {
      setTimeout(() => {
        this.map.loadImage(
          //引入图片
          this.imgUrlBKong,
          (error, image) => {
            if (error) throw error;

            //Add the image to the map style.
            this.map.addImage("Bkong", image);

            this.map.addSource("pointBkong", {
              type: "geojson",
              data: {
                type: "FeatureCollection",
                features: [
                  {
                    type: "Feature",
                    geometry: {
                      type: "Point",
                      coordinates: [114.366645, 30.52162]
                    },
                    properties: {
                      title: "监控1",
                      icon: "monument"
                    }
                  },
                  {
                    type: "Feature",
                    geometry: {
                      type: "Point",
                      coordinates: [114.351194, 30.527594]
                    },
                    properties: {
                      title: "监控4",
                      icon: "monument"
                    }
                  }
                ]
              }
            });

            this.map.addLayer({
              id: "BKongP",
              type: "symbol",
              source: "pointBkong",
              layout: {
                "icon-image": "Bkong",
                "icon-size": 0.3,
                "text-field": "{title}",
                "text-font": ["Open Sans Semibold", "Arial Unicode MS Bold"],
                "text-offset": [0, 0.6],
                "text-anchor": "top"
              }
            });
            this.map.setLayoutProperty("BKongP", "visibility", "visible");
          }
        );
      }, 500);
    },
    //加载街景
    loadStreetView(lngLat) {
      if (this.showStreetView === false) {
        this.showStreetView = true;
      }
      setTimeout(function() {
        let map = new Microsoft.Maps.Map(
          document.getElementById("street-view"),
          {
            /* No need to set credentials if already passed in URL */
            mapTypeId: Microsoft.Maps.MapTypeId.streetside,
            zoom: 18,
            center: new Microsoft.Maps.Location(42.35131, -71.08967)
            //center: new Microsoft.Maps.Location(lngLat.lat, lngLat.lng)
          }
        );
        map.setView({ mapTypeId: Microsoft.Maps.MapTypeId.streetside });
      }, 500);
    },
    //加载视频
    loadVideo(name) {
      if (this.showVideo === false) {
        this.showVideo = true;
      }
      console.log(name);
      setTimeout(function() {
        let videoNode = document.getElementById("filmCut");
        videoNode.style.display = "block";
        videoNode.setAttribute(
          "src",
          require(`../assets/video/shootLocations/交通路口.mp4`)
        );
      }, 3000);
    },
    dragFunc($event, id) {
      let dom = document.getElementById(id);
      let startX = $event.clientX - dom.offsetLeft;
      let startY = $event.clientY - dom.offsetTop;
      dom.onmousemove = function(event) {
        dom.style.left = event.clientX - startX + "px";
        dom.style.top = event.clientY - startY + "px";
        dom.style.cursor = "move";
      };
      dom.onmouseout = function() {
        dom.onmousemove = null;
        dom.style.cursor = "default";
      };
      dom.onmouseup = function(event) {
        dom.onmousemove = null;
        dom.style.cursor = "default";
      };
    },

    streetViewChange($event) {
      if ($event) {
        this.loadStreetView(this.info.lngLat);
      }
    },
    fimlCutChange($event) {
      if ($event) {
        this.loadVideo();
      }
    },
    AlarmChange() {
      var visibility = this.map.getLayoutProperty("emergenceP", "visibility");
      if (visibility === "visible") {
        this.map.setLayoutProperty("emergenceP", "visibility", "none");
      } else {
        this.map.setLayoutProperty("emergenceP", "visibility", "visible");
      }
    },
    //更改地图样式
    switchLayer(layer) {
      var layerId = layer.target.id;
      this.map.setStyle("mapbox://styles/mapbox/" + layerId);
    },

    //改变图层颜色
    changeLayerColor() {
      var swatches = document.getElementById("swatches");
      var layer = document.getElementById("layer");
      var colors = [
        "#ffffcc",
        "#a1dab4",
        "#41b6c4",
        "#006400",
        "#2c7fb8",
        "#253494",
        "#fed976",
        "#feb24c",
        "#fd8d3c",
        "#bd0026"
      ];

      var that = this;
      colors.forEach(function(color) {
        var swatch = document.createElement("button");
        swatch.style.backgroundColor = color;
        swatch.addEventListener("click", function() {
          that.map.setPaintProperty(layer.value, "fill-color", color);
        });
        swatches.appendChild(swatch);
      });
    },

    //3D建筑
    buildings3D() {
      // Insert the layer beneath any symbol layer.
      var layers = this.map.getStyle().layers;
      var labelLayerId;
      for (var i = 0; i < layers.length; i++) {
        if (layers[i].type === "symbol" && layers[i].layout["text-field"]) {
          labelLayerId = layers[i].id;
          break;
        }
      }

      // The 'building' layer in the Mapbox Streets
      // vector tileset contains building height data
      // from OpenStreetMap.
      this.map.addLayer(
        {
          id: "add-3d-buildings",
          source: "composite",
          "source-layer": "building",
          filter: ["==", "extrude", "true"],
          type: "fill-extrusion",
          minzoom: 15,
          paint: {
            "fill-extrusion-color": "#F5F5F5",

            // Use an 'interpolate' expression to
            // add a smooth transition effect to
            // the buildings as the user zooms in.
            "fill-extrusion-height": [
              "interpolate",
              ["linear"],
              ["zoom"],
              15,
              0,
              15.01,
              ["get", "height"]
            ],
            "fill-extrusion-base": [
              "interpolate",
              ["linear"],
              ["zoom"],
              15,
              0,
              15.05,
              ["get", "min_height"]
            ],
            "fill-extrusion-opacity": 0.8
          }
        },

        labelLayerId
      );
    }
  }
};
</script>

<style scoped>
.mapcontainer {
  position: relative;
  width: 100%;
  height: 100%;
  z-index: 1;
}

#map {
  height: 100%;
  width: 100%;
}

#menu {
  position: absolute;
  background: #fff;
  padding: 10px;
  font-family: "Open Sans", sans-serif;
  z-index: 1;
}
#swatches {
  height: 170px;
}
.map-overlay {
  font: 12px/20px "Helvetica Neue", Arial, Helvetica, sans-serif;
  position: absolute;
  width: 300px;
  z-index: 1;
  top: 100px;
  left: 0;
  padding: 10px;
  float: left;
  color: white;
}

.map-overlay .map-overlay-inner {
  background-color: rgba(0, 0, 0, 0.2);
  box-shadow: 0 1px 2px rgba(255, 255, 255, 0.8);
  border-radius: 3px;
  padding: 10px;
  margin-bottom: 10px;
}

.map-overlay-inner fieldset {
  border: none;
  padding: 0;
  margin: 0 0 10px;
  width: 250px;
  height: 50px;
}

.map-overlay-inner fieldset:last-child {
  margin: 0;
}

.map-overlay-inner select {
  width: 100%;
  background-color: rgba(0, 0, 0, 0.2);
  color: white;
}

.map-overlay-inner label {
  display: block;
  font-weight: bold;
  margin: 0 0 5px;
}

.map-overlay-inner button {
  display: inline-block;
  width: 108px;
  height: 80px;
  border: none;
  cursor: pointer;
}

.map-overlay-inner button:focus {
  outline: none;
}

.map-overlay-inner button:hover {
  box-shadow: inset 0 0 0 3px rgba(0, 0, 0, 0.1);
}

.distance-container {
  position: absolute;
  top: 10px;
  right: 30px;
  z-index: 1;
}

.distance-container > * {
  background-color: rgba(0, 0, 0, 0.5);
  color: #fff;
  font-size: 11px;
  line-height: 18px;
  display: block;
  margin: 0;
  padding: 5px 10px;
  border-radius: 3px;
}

.filmCut {
  position: absolute;
  z-index: 3;
  height: 270px;
  width: 480px;
  display: none;
  right: 0;
  top: 0;
}
.street-view {
  position: absolute;
  z-index: 2;
  left: 0;
  top: 0;
  width: 100vw;
  height: 100vh;
}
.newswitch {
  position: absolute;
  left: 30px;
  top: 300px;
  z-index: 2;
  display: flex;
  flex-direction: column;
}
.newswitch1 {
  position: absolute;
  left: 30px;
  top: 100px;
  z-index: 2;
  display: flex;
  flex-direction: column;
}
.switches {
  /* width: 80px; */
  position: absolute;
  left: 30px;
  top: 100px;
  z-index: 2;
  display: flex;
  flex-direction: column;
}
.mapboxgl-popup {
  max-width: 400px;
  font: 12px/20px "Helvetica Neue", Arial, Helvetica, sans-serif;
}
/deep/.el-switch {
  color: white;
}
</style>