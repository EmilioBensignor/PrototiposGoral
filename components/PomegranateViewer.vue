<template>
  <div class="viewer">
    <div ref="container" class="render-container"></div>
  </div>
</template>

<script setup>
  import * as THREE from "three";
  import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
  import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
  import { onMounted, onBeforeUnmount, ref } from "vue";

  const container = ref(null);
  let camera, scene, renderer, controls;

  const init = () => {
    // Configuración básica
    scene = new THREE.Scene();
    scene.background = new THREE.Color(0xf5f5f5);

    // Configurar el renderer con dimensiones específicas
    renderer = new THREE.WebGLRenderer({
      antialias: true,
      powerPreference: "high-performance",
    });
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(320, 300); // Dimensiones fijas
    container.value.appendChild(renderer.domElement);

    // Configurar cámara con aspect ratio fijo
    camera = new THREE.PerspectiveCamera(75, 320 / 300, 0.1, 1000);
    camera.position.set(0, 0, 4); // Ajustado para mantener modelo visible

    const modelGroup = new THREE.Group();
    scene.add(modelGroup);

    // Cargar modelo con posición y escala controladas
    const loader = new GLTFLoader();
    loader.load("/models/fruit.glb", (gltf) => {
      const model = gltf.scene;

      // Centrar modelo
      const box = new THREE.Box3().setFromObject(model);
      const center = box.getCenter(new THREE.Vector3());
      model.position.set(-center.x, -center.y, -center.z);

      // Escala fija para llenar el viewport
      model.scale.setScalar(1.5);

      modelGroup.add(model);
    });

    // Iluminación
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.7);
    scene.add(ambientLight);

    const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
    directionalLight.position.set(5, 5, 5);
    scene.add(directionalLight);

    // Controles limitados
    controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    controls.enableZoom = false;
    controls.enablePan = false;
    controls.minPolarAngle = Math.PI / 3;
    controls.maxPolarAngle = Math.PI / 1.5;

    const animate = () => {
      requestAnimationFrame(animate);
      modelGroup.rotation.y += 0.005;
      controls.update();
      renderer.render(scene, camera);
    };

    animate();
  };

  onMounted(init);

  onBeforeUnmount(() => {
    if (renderer) {
      renderer.dispose();
    }
  });
</script>

<style scoped>
  .viewer {
    width: 320px;
    height: 300px;
    overflow: hidden;
    position: relative;
  }

  .render-container {
    width: 100%;
    height: 100%;
  }
</style>
