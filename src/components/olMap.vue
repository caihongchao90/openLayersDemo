<template>
    <div id="mapDiv"></div>
    <div id="anchor"><img src="./billboard.png" alt="示例锚点"/></div>
    <div id="popup" class="ol-popup">
      <a href="#" id="popup-closer" class="ol-popup-closer"></a>
      <div id="popup-content"></div>
    </div>
    <div id="navigateContainer">
        <input type="button" @click="moveToLeft()" value="左移" />
        <input type="button" @click="moveToRight();" value="右移" />
        <input type="button" @click="moveToUp();" value="上移" />
        <input type="button" @click="moveToDown();" value="下移" />
        <input type="button" @click="moveToChengDu();" value="移到成都" />
        <input type="button" @click="zoomIn();" value="放大" />
        <input type="button" @click="zoomOut();" value="缩小" />
        <input type="button" @click="showInfo();" value="查看点击位置信息" />
    </div>
</template>

<script>
import "ol/ol.css";
import { Map, View } from "ol";
import * as olProj from 'ol/proj';
import TileLayer from "ol/layer/Tile";
import OSM from "ol/source/OSM";
import XYZ from 'ol/source/XYZ.js';
import Overlay from 'ol/Overlay.js';

import VectorLayer from 'ol/layer/Vector.js';
import VectorSource from 'ol/source/Vector.js';
import Feature from 'ol/Feature.js';
import Point from 'ol/geom/Point.js';
import MultiPoint from 'ol/geom/MultiPoint.js';
import CircleStyle from 'ol/style/Circle.js';
import Fill from 'ol/style/Fill.js';
import Style from 'ol/style/Style.js';
import Icon from 'ol/style/Icon.js';

import {toStringHDMS} from 'ol/coordinate.js';

import Select from 'ol/interaction/Select.js';
import * as olEvents from 'ol/events';

import TileDebug from 'ol/source/TileDebug.js';
import { OverviewMap, ScaleLine, MousePosition, defaults } from "ol/control";
import {createStringXY} from 'ol/coordinate';
let map
export default {
  name:"olMap",
  mounted() {
    
    // 我们需要一个vector的layer来放置图标
    let layer = new VectorLayer({
      source: new VectorSource()
    })
    let layer2 = new VectorLayer({
      source: new VectorSource(),
      // 注意：把feature上的style，直接移到layer上，避免直接在feature上设置style
      style: new Style({
        image: new CircleStyle({
          radius: 10,
          fill: new Fill({
            color: 'red'
          })
        })
      })
    });
    map = new Map({
      target: "mapDiv",
      layers: [
        new TileLayer({
          source: 
          // new TileDebug({
          // // projection: 'EPSG:4326',
          // // tileGrid: tilegrid,
          // tileSize: [256, 256],
          // // extent :  projectionExtent,
          // wrapX: false
          // }),
          // new XYZ({
          //   tilePixelRatio: 2,
          //   tileUrlFunction: function(tileCoord){  // 参数tileCoord为瓦片坐标
          //       let z = tileCoord[0];
          //       let x = tileCoord[1];
          //       let y = tileCoord[2];

          //       // 计算当前层级下瓦片总数的一半，用于定位整个地图的中心点
          //       let halfTileNum = Math.pow(2, z-1);
          //       // 原点移到中心点后，计算xy方向上新的坐标位置
          //       let baiduX =  x - halfTileNum;
          //       let baiduY =  y + halfTileNum;

          //       // 百度瓦片服务url将负数使用M前缀来标识
          //       if (baiduX < 0) {
          //           baiduX = 'M' + (-baiduX);
          //       }
          //       if (baiduY < 0) {
          //           baiduY = 'M' + (-baiduY);
          //       }

          //       // 返回经过转换后，对应于百度在线瓦片的url
          //       return 'http://online2.map.bdimg.com/onlinelabel/?qt=tile&x=' + baiduX + '&y=' + baiduY + '&z=' + z + '&styles=pl&udt=20160321&scaler=2&p=0';
          //     }
          //   })
          new OSM()
        }),
        layer,
        layer2
      ],
      view: new View({
        // 设置成都为地图中心，此处进行坐标转换， 把EPSG:4326的坐标，转换为EPSG:3857坐标，因为ol默认使用的是EPSG:3857坐标
        // center: olProj.transform([104.06, 30.67], 'EPSG:4326', 'EPSG:3857'),
        // extent: [102, 29, 104, 31],
        center: [104, 30],
        projection: 'EPSG:4326',
        zoom: 17,
      }),
      
      controls: defaults().extend([
        // 添加一个鹰眼控件
        new OverviewMap({
          // 实例化一个OverviewMap类的对象，并加入到地图中
          collapsed: false,
          view: new View({
            
          }),
          layers: [
            new TileLayer({
              source: new OSM()
            })
          ]
        }),
        // 添加比例尺
        new ScaleLine({
          units: "metric"
        }),
        
        new MousePosition({
          coordinateFormat: createStringXY(6), // 保留6位小数位
        })
      ])
    });
    // 1、Feature + Style的方式
    // 创建一个Feature，并设置好在地图上的位置
    // let anchor = new Feature({
    //   geometry: new Point([104, 30])
    // });
    // // 设置样式，在样式中就可以设置图标
    // anchor.setStyle(new Style({
    //   image: new Icon({
    //     src: './billboard.png',
    //     anchor: [0.5, 0.5]
    //   })
    // }));
    // // 添加到之前的创建的layer中去
    // layer.getSource().addFeature(anchor);
    // // 监听地图层级变化
    // map.getView().on('change:resolution', function(){
    //     let style = anchor.getStyle();
    //     // 重新设置图标的缩放率，基于层级10来做缩放
    //     style.getImage().setScale(this.getZoom() / 20);
    //     anchor.setStyle(style);
    // })

    // 2、传统的overlay方法
    // 下面把上面的图标附加到地图上，需要一个ol.Overlay
    let anchor = new Overlay({
      element: document.getElementById('anchor')
    });
    // 关键的一点，需要设置附加到地图上的位置
    anchor.setPosition([104, 30]);
    // 然后添加到map上
    map.addOverlay(anchor);

    // 要素交互
    // 在地图上添加一个圆
    var circle2 = new Feature({
      geometry: new MultiPoint([[104, 31],[104, 32]])
      //new Point([104, 31])
    })
    // 此处不再为feature设置style
    layer2.getSource().addFeature(circle2);
    // 添加一个用于选择Feature的交互方式
    map.addInteraction(new Select({
      filter: function(feature, layer){
        console.log("feature, layer",feature, layer)

        return layer === layer2;
      },
      // 设置选中后的style
      style: new Style({
        image: new CircleStyle({
          radius: 10,
          fill: new Fill({
            color: 'blue'
          })
        })
      })
    }));
  },
  methods:{
    // 向左移动地图
    moveToLeft() {
      let view = map.getView();
      let mapCenter = view.getCenter();
      // 让地图中心的x值增加，即可使得地图向左移动，增加的值根据效果可自由设定
      mapCenter[0] += 50000;
      view.setCenter(mapCenter);
      map.render();
    },
    // 向右移动地图
    moveToRight() {
      let view = map.getView();
      let mapCenter = view.getCenter();
      // 让地图中心的x值减少，即可使得地图向右移动，减少的值根据效果可自由设定
      mapCenter[0] -= 50000;
      view.setCenter(mapCenter);
      map.render();
    },

    // 向上移动地图
    moveToUp() {
      let view = map.getView();
      let mapCenter = view.getCenter();
      // 让地图中心的y值减少，即可使得地图向上移动，减少的值根据效果可自由设定
      mapCenter[1] -= 50000;
      view.setCenter(mapCenter);
      map.render();
    },

    // 向下移动地图
    moveToDown() {
      let view = map.getView();
      let mapCenter = view.getCenter();
      // 让地图中心的y值增加，即可使得地图向下移动，增加的值根据效果可自由设定
      mapCenter[1] += 50000;
      view.setCenter(mapCenter);
      map.render();
    },

    // 移动到成都
    moveToChengDu() {
      let view = map.getView();
      // 设置地图中心为成都的坐标，即可让地图移动到成都
      view.setCenter(olProj.transform([104.06, 30.67], 'EPSG:4326', 'EPSG:3857'));
      map.render();
    },

    // 放大地图
    zoomIn() {
      let view = map.getView();
      // 让地图的zoom增加1，从而实现地图放大
      view.setZoom(view.getZoom() + 1);
    },

    // 缩小地图
    zoomOut() {
      let view = map.getView();
      // 让地图的zoom减小1，从而实现地图缩小
      view.setZoom(view.getZoom() - 1);
    },
    showInfo(){
      // console.log("map.getTarget()",map.getTarget())
      // console.log("map.getTargetElement()",map.getTargetElement())
      // console.log("map.getSize()",map.getSize())
      // console.log("map.getView()",map.getView())
      // console.log("map.getViewport()",map.getViewport())
      // 获取到popup的节点
      let container = document.getElementById('popup');
      let content = document.getElementById('popup-content');
      let closer = document.getElementById('popup-closer');

      // 创建一个overlay, 绑定html元素container
      let overlay = new Overlay({
        element: container,
        autoPan: true,
        autoPanAnimation: {
          duration: 250
        }
      });
      // 监听地图点击事件
      map.on('singleclick', function(evt) {
          // 获取当前点击坐标，并设置到HTML元素上去
        var coordinate = evt.coordinate;
        
        var hdms = toStringHDMS(olProj.transform(
            coordinate, 'EPSG:4326', 'EPSG:3857'));

        content.innerHTML = '<p>You clicked here:</p><code>' + hdms +
            '</code>';
        // 设置overlay的位置，从而显示在鼠标点击处
        console.log("coordinate,content.innerHTML",coordinate,content.innerHTML)
        overlay.setPosition(coordinate);
        map.addOverlay(overlay);
      });
    },


  }
};
</script>

<style scoped>
#navigateContainer{
  position: absolute;
  bottom: 50px;
  right: 30px;
}
</style>