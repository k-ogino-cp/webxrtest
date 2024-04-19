<template>
 <canvas id="myCanvas"></canvas>
</template>

<script>
export default{
    name:'WebXRTest'
}

import * as THREE from 'three';
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { VRButton } from "three/examples/jsm/webxr/VRButton.js";
import { XRControllerModelFactory } from 'three/examples/jsm/webxr/XRControllerModelFactory.js';		
import { XRHandModelFactory } from 'three/examples/jsm/webxr/XRHandModelFactory.js';

window.addEventListener("DOMContentLoaded", init);

function init(){
    const width = 960;
    const hight = 540;

    const canvasElement = document.querySelector('#myCanvas');
    const renderer = new THREE.WebGLRenderer({
        antialias:true,
        canvas:canvasElement
    });
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(width,hight);
    renderer.xr.enabled = true;

    const scene = new THREE.Scene();

    const camera = new THREE.PerspectiveCamera(50,width/hight,0.1,10);
    camera.position.set(0,1.6,3);

    const controls = new OrbitControls(camera,canvasElement);
    controls.enableDamping = true;
    controls.dampingFactor = 0.2;
    controls.target.set(0,1.6,0);
    controls.update();

    const floorGeometry = new THREE.PlaneGeometry( 4, 4 );
    const floorMaterial = new THREE.MeshStandardMaterial( { color: 0x666666 } );
    const floor = new THREE.Mesh( floorGeometry, floorMaterial );
    floor.rotation.x = - Math.PI / 2;
    floor.receiveShadow = true;
    scene.add( floor );

    const ambientlight = new THREE.AmbientLight(0xffffff);
    ambientlight.intensity = 0.5;
    scene.add(ambientlight);

    const directionallight = new THREE.DirectionalLight(0xffffff);
    directionallight.intensity=1;
    directionallight.position.set(1,6,1);
    scene.add(directionallight);

    const sessionInit = 
    {
        requiredFeatures: [ 'hand-tracking' ]
    };

    document.body.appendChild(VRButton.createButton(renderer,sessionInit));

    const controller1 = renderer.xr.getController( 0 );
	scene.add( controller1 );

	const controller2 = renderer.xr.getController( 1 );
	scene.add( controller2 );

    const controllerModelFactory = new XRControllerModelFactory();
	const handModelFactory = new XRHandModelFactory();

    const controllerGrip1 = renderer.xr.getControllerGrip(0);
    controllerGrip1.add(controllerModelFactory.createControllerModel(controllerGrip1));
    scene.add(controllerGrip1);

    const hand1 = renderer.xr.getHand(0);
    hand1.add(handModelFactory.createHandModel(hand1));
    scene.add(hand1);

    const controllerGrip2 = renderer.xr.getControllerGrip(1);
    controllerGrip2.add(controllerModelFactory.createControllerModel(controllerGrip2));
    scene.add(controllerGrip2);

    const hand2 = renderer.xr.getHand(1);
    hand2.add(handModelFactory.createHandModel(hand2));
    hand2.addEventListener('pinchstart',event=>{
        const geometry = new THREE.BoxGeometry( 1, 1, 1 );
        const pinchPosition = event.pinchPosition;
        geometry.position.copy(pinchPosition);
        const material = new THREE.MeshBasicMaterial({ color: 0xff0000 });
        const cube = new THREE.Mesh(geometry, material);
        scene.add(cube);
    })
    scene.add(hand2);

    renderer.setAnimationLoop(tick);

    function tick(){
        renderer.render(scene,camera);
    }
}
</script>