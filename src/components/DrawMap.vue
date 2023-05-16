<template>
    <div id="mapDiv"></div>
</template>
<script>
import { Map, View } from "ol";

import {defaults} from 'ol/control/defaults';
import FullScreen from 'ol/control/FullScreen.js';
import MousePosition from 'ol/control/MousePosition.js';
import OverviewMap from 'ol/control/OverviewMap.js';
import ScaleLine from 'ol/control/ScaleLine.js';
import ZoomSlider from 'ol/control/ZoomSlider.js';
import ZoomToExtent from 'ol/control/ZoomToExtent.js';

import TileLayer from "ol/layer/Tile";
import OSM from "ol/source/OSM";
import VectorLayer from 'ol/layer/Vector.js';
import VectorSource from 'ol/source/Vector.js';
import Stroke from 'ol/style/Stroke.js';
import Draw from 'ol/interaction/Draw.js';
import Feature from 'ol/Feature.js';
import Point from 'ol/geom/Point.js';
import Style from 'ol/style/Style.js';
import Icon from 'ol/style/Icon.js';
export default({
    name:"DrawMap",
    mounted(){
		var map = new Map({
			layers: [
				new TileLayer({
					source: new OSM()
				})
			],
			controls: defaults().extend([
				new FullScreen(),
				new MousePosition(),
				new OverviewMap(),
				new ScaleLine(),
				new ZoomSlider(),
				new ZoomToExtent()
			]),
			target: 'mapDiv',
			view: new View({
				projection: 'EPSG:4326',
				center: [104, 30],
				zoom: 10
			})
		});
		// 我们需要一个vector的layer来放置图标
		var lineLayer = new VectorLayer({
			source: new VectorSource(),
			style: new Style({
				stroke: new Stroke({
					color: 'red',
					size: 1
				})
        })
			})
			map.addLayer(lineLayer);
			var lineDraw = new Draw({
        type: 'LineString',
        source: lineLayer.getSource(),    // 注意设置source，这样绘制好的线，就会添加到这个source里
        style: new Style({            // 设置绘制时的样式
					stroke: new Stroke({
						color: '#009933',
						size: 1
					})
        }),
        maxPoints: 99999    // 限制不超过4个点
    });
		lineDraw.on('drawend', function(event){
			// event.feature 就是当前绘制完成的线的Feature
			// document.getElementById('points').innerHTML = JSON.stringify(event.feature.getGeometry().getCoordinates());
			alert(JSON.stringify(event.feature.getGeometry().getCoordinates()))
		});

    map.addInteraction(lineDraw);
			
    }
})
</script>
<style scoped>

</style>