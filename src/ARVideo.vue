<template>
  <div class="ar-container">
    <div ref="container" class="ar-viewport"></div>
    <button @click="startAR">Iniciar AR</button>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import {MindARThree} from 'mind-ar/dist/mindar-image-three.prod.js';
import * as THREE from 'three';

const container = ref(null);
let mindarThree = null;

const startAR = async () => {
  console.log("Iniciando AR...");
  // Creamos la instancia de MindAR (modo Three.js)
  mindarThree = new MindARThree({
    container: container.valueOf(),
    // Ruta al fichero .mind que hayas generado con la imagen target
    imageTargetSrc: '/targets.mind',
    maxTrack: 1,
  });

  const { renderer, scene, camera } = mindarThree;

  // Añade una luz básica
  const light = new THREE.HemisphereLight(0xffffff, 0xbbbbff, 1);
  scene.add(light);

  // Creamos un anchor para el target 0
  const anchor = mindarThree.addAnchor(0);

  // Creamos un plano donde proyectar el vídeo
  const geometry = new THREE.PlaneGeometry(1, 0.5);
  const video = document.createElement('video');
  video.src = '/fiesta.mp4';   // Asegúrate de que tu vídeo se llame así y esté en /public
  video.loop = true;
  video.muted = true;
  video.playsInline = true;

  const texture = new THREE.VideoTexture(video);
  const material = new THREE.MeshBasicMaterial({ map: texture });
  const plane = new THREE.Mesh(geometry, material);
  anchor.group.add(plane);

  // Iniciamos MindAR
  await mindarThree.start();
  await video.play();

  // Bucle de render
  const clock = new THREE.Clock();
  const animate = () => {
    const delta = clock.getDelta();
    renderer.render(scene, camera);
    requestAnimationFrame(animate);
  };
  animate();
};
</script>

<style scoped>
.ar-container {
  width: 100vw;
  height: 100vh;
  position: relative;
  background: #000;
}

.ar-viewport {
  width: 100%;
  height: 100%;
  position: absolute;
  left: 0; top: 0;
}

button {
  position: absolute;
  z-index: 999;
  left: 50%;
  transform: translateX(-50%);
  bottom: 20px;
  padding: 10px 20px;
}
</style>
