<template>
  <div class="home">
    <div class="btns">
      <el-button @click="addPoint">添加点</el-button>
      <el-button @click="addLine">添加线</el-button>
      <el-button @click="addArea">添加面</el-button>
      <el-button @click="toPoint">定位到某个点</el-button>
      <div>选择绘制</div>
      <el-select v-model="selectValue" @change="selectChange">
        <el-option v-for="item in selectList" :key="item.value" :label="item.label" :value="item.value"> </el-option>
      </el-select>

      <el-button @click="addPic">添加图片标注</el-button>
      <el-button @click="addText">添加文字标注</el-button>
      <el-button @click="addPicAndText">添加图文标注</el-button>
      <el-button @click="addHeatmap">添加热力图</el-button>
      <el-button @click="addPointAndView">轨迹回放</el-button>
      <el-button @click="addManyPoints">添加点聚合地图</el-button>
    </div>
    <div ref="popup_content" class="popup_content" v-if="popupContentShow">
      <div class="fater">
        <div class="son">这里是注解文字</div>
      </div>
    </div>
    <div class="map" id="map"></div>
  </div>
</template>

<script>
import * as ol from 'openlayers'
import heatData from './heatData'

export default {
  name: 'HomeView',
  components: {
    // HelloWorld
  },
  data () {
    return {
      map: null,
      layerList: [],
      selectList: [
        {
          label: '清空绘制图形',
          value: 'None'
        },
        {
          label: '点',
          value: 'Point'
        },
        {
          label: '线',
          value: 'LineString'
        },
        {
          label: '多边形',
          value: 'Polygon'
        },
        {
          label: '圆',
          value: 'Circle'
        },
        {
          label: '正方形',
          value: 'Square'
        },
        {
          label: '长方形',
          value: 'Box'
        }
      ],
      selectValue: 'None',
      draw: '',
      source: '',
      vector: '',
      popupContentShow: false
    }
  },
  mounted () {
    // 初始化地图
    this.initMap()
    // 生成一个绘制层，用于地图绘制
    this.initDraw()
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
          center: ol.proj.transform([114.759, 25.522], 'EPSG:4326', 'EPSG:3857'), // ⼤地坐标系EPSG:4326   墨卡托EPSG:3857
          // center: ol.proj.fromLonLat([114.759, 25.522]),
          zoom: 15,
          minZoom: 3
        }),
        controls: ol.control.defaults().extend([new ol.control.FullScreen(), new ol.control.ScaleLine()]) // 为什么不生效
      })
    },
    // 清除图层
    clearMap () {
      if (this.layerList.length) {
        this.layerList.forEach(item => {
          this.map.removeLayer(item)
        })
      }
      this.map.getOverlays().clear()
    },
    moveToPosition (postion, zoom = 14) {
      this.map.getView().animate({
        // 将地理坐标转为投影坐标
        center: ol.proj.transform(postion, 'EPSG:4326', 'EPSG:3857'),
        duration: 500,
        zoom: zoom
      })
    },
    // 添加点
    addPoint () {
      this.clearMap()
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
      // 将已添加的图层存起来
      this.layerList.push(vector)
    },
    // 添加线
    addLine () {
      this.clearMap()
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
      // 将已添加的图层存起来
      this.layerList.push(vector)
    },
    // 坐标添加矩形
    addArea () {
      this.clearMap()
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
      // 将已添加的图层存起来
      this.layerList.push(vector)
    },
    // 定位到某坐标
    toPoint () {
      this.map.getView().animate({
        // 将地理坐标转为投影坐标
        center: ol.proj.transform([114.099, 22.451], 'EPSG:4326', 'EPSG:3857'),
        duration: 1000,
        zoom: 12
      })
    },
    // 实例化一个矢量图层Vector作为绘制层
    initDraw () {
      this.source = new ol.source.Vector({ wrapX: false })
      this.vector = new ol.layer.Vector({
        source: this.source,
        style: new ol.style.Style({
          fill: new ol.style.Fill({
            color: 'rgba(255, 255, 255, 0.2)'
          }),
          stroke: new ol.style.Stroke({
            color: '#ffcc33',
            width: 2
          }),
          image: new ol.style.Circle({
            radius: 5,
            fill: new ol.style.Fill({
              color: '#ffcc33'
            })
          })
        })
      })
      // 将绘制层添加到地图容器中
      this.map.addLayer(this.vector)
      // 将已添加的图层装起来
      this.layerList.push(this.vector)
    },
    selectChange () {
      // 移除绘制图形
      this.map.removeInteraction(this.draw)
      // 添加交互绘制功能控件
      this.addInteraction()
    },
    addInteraction () {
      let type
      if (this.selectValue !== 'None') {
        type = this.selectValue
        if (this.source == null) {
          this.source = new ol.source.Vector({ wrapX: false })
          // 添加绘制层数据源
          this.vector.setSource(this.source)
        }

        let geometryFunction, maxPoints
        if (this.selectValue === 'Square') {
          type = 'Circle' // 正方形图形（圆）
          geometryFunction = ol.interaction.Draw.createRegularPolygon(4)
        } else if (this.selectValue === 'Box') {
          type = 'LineString'
          maxPoints = 2
          geometryFunction = (coordinates, geometry) => {
            if (!geometry) { // 多边形
              geometry = new ol.geom.Polygon(null)
            }
            const start = coordinates[0]
            const end = coordinates[1]
            geometry.setCoordinates([[start, [start[0], end[1]], end, [end[0], start[1]], start]])
            return geometry
          }
        }
        // 实例化交互绘制类对象并添加到地图容器中
        this.draw = new ol.interaction.Draw({
          source: this.source,
          type,
          geometryFunction, // 几何信息变更时调用函数
          maxPoints // 最大点数
        })
        this.map.addInteraction(this.draw)
      } else {
        this.source = null
        // 清空绘制图形
        this.vector.setSource(this.source)
      }
    },
    // 添加图片标注
    addPic () {
      this.clearMap()
      const createLabelStyle = (feature) => {
        return new ol.style.Style({
          image: new ol.style.Icon({
            anchor: [0.5, 60],
            anchorOrigin: 'top-right',
            anchorXUnits: 'fraction',
            anchorYUnits: 'pixels',
            offsetOrigin: 'top-right',
            // offset:[0,10],
            // 图标缩放比例
            scale: 0.2,
            // 透明度
            opacity: 0.5,
            // 图标的url
            src: require('@/assets/logo.png')
          })
        })
      }
      // 实例化Vector要素，通过矢量图层添加到地图容器中
      const iconFeature = new ol.Feature({
        geometry: new ol.geom.Point(ol.proj.transform([114.769, 25.522], 'EPSG:4326', 'EPSG:3857'))
      })
      iconFeature.setStyle(createLabelStyle(iconFeature))
      // 矢量标注的数据源
      const vectorSource = new ol.source.Vector({
        features: [iconFeature]
      })
      // 矢量标注图层
      const vectorLayer = new ol.layer.Vector({
        source: vectorSource
      })
      this.map.addLayer(vectorLayer)
      // 将已添加的图层装起来
      this.layerList.push(vectorLayer)
    },
    // 添加文字标注
    addText () {
      this.clearMap()
      const createLabelStyle = (feature) => {
        return new ol.style.Style({
          text: new ol.style.Text({
            textAlign: 'center', // 位置
            textBaseline: 'middle', // 基准线
            font: 'normal 14px 微软雅黑', // 文字样式
            text: feature.get('name'), // 文本内容
            fill: new ol.style.Fill({ color: '#aa3300' }), // 文本填充样式（即文字颜色）
            stroke: new ol.style.Stroke({ color: '#ffcc33', width: 2 })
          })
        })
      }
      // 实例化Vector要素，通过矢量图层添加到地图容器中
      const iconFeature = new ol.Feature({
        geometry: new ol.geom.Point(ol.proj.transform([114.769, 25.522], 'EPSG:4326', 'EPSG:3857')),
        name: '南康区',
        population: 900
      })
      iconFeature.setStyle(createLabelStyle(iconFeature))
      // 矢量标注的数据源
      const vectorSource = new ol.source.Vector({
        features: [iconFeature]
      })
      // 矢量标注图层
      const vectorLayer = new ol.layer.Vector({
        source: vectorSource
      })
      this.map.addLayer(vectorLayer)
      this.moveToPosition([114.769, 25.522])
      // 将已添加的图层装起来
      this.layerList.push(vectorLayer)
    },
    // 添加图文标注
    addPicAndText () {
      this.clearMap()
      const createLabelStyle = (feature) => {
        return new ol.style.Style({
          text: new ol.style.Text({
            textAlign: 'center', // 位置
            textBaseline: 'middle', // 基准线
            font: 'normal 14px 微软雅黑', // 文字样式
            text: feature.get('name'), // 文本内容
            fill: new ol.style.Fill({ color: '#aa3300' }), // 文本填充样式（即文字颜色）
            stroke: new ol.style.Stroke({ color: '#ffcc33', width: 2 })
          }),
          image: new ol.style.Icon({
            anchor: [0.5, 60],
            anchorOrigin: 'top-right',
            anchorXUnits: 'fraction',
            anchorYUnits: 'pixels',
            offsetOrigin: 'top-right',
            // offset:[0,10],
            // 图标缩放比例
            scale: 0.2,
            // 透明度
            opacity: 0.5,
            // 图标的url
            src: require('@/assets/logo.png')
          })
        })
      }
      // 实例化Vector要素，通过矢量图层添加到地图容器中
      const iconFeature = new ol.Feature({
        geometry: new ol.geom.Point(ol.proj.transform([114.769, 25.522], 'EPSG:4326', 'EPSG:3857')),
        name: '南康区',
        population: 900
      })
      iconFeature.setStyle(createLabelStyle(iconFeature))
      // 矢量标注的数据源
      const vectorSource = new ol.source.Vector({
        features: [iconFeature]
      })
      // 矢量标注图层
      const vectorLayer = new ol.layer.Vector({
        source: vectorSource
      })
      this.map.addLayer(vectorLayer)
      this.moveToPosition([114.769, 25.522])
      // 将已添加的图层装起来
      this.layerList.push(vectorLayer)
    },
    // 添加热力图
    addHeatmap () {
      this.clearMap()
      this.moveToPosition([116.403, 39.924], 4)
      // 创建一个Heatmap图层
      const vector = new ol.layer.Heatmap({
        source: new ol.source.Vector({
          features: new ol.format.GeoJSON().readFeatures(heatData, {
            dataProjection: 'EPSG:4326',
            featureProjection: 'EPSG:3857'
          })
        }),
        // 热点半径
        radius: parseInt(15, 10),
        // 模糊尺寸
        blur: parseInt(25, 10)
      })

      this.map.addLayer(vector)
      // 将已添加的图层装起来
      this.layerList.push(vector)
    },
    // 添加点聚合地图
    addManyPoints () {
      this.clearMap()
      this.moveToPosition([116.403, 39.924], 7)

      // 此示例创建1000个要素
      const count = 1000
      const features = new Array(count)
      for (let i = 0; i < count; ++i) {
        const coordinates = ol.proj.transform([Math.random() * 100 + 100, Math.random() * 10 + 35], 'EPSG:4326', 'EPSG:3857')
        features[i] = new ol.Feature(new ol.geom.Point(coordinates))
      }
      // 矢量要素数据源
      const source = new ol.source.Vector({
        features: features
      })
      // 聚合标注数据源
      const clusterSource = new ol.source.Cluster({
        distance: 40,
        source: source
      })
      // 加载聚合标注的矢量图层
      const styleCache = {}
      const clusters = new ol.layer.Vector({
        source: clusterSource,
        style: function (feature, resolution) {
          const size = feature.get('features').length
          let style = styleCache[size]
          if (!style) {
            style = [
              new ol.style.Style({
                image: new ol.style.Circle({
                  radius: 20,
                  stroke: new ol.style.Stroke({
                    color: '#fff'
                  }),
                  fill: new ol.style.Fill({
                    color: '#3399CC'
                  })
                }),
                text: new ol.style.Text({
                  text: size.toString(),
                  fill: new ol.style.Fill({
                    color: '#fff'
                  })
                })
              })
            ]
            styleCache[size] = style
          }
          return style
        }
      })
      this.map.addLayer(clusters)
      // 将已添加的图层装起来
      this.layerList.push(clusters)

      // 下面的代码是让鼠标变成小手，并且有点击事件
      this.map.on('pointermove', e => {
        const pixel = this.map.getEventPixel(e.originalEvent)
        const hit = this.map.hasFeatureAtPixel(pixel)
        this.map.getTargetElement().style.cursor = hit ? 'pointer' : ''
      })

      // 为map添加点击事件监听，渲染弹出popup
      this.map.on('click', evt => {
        // 判断当前单击处是否有要素，捕获到要素时弹出popup
        const feature = this.map.forEachFeatureAtPixel(evt.pixel, (feature, layer) => {
          return feature
        })
        if (feature) {
          console.log(666)
          const popup = new ol.Overlay(({
            // 要转换成overlay的HTML元素
            element: this.$refs.popup_content,
            // 当前窗口可见
            autoPan: true,
            // Popup放置的位置
            positioning: 'bottom-center',
            // 是否应该停止事件传播到地图窗口
            stopEvent: false,
            autoPanAnimation: {
              // 当Popup超出地图边界时，为了Popup全部可见，地图移动的速度
              duration: 250
            }
          }))
          this.popupContentShow = true
          popup.setPosition(evt.coordinates)
          this.map.addOverlay(popup)
        }
      })
    },
    // 轨迹回放
    addPointAndView () {
      this.clearMap()
      this.moveToPosition([116.403, 39.924], 4)
      const Coordinates = []
      Coordinates.push(
        ol.proj.transform([117.403, 42.924], 'EPSG:4326', 'EPSG:3857'),
        ol.proj.transform([117.403, 41.924], 'EPSG:4326', 'EPSG:3857'),
        ol.proj.transform([117.403, 40.924], 'EPSG:4326', 'EPSG:3857'),
        ol.proj.transform([117.403, 38.924], 'EPSG:4326', 'EPSG:3857'),
        ol.proj.transform([117.403, 37.924], 'EPSG:4326', 'EPSG:3857'),
        ol.proj.transform([117.403, 32.924], 'EPSG:4326', 'EPSG:3857'),
        ol.proj.transform([127.403, 42.924], 'EPSG:4326', 'EPSG:3857'),
        ol.proj.transform([127.403, 52.924], 'EPSG:4326', 'EPSG:3857'),
        ol.proj.transform([137.403, 49.924], 'EPSG:4326', 'EPSG:3857')
      )
      const passCoordinate = []

      // 将离散点构建成一条折线
      const route = new ol.geom.LineString(Coordinates)

      const trackLineLength = route.getLength()
      const pointCount = trackLineLength / (this.map.getView().getResolution() * 15)
      for (let i = 1; i < pointCount; i++) {
        passCoordinate.push(route.getCoordinateAt(i / pointCount))
      }
      passCoordinate.unshift(Coordinates[0])
      passCoordinate.push(Coordinates[Coordinates.length - 1])

      // 获取直线的坐标
      const routeCoords = passCoordinate
      const routeLength = routeCoords.length

      const routeFeature = new ol.Feature({
        type: 'route',
        geometry: route
      })
      const geoMarker = new ol.Feature({
        type: 'geoMarker',
        geometry: new ol.geom.Point(routeCoords[0])
      })
      const startMarker = new ol.Feature({
        type: 'icon',
        geometry: new ol.geom.Point(routeCoords[0])
      })
      const endMarker = new ol.Feature({
        type: 'icon',
        geometry: new ol.geom.Point(routeCoords[routeLength - 1])
      })

      const styles = {
        route: new ol.style.Style({
          stroke: new ol.style.Stroke({
            width: 6,
            color: [237, 212, 0, 0.8]
          })
        }),
        icon: new ol.style.Style({
          image: new ol.style.Icon({
            anchor: [0.5, 1],
            src: require('@/assets/icon_address@2x.png')
          })
        }),
        geoMarker: new ol.style.Style({
          image: new ol.style.Circle({
            radius: 7,
            snapToPixel: false,
            fill: new ol.style.Fill({ color: 'black' }),
            stroke: new ol.style.Stroke({
              color: 'white',
              width: 2
            })
          })
        })
      }

      let animating = false
      let speed, now
      const vectorLayer = new ol.layer.Vector({
        source: new ol.source.Vector({
          features: [routeFeature, geoMarker, startMarker, endMarker]
        }),
        style: function (feature) {
          // 如果动画是激活的就隐藏geoMarker
          if (animating && feature.get('type') === 'geoMarker') {
            return null
          }
          return styles[feature.get('type')]
        }
      })
      this.map.addLayer(vectorLayer)
      // 将已添加的图层装起来
      this.layerList.push(vectorLayer)

      const moveFeature = event => {
        const vectorContext = event.vectorContext
        const frameState = event.frameState
        if (animating) {
          const elapsedTime = frameState.time - now
          // 通过增加速度，来获得lineString坐标
          const index = Math.round(speed * elapsedTime / 1000)
          if (index >= routeLength) {
            stopAnimation(true)
            return
          }
          const currentPoint = new ol.geom.Point(routeCoords[index])
          const feature = new ol.Feature(currentPoint)
          vectorContext.drawFeature(feature, styles.geoMarker)
        }
        // 继续动画效果
        this.map.render()
      }

      const startAnimation = () => {
        if (animating) {
          stopAnimation(false)
        } else {
          animating = true
          now = new Date().getTime()
          speed = 8000
          // 隐藏geoMarker
          geoMarker.setStyle(null)
          // 设置显示范围
          this.map.getView().setCenter(ol.proj.transform([116.403, 39.924], 'EPSG:4326', 'EPSG:3857'))
          this.map.on('postcompose', moveFeature)
          this.map.render()
        }
      }

      const stopAnimation = (ended) => {
        animating = false
        // 如果动画取消就开始动画
        const coord = ended ? routeCoords[routeLength - 1] : routeCoords[0]
        geoMarker.getGeometry().setCoordinates(coord)
        // 移除监听
        this.map.un('postcompose', moveFeature)
      }

      setTimeout(() => {
        startAnimation()
      }, 2000)
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
  .popup_content {
    .fater {
      width: 150px;
      height: 50px;
      background-color: aquamarine;
    }
  }
  .map{
    margin-top: 100px;
    height: calc(100vh - 200px);
  }
}
</style>
