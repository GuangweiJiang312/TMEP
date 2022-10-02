<template>
  <div id="app">
    <pagetitle></pagetitle>
    <mapview id="mapView" :style="{ height: clientHeight + 'px' }"></mapview>
  </div>
</template>

<script>
// 导入组件
import hello from './components/hello'
import pagetitle from './components/pagetitle'
import mapview from './components/mapview'
 
export default {
  name: 'app',
  data(){
    return {
      clientHeight:0
    }
  },
  components: {
    hello,
    pagetitle,
    mapview,
  },
  mounted(){
    this.clientHeight = this.getClientHeight()
  },
  methods:{

getClientHeight()
{
  var clientHeight=0;
  if(document.body.clientHeight&&document.documentElement.clientHeight)
  {
  var clientHeight = (document.body.clientHeight<document.documentElement.clientHeight)?document.body.clientHeight:document.documentElement.clientHeight;
  }
  else
  {
  var clientHeight = (document.body.clientHeight>document.documentElement.clientHeight)?document.body.clientHeight:document.documentElement.clientHeight;
  }
  console.log(clientHeight);
  
  return clientHeight;
  }
}
}
</script>

<style>

</style>
