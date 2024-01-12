<script setup>
import {ref, onMounted} from 'vue';
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { GUI } from 'three/examples/jsm/libs/lil-gui.module.min.js';
const webGL = ref(null);
let scene,camera,renderer
let control = null
// 初始化
function initScene(){
    scene = new THREE.Scene();
}
function initCamera(){
    camera = new THREE.PerspectiveCamera(
        75,
        webGL.value.clientWidth/webGL.value.clientHeight,
        1,
        1000
    )
    camera.position.set(0,2,11)
}
function initRenderer(){
    renderer = new THREE.WebGLRenderer({
        antialias:true,
    })
    renderer.setSize(webGL.value.clientWidth,webGL.value.clientHeight)
    webGL.value.appendChild(renderer.domElement);
}
function initAxesHelper(){
    const axesHelper = new THREE.AxesHelper(10);
    scene.add(axesHelper);
}

function initControls(){
    control = new OrbitControls(camera,renderer.domElement);
}
// 物体
const params = {
    count:90000,
    size:0.01,
    branch:5,
    radius:7,
    spin:1,//随半径增长的弯曲程度
    diffuse:0.5,//扩散程度
    pow:3,
    innerColor:0xff6030,
    outerColor:0x1b3984,
    colorDeep:0.59,
}
let positions = null;
let colors = null;
let material = null;
let geometry = null;
let points = null;
function createGalaxy(){
    if(positions){
        material.dispose();
        geometry.dispose();
        scene.remove(points);
    }
    positions = new Float32Array(params.count*3);
    colors = new Float32Array(params.count*3);
    for(let i = 0;i<params.count;i++){
        const r = Math.random()*params.radius;
        const branchAngle = (i%params.branch)*(Math.PI*2/params.branch);
        const spinAngle = r*params.spin;
        const randomX = Math.pow(Math.random(),params.pow)*(Math.random()<0.5? -1:1) * params.diffuse * r;
        const randomY = Math.pow(Math.random(),params.pow)*(Math.random()<0.5? -1:1) * params.diffuse * r;
        const randomZ = Math.pow(Math.random(),params.pow)*(Math.random()<0.5? -1:1) * params.diffuse * r;
        positions[i*3] = Math.sin(branchAngle+spinAngle)*r+randomX;
        positions[i*3+1] = randomY;
        positions[i*3+2] = Math.cos(branchAngle+spinAngle)*r+randomZ;
        
        const colorInside = new THREE.Color(params.innerColor);
        const colorOutside = new THREE.Color(params.outerColor);
        const mixedColor = colorInside.clone().lerp(colorOutside,r/(params.radius*params.colorDeep));
        colors[i*3] = mixedColor.r;
        colors[i*3+1] = mixedColor.g;
        colors[i*3+2] = mixedColor.b;
    }
    const positionAttribute = new THREE.BufferAttribute(positions,3);
    const colorsAttribute = new THREE.BufferAttribute(colors,3);
    geometry = new THREE.BufferGeometry();
    geometry.setAttribute('position', positionAttribute);
    geometry.setAttribute('color',colorsAttribute);
    material = new THREE.PointsMaterial({
        vertexColors:true,
        size:params.size,
        blending:THREE.AdditiveBlending,
    })
    points = new THREE.Points(geometry,material)
    scene.add(points);

}
function initGui(){
    const gui = new GUI({width:300});
    gui.domElement.id = 'gui';
    const f = gui.addFolder('配置');
    f.add(params,'count').min(1000).max(100000).step(1000).onFinishChange(createGalaxy);
    f.add(params,'size').min(0.01).max(1).step(0.01).onFinishChange(createGalaxy);
    f.add(params,'branch').min(2).max(20).step(1).onFinishChange(createGalaxy);
    f.add(params,'radius').min(5).max(20).step(1).onFinishChange(createGalaxy);
    f.add(params,'spin').min(0.01).max(2).step(0.01).onFinishChange(createGalaxy);
    f.add(params,'diffuse').min(0.1).max(2).step(0.1).onFinishChange(createGalaxy);
    f.add(params,'pow').min(1).max(10).step(1).onFinishChange(createGalaxy);
    f.add(params,'colorDeep').min(0.1).max(1).step(0.01).onFinishChange(createGalaxy);
    f.open();
}
// 渲染
function render(){
    
    if(control){
        control.update()
    }
    renderer.render(scene,camera);
    points.rotation.y+=0.001;
    requestAnimationFrame(render)
}
onMounted(()=>{
    //初始化
    initScene();
    initCamera();
    initRenderer();
    // initAxesHelper();
    initControls();
    initGui();
    //创建星系
    createGalaxy();
    //渲染
    render();

})
</script>

<template>
  <div ref="webGL" class="contain">
  </div>
</template>

<style lang="scss" scoped>
.contain{
    width:100vw;
    height:100vh;
    :deep(#gui){
        position:absolute;
        top:0;
        right:0;
    }
}
</style>
