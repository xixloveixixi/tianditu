<template>
    <div id="xtMap" class="xt-map"></div>
</template>

<script>
import axios from 'axios';
import XTSData from '@/util/XTS.json';
import XiangtanData from '@/util/湘潭市.json';
export default {
    name: 'XiangtanMap',
    data() {
        return {
            map: null,
            polygonCoordinates: [], // 湘潭区域的边界坐标点
        };
    },
    mounted() {
        this.loadTiandituScript();
    },
    methods: {
        async loadTiandituScript() {
            const script = document.createElement('script');
            script.src = "http://api.tianditu.gov.cn/api?v=4.0&tk=你的key";
            script.onload = () => {
                console.log('天地图 API 脚本加载成功');
                if (typeof T === 'undefined') {
                    console.error('天地图 API 未正确加载');
                    return;
                }
                this.init();
            };
            script.onerror = () => {
                console.error('天地图 API 脚本加载失败');
            };
            document.head.appendChild(script);
        },
        async init() {
            try {
                await this.initTiandituMap(); // 初始化地图
            } catch (error) {
                console.error('Error initializing map:', error);
            }
        },
        async initTiandituMap() {
            const mapContainer = document.getElementById('xtMap');
            if (!mapContainer) {
                console.error('Map container not found');
                return;
            }
    
            const imageURL = "http://t0.tianditu.gov.cn/img_w/wmts?" +
                "SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=img&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles" +
                "&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}&tk=你的key";
    
            const imageURLT = "http://t0.tianditu.gov.cn/cia_w/wmts?" +
                "SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=cia&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles" +
                "&TILEMATRIX={z}&TILEROW={y}&TILECOL={x}" +
                "&tk=你的key";
    
            const lay = new T.TileLayer(imageURL, { minZoom: 0, maxZoom: 18 });
            const lay2 = new T.TileLayer(imageURLT, { minZoom: 0, maxZoom: 18 });
            const config = { layers: [lay, lay2] };
    
            this.map = new T.Map("xtMap", config);
            this.map.centerAndZoom(new T.LngLat(112.584052, 27.72973), 9.6);
            this.map.enableInertia();
    
            try {
                // 引入 util 目录下的 JSON 文件
    
    
                const countries = XTSData.features;
                if (countries) {
                    this.processBoundaries(countries, this.map);
                } else {
                    console.error('Boundary data features in XTS.json is undefined');
                }
    
                const shiyanCountries = XiangtanData.features;
                if (shiyanCountries) {
                    this.processBoundaries(shiyanCountries, this.map);
                } else {
                    console.error('Boundary data features in 湘潭市.json is undefined');
                }
            } catch (error) {
                console.error('Error loading boundary data:', error);
            }
        },
        processBoundaries(countries, map) {
            countries.forEach(item => {
                var districtName = item.properties.name;
                var bounds = item.geometry.coordinates[0][0];
                console.log(`${districtName} Boundary coordinates:`, bounds); // 输出坐标点数组

                var style = {
                    color: 'rgb(127,181,255)',
                    weight: 2,
                    opacity: 1,
                    lineStyle: 'solid',
                    fillColor: 'rgb(127,181,255,0.5)',
                    fillOpacity: 0.3
                };

                var points = bounds.map(line => new T.LngLat(line[0], line[1]));
                console.log(`${districtName} Polygon points:`, points); // 输出多边形的坐标点数组

                var polygon = new T.Polygon(points, style);
                // 将多边形添加到地图上
                map.addOverLay(polygon);
            });
        }
    }
};
</script>

<style>
.xt-map {
    width: 100%;
    height: 100vh;
}
</style>
