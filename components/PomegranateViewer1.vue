<!-- components/PomegranateViewer.vue -->
<template>
  <div class="viewer w-full h-screen flex flex-col">
    <div ref="container" class="flex-1"></div>
  </div>
</template>

<script setup>
  import * as THREE from "three";
  import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
  import { onMounted, onBeforeUnmount, ref } from "vue";

  const container = ref(null);
  let camera, scene, renderer, controls;

  const init = () => {
    if (!container.value) return;

    scene = new THREE.Scene();
    scene.background = new THREE.Color(0xf5f5f5);

    camera = new THREE.PerspectiveCamera(
      75,
      container.value.clientWidth / container.value.clientHeight,
      0.1,
      1000
    );
    camera.position.set(0, 0, 10);

    renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(container.value.clientWidth, container.value.clientHeight);
    container.value.appendChild(renderer.domElement);

    // Crear forma de granada
    const sphereGeometry = new THREE.SphereGeometry(3, 32, 32);
    const texture = new THREE.TextureLoader().load(
      "/images/pomeTexture.webp"
    );
    const material = new THREE.MeshStandardMaterial({
      color: 0xc41e3a,
      roughness: 0.7,
      metalness: 0.1,
      map: texture,
    });

    const pomegranate = new THREE.Mesh(sphereGeometry, material);

    // Corona de la granada
    const crownGeometry = new THREE.CylinderGeometry(0.5, 1, 1, 6);
    const crownMaterial = new THREE.MeshStandardMaterial({ color: 0x4a4a4a });
    const crown = new THREE.Mesh(crownGeometry, crownMaterial);
    crown.position.y = 3.5;

    // Grupo para la granada completa
    const pomegranateGroup = new THREE.Group();
    pomegranateGroup.add(pomegranate);
    pomegranateGroup.add(crown);
    scene.add(pomegranateGroup);

    // Iluminación
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
    scene.add(ambientLight);

    const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
    directionalLight.position.set(5, 5, 5);
    scene.add(directionalLight);

    controls = new OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;

    const animate = () => {
      requestAnimationFrame(animate);
      pomegranateGroup.rotation.y += 0.005;
      controls.update();
      renderer.render(scene, camera);
    };

    animate();
  };

  onMounted(() => {
    setTimeout(init, 0);
  });

  onBeforeUnmount(() => {
    if (renderer) {
      renderer.dispose();
    }
  });
</script>

<style scoped>
  .viewer {
    max-width: 320px;
    max-height: 300px;
  }
</style>
