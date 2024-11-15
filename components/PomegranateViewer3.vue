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

    // Crear forma base
    const sphereGeometry = new THREE.SphereGeometry(2, 32, 32);
    // Agregar deformaciones
    const vertices = sphereGeometry.attributes.position;
    for (let i = 0; i < vertices.count; i++) {
      const x = vertices.getX(i);
      const y = vertices.getY(i);
      const z = vertices.getZ(i);

      vertices.setXYZ(
        i,
        x + Math.random() * 0.2,
        y + Math.random() * 0.2,
        z + Math.random() * 0.2
      );
    }

    // Cargar y aplicar textura
    const textureLoader = new THREE.TextureLoader();
    const material = new THREE.MeshStandardMaterial({
      map: textureLoader.load("/images/pomeTexture.webp"), // Reemplaza con tu ruta
      roughness: 0.8,
      metalness: 0.1,
    });

    const fruit = new THREE.Mesh(sphereGeometry, material);

    const fruitGroup = new THREE.Group();
    fruitGroup.add(fruit);
    scene.add(fruitGroup);

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
      fruitGroup.rotation.y += 0.005;
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
