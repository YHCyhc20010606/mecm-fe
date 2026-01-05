<!--
  -  Copyright 2020-2021 Huawei Technologies Co., Ltd.
  -
  -  Licensed under the Apache License, Version 2.0 (the "License");
  -  you may not use this file except in compliance with the License.
  -  You may obtain a copy of the License at
  -
  -      http://www.apache.org/licenses/LICENSE-2.0
  -
  -  Unless required by applicable law or agreed to in writing, software
  -  distributed under the License is distributed on an "AS IS" BASIS,
  -  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  -  See the License for the specific language governing permissions and
  -  limitations under the License.
  -->

<template>
  <div class="mapContainer">
    <div class="content">
      <div
        id="mapChart"
        class="chart"
        v-show="showMainView"
      />
      <div
        id="mapDetailMap"
        class="detailMap"
        v-show="!showMainView"
      />
      <el-button
        type="primary"
        class="return"
        v-if="btnShow"
        @click="returnOverviewModel"
      >
        {{ $t('overview.returnOverview') }}
      </el-button>
    </div>
  </div>
</template>

<script>
import { inventory, check } from '../../tools/request.js'
import echarts from 'echarts'

import 'ol/ol.css'
import { Map, View } from 'ol'
import TileLayer from 'ol/layer/Tile'
import OlFeature from 'ol/Feature'
import OlGeomPoint from 'ol/geom/Point'
import OlLayerVector from 'ol/layer/Vector'
import OlSourceVector from 'ol/source/Vector'
import OlCluster from 'ol/source/Cluster'
import Style from 'ol/style/Style'
import Icon from 'ol/style/Icon'
import XYZ from 'ol/source/XYZ'
import InteractionSelect from 'ol/interaction/Select'

import axios from 'axios'
export default {
  name: 'Map',
  props: {
    detail: {
      required: true,
      type: Object,
      detault: function () {
        return null
      }
    }
  },
  data () {
    return {
      chinaId: 'world',
      chinaName: 'world',
      chinaJson: null,
      nodeData: [],
      continue: true,
      btnShow: false,
      map: null,
      showMainView: true,
      language: localStorage.getItem('language') || 'cn'
    }
  },
  mounted () {
    this.getNodeList()
    let detailMap = document.getElementById('mapDetailMap')
    detailMap.style.height = window.innerHeight
  },
  watch: {
    detail () {
      if (this.detail.mechostIp) {
        let arr = []
        arr.push(this.detail)
        this.showLayers(arr)
        this.$emit('node', this.detail)
      }
    },
    '$i18n.locale': function () {
      let language = localStorage.getItem('language')
      this.language = language
      this.getNodeList()
    }
  },
  methods: {
    getNodeList () {
      inventory.getList(2).then(res => {
        if (res.data && res.data.length > 0) {
          this.nodeData = res.data
          this.checkStatus()
        }
      }, error => {
        console.log(error)
      })
    },
    checkStatus () {
      check.healthCheck().then(res => {
        let result = res.data.data
        this.nodeData.forEach((item) => {
          item.coordinates = item.coordinates.split(',')
          if (result[item.mechostIp] === 'Healthy') {
            item.status = true
          }
        })
        this.$emit('area', this.nodeData, true)
        this.mapChart('mapChart')
      })
    },
    showLayers (arr) {
      this.showMainView = false
      this.$nextTick(() => {
        this.openlayers(arr)
      })
    },
    mapChart (divid) {
      axios.get('./map/' + this.chinaId + '.json', {}).then(response => {
        const mapJson = response.data
        this.chinaJson = mapJson
        let myChart = echarts.init(document.getElementById(divid))
        window.onresize = function () {
          myChart.resize()
        }
        this.regAndSetOption(myChart, this.chinaName, mapJson, false)
        myChart.on('click', (param) => {
          if (param.componentType === 'markPoint') {
            let arr = []
            arr.push(param.data)
            this.$emit('node', param.data)
            this.showLayers(arr)
          }
        })
      })
    },
    regAndSetOption (myDetailMap, name, mapJson) {
      let data = this.nodeData
      data.forEach(item => {
        item.coord = item.coordinates
      })
      echarts.registerMap(name, mapJson)
      myDetailMap.setOption({
        visualMap: {
          show: false
        },
        tooltip: {
          padding: 0,
          enterable: true,
          transitionDuration: 1,
          textStyle: {
            color: '#000',
            decoration: 'none'
          },
          formatter: function (params) {
            var tipHtml = ''
            let onlineNum = 0
            let offlineNum = 0
            data.forEach(item => {
              if (item.status) {
                onlineNum++
              } else {
                offlineNum++
              }
            })
            if (localStorage.getItem('language') === 'en') {
              if (params.componentType === 'markPoint') {
                tipHtml = `<div class="showLabelArea">
                            <div class="secondLabel">
                              Node Info
                            </div>
                            <div class="labelContent">
                              <p>Name：${params.data.mechostName}</p>
                              <p>IP：${params.data.mechostIp}</p>
                              <p>Address：${params.data.city}</p>
                            </div>
                          </div>`
              } else {
                tipHtml = `<div class="showLabelArea">
                            <div class="secondLabel">
                              Area Node Info
                            </div>
                            <div class="labelContent">
                              <p>Online：${onlineNum}</p>
                              <p>Offline：${offlineNum}</p>
                            </div>
                          </div>`
              }
            } else {
              if (params.componentType === 'markPoint') {
                tipHtml = `<div class="showLabelArea">
                            <div class="secondLabel">
                              节点信息
                            </div>
                            <div class="labelContent">
                              <p>名称：${params.data.mechostName}</p>
                              <p>IP：${params.data.mechostIp}</p>
                              <p>地址：${params.data.city}</p>
                            </div>
                          </div>`
              } else {
                tipHtml = `<div class="showLabelArea">
                            <div class="secondLabel">
                              地域节点信息
                            </div>
                            <div class="labelContent">
                              <p>在线节点：${onlineNum}</p>
                              <p>离线节点：${offlineNum}</p>
                            </div>
                          </div>`
              }
            }
            return tipHtml
          }
        },
        series: [
          {
            type: 'map',
            map: name,
            zoom: 1.2,
            roam: true,
            aspectScale: 0.75,
            animationDelayUpdate: 300,
            top: '30%',
            label: {
              emphasis: {
                show: true,
                color: '#fff'
              }

            },
            itemStyle: {
              normal: {
                areaColor: '#3E279B',
                borderColor: '#ffffff',
                boxShadow: '10px 20px 30px '
              },
              emphasis: {
                areaColor: '#43F6AD'
              }
            },
            data: this.initMapData(mapJson),
            markPoint: {
              symbol: 'image://./outer.png',
              symbolSize: [36, 36],
              hoverAnimation: true,
              data: data
            }
          }
        ]
      })
    },
    initMapData (mapJson) {
      let mapData = []
      for (let features of mapJson.features) {
        mapData.push({
          name: features.properties.name
        })
      }
      return mapData
    },
    returnOverviewModel () {
      let myChart = echarts.init(document.getElementById('mapChart'))
      this.regAndSetOption(myChart, this.chinaName, this.chinaJson, false)
      this.showMainView = true
      this.btnShow = false
      this.continue = true
      this.$emit('area', this.nodeData, false)
    },
    openlayers (data) {
      let _this = this
      this.btnShow = true
      if (this.map) {
        this.map.setView(new View({
          projection: 'EPSG:4326',
          center: data[0].coordinates,
          zoom: 16
        }))
      } else {
        this.map = new Map({
          target: 'mapDetailMap',
          layers: [
            new TileLayer({
              source: new XYZ({
                url: 'https://{a-c}.tile.openstreetmap.org/{z}/{x}/{y}.png'
              })
            })
          ],
          view: new View({
            projection: 'EPSG:4326',
            center: data[0].coordinates,
            zoom: 16
          })
        })
      }
      let lnglats = []
      this.nodeData.forEach(item => {
        lnglats.push(item.coordinates)
      })

      let features = []
      for (let lnglat of lnglats) {
        features.push(
          new OlFeature({
            type: 'icon',
            geometry: new OlGeomPoint(lnglat),
            eventTarget_: data
          })
        )
      }
      var source = new OlSourceVector({
        features: features
      })

      var clusterSource = new OlCluster({
        distance: 0,
        source: source
      })

      var clusters = new OlLayerVector({
        source: clusterSource,
        style: new Style({
          image: new Icon({
            src: './outer.png',
            scale: 1
          })
        }),
        zIndex: 66
      })

      this.map.addLayer(clusters)

      this.map.on('click', (e) => {
        var pixel = this.map.getEventPixel(e.originalEvent)
        this.map.forEachFeatureAtPixel(pixel, function (feature) {
          data.forEach(item => {
            if (feature.geometryChangeKey_.target.extent_[0] === parseFloat(item.coordinates[0])) {
              _this.$emit('node', item)
            }
          })
        })
      })

      let selectSingleClick = new InteractionSelect({})

      this.map.addInteraction(selectSingleClick)
      selectSingleClick.on('select', function (event) {
        event.selected[0].setStyle(new Style({
          image: new Icon({
            src: './inner.png',
            scale: 0.3
          })
        }))
      })
    }
  }
}

</script>

<style lang='less' scoped>
.mapContainer{
  height: 100%;
  padding: 25px;
  border-radius: 16px;
  background: rgba(46,20,124,0.7);
}
.content {
  width: 100%;
  height: 100%;
  .chart,.detailMap {
    height: 100%;
    width: 100%;
    z-index: 99;
  }
  .return{
    position: absolute;
    top: 40px;
    left: 81%;
    z-index: 999;
    background: #3E279B;
    color: #ffffff;
    border-radius: 6px;
    height: 35px;
    font-size: 16px;
    line-height: 35px;
    padding: 0 10px;
  }
}

</style>
