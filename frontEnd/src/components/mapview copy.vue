<template>
  <div class="mapview">
    <div id='map' class="mapcontainer">
     <el-amap
      :center='center'
      :zoom='zoom'
      vid="amapDemo"  
      :events="mapEvents"  
      class="amap-demo"
      >
      <el-amap-marker 
        :position="center"
        :label="label"
      ></el-amap-marker>
    </el-amap>
    </div>
  </div>
</template>
 
<script>

// import {IconLayer} from '@deck.gl/layers';
// import {MapboxLayer} from '@deck.gl/mapbox';
// import 'mapbox-gl/dist/mapbox-gl.css'
// let mapboxgl = require('mapbox-gl');
// mapboxgl.accessToken = 'pk.eyJ1IjoiaW52aXNpdmxlIiwiYSI6ImNrMjAwd3ZpODBqYWczanJ6cjR2MnVzN2EifQ.HfLXEXo4beVSXenqVeCu-w';
export default {
  name: 'mapview',
  data () {
    return {
      center: [114.356574, 30.526436],
      zoom: 12,
      label:{
      // content: '自定义标题',
      offset:[10,12]
      },
      mapEvents: {
                init(o) {
                     o.setMapStyle('amap://styles/dark');//自定义的高德地图的样式，我选的是马卡龙
                       
               }
          }
      
      
    }
  },
  // mounted() {
  //   this.init();

  // },
  // methods: {
  //   init() {
  //                   let INITIAL_VIEW_STATE = {
  //                       latitude: 44.1123006,
  //                       longitude: -117.9652526,
  //                       zoom: 5,
  //                       bearing: 0,
  //                       pitch: 30
  //                   };
  //                   mapboxgl.accessToken = 'pk.eyJ1IjoiaW52aXNpdmxlIiwiYSI6ImNrYTlwOGh5aTBwMjUzMG10c3R0dTltOG4ifQ.ZP3u66pJEeSZX4fJ7lL3SQ'; // eslint-disable-line
  //                   this.map = new mapboxgl.Map({
  //                       container: 'map',
  //                       style: 'mapbox://styles/mapbox/dark-v10',
  //                       attributionControl: false,
  //                       // Note: deck.gl will be in charge of interaction and event handling
  //                       interactive: true,
  //                       center: [INITIAL_VIEW_STATE.longitude, INITIAL_VIEW_STATE.latitude],
  //                       zoom: INITIAL_VIEW_STATE.zoom,
  //                       bearing: INITIAL_VIEW_STATE.bearing,
  //                       pitch: INITIAL_VIEW_STATE.pitch
  //                   });
                    
  //                   this.map.setMaxBounds([
  //                   [-180,-90],
  //                   [180,90]
  //                                   ])
  //   }
  // }
}
</script>

<style scoped>
.mapcontainer {
        position: absolute;
        left: 0;
        bottom: 0;
        width: 100%;
        height: 90%;
        z-index: 0; 
    }
</style>