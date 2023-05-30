<template>
  <div></div>
</template>

<script>
import * as THREE from "three";
import proj4 from "proj4";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import * as d3 from "d3";
import Stats from "three/examples/jsm/libs/stats.module.js";
// import { operation } from "retry";

export default {
  name: "HelloWorld",
  data() {
    return {
      scene: null,
      camera: null,
      renderer: null,
      map: null,
      // axesHelper:null,
      directionalLight: null,
      light: null,
      controls: null,
      stats: null,
      // textureLoader:null,
      loader: null,
      jsonData: null,
      deltaTime: null,
    };
  },
  mounted() {
    this.initFPS();
    this.initTHREE();
    this.addJsonData();
    this.animate();
  },
  methods: {
    // 显示帧率的窗口
    initFPS() {
      this.stats = new Stats();
      document.body.appendChild(this.stats.dom);
    },
    // 初始化场景，{相机、场景、渲染器}
    initTHREE() {
      this.scene = new THREE.Scene();
      this.camera = new THREE.PerspectiveCamera(
        90,
        window.innerHeight / window.innerWidth,
        0.1,
        1000
      );
      this.camera.position.set(0, 0, 30);
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.scene.add(this.camera);
      // 加入坐标轴
      this.axesHelper = new THREE.AxesHelper(5);
      this.scene.add(this.axesHelper);
      // 加载纹理
      this.map = new THREE.Object3D();
      this.directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
      this.scene.add(this.directionalLight);
      this.light = new THREE.AmbientLight(0xffffff, 0.5);
      this.scene.add(this.light);
      // 加载渲染器
      this.renderer = new THREE.WebGLRenderer({ alpha: true });
      this.renderer.setSize(window.innerWidth, window.innerHeight);
      // 将渲染器添加到body
      document.body.appendChild(this.renderer.domElement);
      // 初始化控制器 可以旋转
      this.controls = new OrbitControls(this.camera, this.renderer.domElement);
      // 创建纹理加载器对象
      // this.textureLoader = new THREE.TextureLoader();
    },
    animate() {
      this.controls.update();
      this.stats.update();
      const clock = new THREE.Clock();
      this.deltaTime = clock.getDelta();
      requestAnimationFrame(this.animate);
      this.renderer.render(this.scene, this.camera);
    },
    addJsonData() {
      this.loader = new THREE.FileLoader();
      this.loader.load("/edge.geojson", (data) => {
        this.jsonData = JSON.parse(data);
        // this.operationData(this.jsonData)
        const projection1 = d3
          .geoMercator()
          .center([116.5, 39.8])
          .scale(5)
          .translate([0, 0]);
        const features = this.jsonData.features;
        const edge = new THREE.Object3D();

        features.forEach((feature, index) => {
          const coordinates = feature.geometry.coordinates;

          const mesh = this.drawExtrudeMesh(coordinates, projection1, index);
          mesh.scale.set(20, 20, 1);
          edge.add(mesh);

          this.map.add(edge);
        });
        this.scene.add(this.map);
      });
    },
    UTMtoWGS(easting, northing) {
      //定义 WGS84 投影和 UTM 投影，其中 zone 为 UTM 带号
      const wgs84Projection =
        "+proj=longlat +ellps=WGS84 +datum=WGS84 +no_defs";
      const utmProjection =
        "+proj=utm +zone=" +
        50 +
        " +ellps=WGS84 +datum=WGS84 +units=m +no_defs";

      //创建转换对象，并进行坐标转换
      const conversion = proj4(utmProjection, wgs84Projection);
      console.log(easting, northing);
      const lngLat = conversion.forward([easting, northing]);
      console.log(lngLat);
      return [lngLat[0], lngLat[1]];
    },
    drawExtrudeMesh(coordinates, projection, index) {
      const shape = new THREE.Shape();

      console.log(coordinates);
      coordinates.forEach((coordinate, i) => {
        const wgs = this.UTMtoWGS(coordinate[0], coordinate[1]);
        const [x, y] = projection(wgs);
        console.log(wgs);
        // const [x, y] = projection(coordinate);
        console.log(x, -y);
        if (i === 0) {
          shape.moveTo(x, -y);
        }
        shape.lineTo(x, -y);
      });

      // console.log(projection(polygon));

      // 拉伸
      const geometry = new THREE.ExtrudeGeometry(shape, {
        depth: 0,
        bevelEnabled: true,
      });
      const randomColor = (0.5 + index * 5000) * 0xffffff;
      const material = new THREE.MeshBasicMaterial({
        color: randomColor,
        transparent: true,
        opacity: 0.5,
      });

      return new THREE.Line(geometry, material);
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped></style>
