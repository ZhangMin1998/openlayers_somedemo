<template>
  <div class="home">
    <div class="map" id="map"></div>
  </div>
</template>

<script>
import * as ol from 'openlayers'

export default {
  name: 'HomeView',
  components: {
    // HelloWorld
  },
  data () {
    return {
      map: null
    }
  },
  mounted () {
    // 初始化地图
    this.initMap()
  },
  methods: {
    initMap () {
      const projection = ol.proj.get('EPSG:3857')
      // 分辨率
      const resolutions = []
      for (let i = 0; i < 19; i++) {
        resolutions[i] = Math.pow(2, 18 - i)
      }
      const tilegrid = new ol.tilegrid.TileGrid({
        origin: [0, 0],
        resolutions: resolutions
      })
      // 拼接百度地图出图地址
      const baiduSource = new ol.source.TileImage({
        // 设置坐标参考系
        projection: projection,
        // 设置分辨率
        tileGrid: tilegrid,
        // 出图基地址
        tileUrlFunction: function (tileCoord, pixelRatio, proj) {
          if (!tileCoord) {
            return ''
          }
          const z = tileCoord[0]
          let x = tileCoord[1]
          let y = tileCoord[2]

          if (x < 0) {
            x = 'M' + -x
          }
          if (y < 0) {
            y = 'M' + -y
          }
          return 'http://online3.map.bdimg.com/onlinelabel/?qt=tile&x=' + x + '&y=' + y + '&z=' + z + '&styles=pl&udt=20151021&scaler=1&p=1'
        }
      })
      // 百度地图
      const baiduLayer = new ol.layer.Tile({
        source: baiduSource
      })
      console.log(baiduLayer)

      // 实例化比例尺控件（ScaleLine）
      // const scaleLineControl = new ol.control.ScaleLine({
      //   // 设置比例尺单位，degrees、imperial、us、nautical、metric（度量单位）
      //   units: 'metric'
      // })

      // 地图容器
      this.map = new ol.Map({
        target: 'map',
        layers: [baiduLayer],
        // layers: [new ol.layer.Tile({ source: new ol.source.OSM() })],
        view: new ol.View({
          center: ol.proj.transform([114.759, 25.522], 'EPSG:4326', 'EPSG:3857'),
          // center: ol.proj.fromLonLat([114.759, 25.522]),
          zoom: 15,
          minZoom: 3
        }),
        controls: ol.control.defaults().extend([new ol.control.FullScreen(), new ol.control.ScaleLine()]) // 为什么不生效
      })
    }
  }
}
</script>

<style lang="less" scoped>
.home{
  .map{
    height: calc(100vh - 100px);
  }
}
</style>
