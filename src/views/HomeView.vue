<template>
  <div class="home">
    <div class="btns">
      <el-button @click="addPoint">添加点</el-button>
      <el-button @click="addLine">添加线</el-button>
      <el-button @click="addArea">添加面</el-button>
      <!-- <el-button @click="toPoint">定位到某个点</el-button> -->
      <!-- <div>选择进行绘制</div> -->
      <!-- <el-select v-model="selectValue" @change="selectChange">
        <el-option v-for="item in selectList" :key="item.value" :label="item.label" :value="item.value"> </el-option>
      </el-select> -->

      <!-- <el-button @click="addPic">添加图片标注</el-button> -->
      <!-- <el-button @click="addText">添加文字标注</el-button> -->
      <!-- <el-button @click="addPicAndText">添加图文标注</el-button> -->
      <!-- <el-button @click="addPopup">添加popup</el-button> -->
      <!-- <el-button @click="addPointAndView">轨迹回放</el-button> -->
      <!-- <el-button @click="addHeatmap">添加热力图</el-button> -->
      <!-- <el-button @click="addManyPoints">添加点聚合地图</el-button> -->
    </div>
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
    // 初始化百度地图
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
    },
    // 添加点
    addPoint () {
      // 创建一个红点
      const redPoint = new ol.Feature({
        geometry: new ol.geom.Point(ol.proj.transform([114.759, 25.522], 'EPSG:4326', 'EPSG:3857'))
      })
      // 设置点1的样式信息
      redPoint.setStyle(new ol.style.Style({
        // 填充色
        // fill: new ol.style.Fill({
        //   color: 'rgba(255, 255, 255, 0.2)'
        // }),
        // 边线颜色
        stroke: new ol.style.Stroke({
          color: 'pink',
          width: 2
        }),
        // 形状
        image: new ol.style.Circle({
          radius: 10,
          fill: new ol.style.Fill({
            color: 'red'
          })
        })
      }))
      // 实例化一个矢量图层Vector作为绘制层
      const source = new ol.source.Vector({
        features: [redPoint]
      })
      // 创建一个图层
      const vector = new ol.layer.Vector({
        source
      })
      // 将绘制层添加到地图容器中
      this.map.addLayer(vector)
    },
    // 添加线
    addLine () {
      // 创建一条黑线  2个点组成一条线
      const blackLine = new ol.Feature({
        geometry: new ol.geom.LineString([
          ol.proj.transform([114.752, 25.532], 'EPSG:4326', 'EPSG:3857'),
          ol.proj.transform([114.752, 25.512], 'EPSG:4326', 'EPSG:3857')
        ])
      })
      // 设置线的样式信息
      blackLine.setStyle(new ol.style.Style({
        // 填充色
        // fill: new ol.style.Fill({
        //   color: 'rgba(255, 255, 255, 0.2)'
        // }),
        // 边线颜色
        stroke: new ol.style.Stroke({
          color: '#000',
          width: 2
        }),
        // 形状
        image: new ol.style.Circle({
          radius: 10,
          fill: new ol.style.Fill({
            color: 'red'
          })
        })
      }))
      // 实例化一个矢量图层Vector作为绘制层
      const source = new ol.source.Vector({
        features: [blackLine]
      })
      // 创建一个图层
      const vector = new ol.layer.Vector({
        source
      })
      // 将绘制层添加到地图容器中
      this.map.addLayer(vector)
    },
    // 坐标添加矩形
    addArea () {
      // 创建一条黑线  2个点组成一条线
      const Rectangle = new ol.Feature({
        // geometry: new ol.geom.LineString([
        //   ol.proj.transform([114.752, 25.532], 'EPSG:4326', 'EPSG:3857'),
        //   ol.proj.transform([114.752, 25.512], 'EPSG:4326', 'EPSG:3857')
        // ])
        // eslint-disable-next-line
        geometry: new ol.geom.Polygon.fromExtent([16208725.0, 3035304.0, 9841418.0, 5068487.0])
      })
      // 设置样式信息
      Rectangle.setStyle(new ol.style.Style({
        // 填充色
        fill: new ol.style.Fill({
          color: 'rgba(255, 255, 255, 0.2)'
        }),
        // 边线颜色
        stroke: new ol.style.Stroke({
          color: '#000',
          width: 2
        }),
        // 形状
        image: new ol.style.Circle({
          radius: 10,
          fill: new ol.style.Fill({
            color: 'red'
          })
        })
      }))
      // 实例化一个矢量图层Vector作为绘制层
      const source = new ol.source.Vector({
        features: [Rectangle]
      })
      // 创建一个图层
      const vector = new ol.layer.Vector({
        source
      })
      // 将绘制层添加到地图容器中
      this.map.addLayer(vector)
    }
  }
}
</script>

<style lang="less" scoped>
.home{
  .btns{
    width: 100%;
    // height: 100px;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-wrap: wrap;
  }
  .map{
    height: calc(100vh - 100px);
  }
}
</style>
