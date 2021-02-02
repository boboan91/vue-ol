<template>
  <div id="map" ref="rootmap"></div>
</template>

<script>
import "ol/ol.css";
import { Map, View } from "ol";
import TileLayer from "ol/layer/Tile";
import OSM from "ol/source/OSM";
import XYZ from "ol/source/XYZ";
import timg from "@/assets/timg.gif";
import Feature from "ol/Feature";
import Point from "ol/geom/Point";
import Polygon from "ol/geom/Polygon";
import { Vector as VectorLayer } from "ol/layer";
import { fromLonLat } from "ol/proj";
import TileJSON from "ol/source/TileJSON";
import VectorSource from "ol/source/Vector";
import GeoJSON from 'ol/format/GeoJSON';
import Circle from 'ol/geom/Circle';
import { Icon, Style, Circle as CircleStyle, Fill, Stroke} from "ol/style";
import area from '@/assets/area.json';
import * as turf from "@turf/turf";
import {unByKey} from 'ol/Observable';
import { defaults as controldefaults } from 'ol/control';
import FullScreen from 'ol/control/FullScreen';
export default {
  data() {
    return {
      map: null,
      arrs: [
        {
          coor: [114.064839, 22.548857],
          src: timg,
          id: "timg1"
        },
        {
          coor: [114.063839, 22.539857],
          src: timg,
          id: "timg2"
        }
      ],
      mapkey: null
    };
  },
  mounted() {
    //初始化地图
    this.initMap();
    //添加点（图片）
    this.addPoint();
    //添加GIF
    // this.addGif();
    //添加事件
    this.addMapEvent();
    //图层显示隐藏
    // this.controlLayerVisible();
    //添加json
    // this.addGeoJson();
  },
  methods: {
    initMap() {
      this.map = new Map({
        target: "map",
        controls: controldefaults().extend([
          new FullScreen(),//全屏
        ]),
        layers: [
          new TileLayer({
            source: new XYZ({
              url:
                "http://t" +
                Math.round(Math.random() * 7) +
                ".tianditu.com/DataServer?T=vec_w&x={x}&y={y}&l={z}&tk=d2ed074249289de3c086f6b95f1f89a0"
            }),
            isGroup: true,
            name: "天地图路网"
          }),
          new TileLayer({
            source: new XYZ({
              url:
                "http://t" +
                Math.round(Math.random() * 7) +
                ".tianditu.com/DataServer?T=cva_w&x={x}&y={y}&l={z}&tk=d2ed074249289de3c086f6b95f1f89a0"
            }),
            isGroup: true,
            name: "天地图路网标注"
          })
        ],
        view: new View({
          // projection: "EPSG:3857", //使用这个坐标系
          projection:"EPSG:4326",
          // center:[0,0],
          center: [114.064839, 22.548857], //深圳
          zoom: 8,//级别
          // extent:[113.005,21.882,115.653,23.521],
          // smoothExtentConstraint:false,
          minZoom:5,
          maxZoom:16
        })
      });
    },
    addPoint() {
      //1
      var point = new Point([114.074839, 22.548857]);
      //2
      var rome = new Feature({
        geometry: point
      });
      var rome1 = new Feature({
        geometry: new Point([114.084839, 22.548857])
      });
      var rome2 = new Feature({
        geometry: new Point([114.064839, 22.548857])
      });
      rome.on("singleclick",function(event){
        console.log("rome我被点击了");
      })
      rome.setStyle(
        new Style({
          image: new Icon({
            color: "#8959A8",
            crossOrigin: "anonymous",
            src: timg,
            scale: 0.1
          })
        })
      );
      rome1.setStyle(
        new Style({
          image: new Icon({
            color: "#8959A8",
            crossOrigin: "anonymous",
            src: timg,
            scale: 0.1
          })
        })
      );
      rome2.setStyle(
        new Style({
          image: new Icon({
            color: "#8959A8",
            crossOrigin: "anonymous",
            src: timg,
            scale: 0.1
          })
        })
      );
      //3
      var vectorSource = new VectorSource({
        features: [rome,rome1,rome2]
        // features:[rome]
      });
      //4
      var vectorLayer = new VectorLayer({
        layerName:"pointLayer",
        source: vectorSource
      });
      //5
      this.map.addLayer(vectorLayer);
    },
    addGif() {
      var that = this;
      let key = this.map.on("postcompose", function() {
        for (var i = that.arrs.length - 1; i >= 0; i--) {
          var obj = that.arrs[i];
          let img;
          var xy = that.map.getPixelFromCoordinate(obj.coor);

          if (document.getElementById(obj.id)) {
            img = document.getElementById(obj.id);
            img.style.left = xy[0] + "px";
            img.style.top = xy[1] + "px";
          } else {
            img = document.createElement("img");
            img.id = obj.id;
            img.src = obj.src;
            img.style.position = "absolute";
            img.style.left = xy[0] + "px";
            img.style.top = xy[1] + "px";
            img.width = 50;
            img.height = 50;
            document.body.appendChild(img);
          }
        }
      });
    },
    addMapEvent(){
      var map = this.map;
      // 响应单击事件
      let singleclickkey = map.on('singleclick', function(event){
          console.log('触发了ol.Map的单击事件：singleclick');
          map.forEachFeatureAtPixel(event.pixel, feature => {
              // 为移动到的feature发送自定义的singleclick消息
              feature.dispatchEvent({type:'singleclick', event: event});

              console.log(feature);
          });
      });

      setTimeout(function(){
          unByKey(singleclickkey);
      },10000)
  
      // 响应双击事件
      // map.on('dblclick', function(event){
      //   console.log(event);
      //     console.log('触发了ol.Map的双击事件：dblclick');
      // });
  
      // 响应点击事件
      // map.on('click', function(event){
      //     console.log('触发了ol.Map的点击事件：click');
      // });
  
      //响应鼠标移动事件
      // map.on('pointermove', function(event){
      //     console.log('触发了ol.Map的鼠标移动事件：pointermove');
      //     // console.log(map.getView().getCenter());
      // });
  
      // 响应拖拽事件
      // map.on('pointerdrag', function(event){
      //     console.log('触发了ol.Map的拖拽事件：pointerdrag');
      // });

      // 地图开始移动事件
      // map.on('movestart', function(event){
      //     console.log('触发了ol.Map的地图移动事件：movestart');
      // });

      // 地图移动结束事件
      // map.on('moveend', function(event){
      //     console.log('触发了ol.Map的地图移动事件：moveend');
      //     console.log(map.getView().getCenter());
      // });

      // // 渲染地图框前触发
      // map.on('prerender', function(event){
      //     console.log('触发了ol.Map的地图渲染前事件：prerender');
      // });

      // // 渲染地图框后触发
      // map.on('postrender', function(event){
      //     console.log('触发了ol.Map的地图渲染后事件：postrender');
      // });

      //渲染图层之前触发，事件对象将没有上下文设置
      // map.on('precompose', function(event){
      //     console.log('触发了ol.Map的图层渲染之前触发：precompose');
      // });

      // 渲染图层后触发，事件对象将没有上下文设置
      // map.on('postcompose', function(event){
      //     console.log('触发了ol.Map的图层渲染完成后触发：postcompose');
      // });

      // 所有资源加载完成后触发，事件对象将没有上下文设置
      // map.on('rendercomplete', function(event){
      //     console.log('触发了ol.Map的所有图层渲染完成后触发：rendercomplete');
      // });
    },
    controlLayerVisible(){
      var layer = this.getLayerByName("pointLayer");
      if(layer){
        setInterval(() => {
          layer.setVisible(!layer.getVisible());
        }, 1000);
      }
    },
    getLayerByName(name){
      var layer = null;
      this.map.getLayers().forEach(element => {
          if(element.values_.layerName === name){
            layer = element;
          }
      });
      return layer;
    },
    addGeoJson(){

      var image = new CircleStyle({
        radius: 5,
        fill: null,
        stroke: new Stroke({color: 'red', width: 1})
      });

      var styles = {
      'Point': new Style({
        image: image
      }),
      'LineString': new Style({
        stroke: new Stroke({
          color: 'green',
          width: 1
        })
      }),
      'MultiLineString': new Style({
        stroke: new Stroke({
          color: 'green',
          width: 1
        })
      }),
      'MultiPoint': new Style({
        image: image
      }),
      'MultiPolygon': new Style({
        stroke: new Stroke({
          color: 'yellow',
          width: 1
        }),
        fill: new Fill({
          color: 'rgba(255, 255, 0, 0.5)'
        })
      }),
      'Polygon': new Style({
        stroke: new Stroke({
          color: 'blue',
          lineDash: [4],
          width: 3
        }),
        fill: new Fill({
          color: 'rgba(0, 0, 0, 0.5)'
        })
      }),
      'GeometryCollection': new Style({
        stroke: new Stroke({
          color: 'magenta',
          width: 2
        }),
        fill: new Fill({
          color: 'magenta'
        }),
        image: new CircleStyle({
          radius: 10,
          fill: null,
          stroke: new Stroke({
            color: 'magenta'
          })
        })
      }),
      'Circle': new Style({
        stroke: new Stroke({
          color: 'red',
          width: 2
        }),
        fill: new Fill({
          color: 'rgba(255,0,0,0.2)'
        })
      })
    };

    var styleFunction = function(feature) {
      return styles[feature.getGeometry().getType()];
    };

      console.log(area.features[0].geometry.coordinates[0]);

      // var polygon = turf.polygon(area.features[0].geometry.coordinates[0]);
      var polygon = area.features[0];

      // 66.270，9.229 >> 148.359，63.896
      // 113.005，21.882 >> 115.653，23.521

      var mask = turf.polygon([[[113.005, 21.882], [115.653, 21.882], [115.653, 23.521], [113.005, 23.521], [113.005,21.882]]]);

      var masked = turf.mask(polygon, mask);

      console.log(masked);


      // var vectorSource = new VectorSource({
      //   features: (new GeoJSON()).readFeatures(area
      //   ,
      //   {
      //     dataProjection:"EPSG:4326",
      //     featureProjection:"EPSG:4326"
      //   }
      //   )
      // });

      var vectorSource = new VectorSource({
        features:[new GeoJSON().readFeature(masked)]
      }); 

      var vectorLayer = new VectorLayer({
        source: vectorSource,
        style: styleFunction
      });

      this.map.addLayer(vectorLayer);

    }
  }
};
</script>

<style>
#map {
  height: 100%;
}
/*隐藏ol的一些自带元素*/
.ol-attribution,
.ol-zoom {
  display: none;
}
</style>