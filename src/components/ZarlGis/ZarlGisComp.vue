<template>
  <div ref="cesiumContainer" class="cesium-container">
    <div class="point"></div>
  </div>
</template>

<script>
// The URL on your server where CesiumJS's static files are hosted.
import * as Cesium from "cesium";
import "cesium/Build/Cesium/Widgets/widgets.css";
import { BaiduImageryProvider } from "@dvgis/cesium-map";

// Your access token can be found at: https://ion.cesium.com/tokens.
// Replace `your_access_token` with your Cesium ion access token.
// Ion.defaultAccessToken = "";
export default {
  name: "ZarlGisComp",
  props: {
    cesiumBaseUrl: {
      type: String,
      default: "/Cesium/",
    },
  },
  watch: {
    cesiumBaseUrl: {
      handler(val) {
        window.CESIUM_BASE_URL = val;
      },
      immediate: true,
    },
  },
  async mounted() {
    const viewer = new Cesium.Viewer(this.$refs.cesiumContainer, {
      terrain: Cesium.Terrain.fromWorldTerrain(), // 地形
      // 隐藏工具栏
      animation: false, // 底部动画控制
      baseLayerPicker: false, // 底图选择
      fullscreenButton: false, // 全屏按钮
      geocoder: false, // 地名查找
      homeButton: false, // 主页按钮
      // infoBox: false, // 信息框
      sceneModePicker: false, // 2D/3D选择
      // selectionIndicator: false, // 选中指示器
      timeline: false, // 时间线
      navigationHelpButton: false, // 导航帮助按钮
      navigationInstructionsInitiallyVisible: false, // 初始情况下是否显示导航说明
    });

    // 隐藏版权信息
    viewer.cesiumWidget.creditContainer.style.display = "none";

    // 添加百度地图影像图层
    // viewer.imageryLayers.add(
    //   new Cesium.ImageryLayer(
    //     new BaiduImageryProvider({
    //       style: "img", // style: img、vec、normal、dark
    //       crs: "WGS84", // 使用84坐标系，默认为：BD09
    //     })
    //   )
    // );

    // Example 1. USGS shaded relief tiles (KVP)
    // const shadedRelief1 = new Cesium.WebMapTileServiceImageryProvider({
    //   url: "http://basemap.nationalmap.gov/arcgis/rest/services/USGSShadedReliefOnly/MapServer/WMTS",
    //   layer: "USGSShadedReliefOnly",
    //   style: "default",
    //   format: "image/jpeg",
    //   tileMatrixSetID: "default028mm",
    //   // tileMatrixLabels : ['default028mm:0', 'default028mm:1', 'default028mm:2' ...],
    //   maximumLevel: 19,
    //   credit: new Cesium.Credit("U. S. Geological Survey"),
    // });
    // viewer.imageryLayers.addImageryProvider(shadedRelief1);

    // // Example 2. USGS shaded relief tiles (RESTful)
    // const shadedRelief2 = new Cesium.WebMapTileServiceImageryProvider({
    //   url: "http://basemap.nationalmap.gov/arcgis/rest/services/USGSShadedReliefOnly/MapServer/WMTS/tile/1.0.0/USGSShadedReliefOnly/{Style}/{TileMatrixSet}/{TileMatrix}/{TileRow}/{TileCol}.jpg",
    //   layer: "USGSShadedReliefOnly",
    //   style: "default",
    //   format: "image/jpeg",
    //   tileMatrixSetID: "default028mm",
    //   maximumLevel: 19,
    //   credit: new Cesium.Credit("U. S. Geological Survey"),
    // });
    // viewer.imageryLayers.addImageryProvider(shadedRelief2);

    const times = Cesium.TimeIntervalCollection.fromIso8601({
      iso8601: "2015-07-30/2017-06-16/P1D",
      dataCallback: function dataCallback(interval, index) {
        return {
          Time: Cesium.JulianDate.toIso8601(interval.start),
        };
      },
    });
    const weather = new Cesium.WebMapTileServiceImageryProvider({
      url: "https://gibs.earthdata.nasa.gov/wmts/epsg4326/best/AMSR2_Snow_Water_Equivalent/default/{Time}/{TileMatrixSet}/{TileMatrix}/{TileRow}/{TileCol}.png",
      layer: "AMSR2_Snow_Water_Equivalent",
      style: "default",
      tileMatrixSetID: "2km",
      maximumLevel: 5,
      format: "image/png",
      clock: clock,
      times: times,
      credit: new Cesium.Credit(
        "NASA Global Imagery Browse Services for EOSDIS"
      ),
    });
    viewer.imageryLayers.addImageryProvider(weather);

    // 设置相机位置
    viewer.camera.setView({
      destination: Cesium.Cartesian3.fromDegrees(116.3974, 39.9093, 6000000),
    });

    // 添加 primitive-tileset  http://10.10.11.200:7080/tileset.json
    const tileset = await Cesium.Cesium3DTileset.fromUrl(
      "http://10.10.11.200:7080/tileset.json"
    );
    viewer.scene.primitives.add(tileset);

    // viewer.flyTo(tileset, {
    //   duration: 1,
    //   offset: new Cesium.HeadingPitchRange(
    //     Cesium.Math.toRadians(0),
    //     Cesium.Math.toRadians(-90)
    //   ),
    // });

    // 将点添加到地图上
    // const point = viewer.entities.add({
    //   position: Cesium.Cartesian3.fromDegrees(116.3974, 39.9093),
    //   point: {
    //     pixelSize: 10,
    //     color: Cesium.Color.RED,
    //   },
    // });

    // 将 html 元素添加到地图上 116.3974, 39.9093，并且在地球背面时候不显示
    const point = document.querySelector(".point");
    viewer.scene.preRender.addEventListener(() => {
      const position = Cesium.Cartesian3.fromDegrees(
        116.3974,
        39.9093,
        1000000
      );
      // 判断点是否在地球背面
      const dot = Cesium.Cartesian3.dot(position, viewer.camera.positionWC);
      if (dot < 0) {
        point.style.display = "none";
        return;
      }
      // 将经纬度坐标转换为屏幕坐标
      const canvasPosition = Cesium.SceneTransforms.wgs84ToWindowCoordinates(
        viewer.scene,
        position
      );
      if (Cesium.defined(canvasPosition)) {
        point.style.display = "block";
        point.style.top = canvasPosition.y + "px";
        point.style.left = canvasPosition.x + "px";
      } else {
        point.style.display = "none";
      }
    });
  },
};
</script>

<style lang="scss">
.cesium-container {
  height: 100vh;
  width: 100vw;
  overflow: hidden;
  .point {
    position: absolute;
    top: 0;
    left: 0;
    height: 10px;
    width: 10px;
    background-color: red;
    transform: translate(-50%, -50%);
    z-index: 999;
  }
}
</style>
