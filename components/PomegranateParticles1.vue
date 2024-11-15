<template>
  <div ref="canvasContainer" class="canvas-container">
    <canvas ref="canvas"></canvas>
  </div>
</template>

<script setup>
  import { ref, onMounted, onUnmounted, nextTick } from "vue";
  import random from "canvas-sketch-util/random";

  const canvasContainer = ref(null);
  const canvas = ref(null);
  const particles = ref([]);
  const cursor = ref({ x: 9999, y: 9999 });
  let animationFrameId = null;
  let ctx = null;
  let arilImage = null;

  const loadArilImage = () => {
    return new Promise((resolve) => {
      const img = new Image();
      img.src = "/images/arilo.png";
      img.onload = () => resolve(img);
    });
  };

  class Particle {
    constructor({ x, y, radius = 10, clusterCenter }) {
      // Inicializar partícula directamente en su posición del cluster
      this.x = clusterCenter.x + random.range(-15, 15);
      this.y = clusterCenter.y + random.range(-15, 15);
      this.ax = 0;
      this.ay = 0;
      this.vx = 0;
      this.vy = 0;
      this.clusterCenter = clusterCenter;
      this.rotation = random.range(0, Math.PI * 2);
      this.radius = radius;
      this.minDist = 60;
      this.pushFactor = 0.003;
      this.cohesionFactor = 0.002;
      this.dampFactor = 0.95;
      this.scale = random.range(0.9, 1.1);
      this.initPos = { x: this.x, y: this.y };
    }

    update() {
      const dx = this.clusterCenter.x - this.x;
      const dy = this.clusterCenter.y - this.y;

      this.ax = dx * this.cohesionFactor;
      this.ay = dy * this.cohesionFactor;

      const cursorDx = this.x - cursor.value.x;
      const cursorDy = this.y - cursor.value.y;
      const dd = Math.sqrt(cursorDx * cursorDx + cursorDy * cursorDy);

      if (dd < this.minDist) {
        const distDelta = this.minDist - dd;
        this.ax += (cursorDx / dd) * distDelta * this.pushFactor;
        this.ay += (cursorDy / dd) * distDelta * this.pushFactor;
      }

      this.vx += this.ax;
      this.vy += this.ay;
      this.vx *= this.dampFactor;
      this.vy *= this.dampFactor;

      this.x += this.vx;
      this.y += this.vy;
      this.rotation += this.vx * 0.05;
    }

    draw(context) {
      if (!arilImage) return;

      context.save();
      context.translate(this.x, this.y);
      context.rotate(this.rotation);

      const width = this.radius * 2 * this.scale;
      const height = width * (arilImage.height / arilImage.width);

      context.drawImage(arilImage, -width / 2, -height / 2, width, height);
      context.restore();
    }
  }

  const initCanvas = async () => {
    if (!canvas.value || !canvasContainer.value) return;

    const canvasEl = canvas.value;
    ctx = canvasEl.getContext("2d");
    arilImage = await loadArilImage();

    const resize = () => {
      if (!canvasContainer.value) return;

      const container = canvasContainer.value;
      const { width, height } = container.getBoundingClientRect();

      // Ajustar el tamaño del canvas en píxeles reales
      canvasEl.width = width * window.devicePixelRatio;
      canvasEl.height = height * window.devicePixelRatio;

      // Asegurar que las dimensiones CSS coincidan
      canvasEl.style.width = `${width}px`;
      canvasEl.style.height = `${height}px`;

      ctx.setTransform(1, 0, 0, 1, 0, 0); // Restablece cualquier escala previa
      ctx.scale(window.devicePixelRatio, window.devicePixelRatio);

      // Reiniciar partículas después del ajuste
      if (arilImage) {
        initParticles(width, height);
      }
    };

    await resize();
    window.addEventListener("resize", resize);
    return () => window.removeEventListener("resize", resize);
  };

  const initParticles = (width, height) => {
    particles.value = [];
    const maxRadius = 20;
    const minRadius = 15;

    const clusters = [
      { x: width * 0.3, y: height * 0.5, size: 4 },
      { x: width * 0.7, y: height * 0.5, size: 5 },
    ];

    clusters.forEach((cluster) => {
      for (let i = 0; i < cluster.size; i++) {
        particles.value.push(
          new Particle({
            x: cluster.x,
            y: cluster.y,
            radius: random.range(minRadius, maxRadius),
            clusterCenter: { x: cluster.x, y: cluster.y },
          })
        );
      }
    });
  };

  const animate = () => {
    if (!canvas.value || !ctx) return;

    ctx.fillStyle = "white";
    ctx.fillRect(0, 0, canvas.value.width, canvas.value.height);

    particles.value.forEach((particle) => {
      particle.update();
      particle.draw(ctx);
    });

    animationFrameId = requestAnimationFrame(animate);
  };

  const onMouseMove = (e) => {
    if (!canvas.value) return;
    const rect = canvas.value.getBoundingClientRect();
    cursor.value = {
      x: ((e.clientX - rect.left) / rect.width) * canvas.value.width,
      y: ((e.clientY - rect.top) / rect.height) * canvas.value.height,
    };
  };

  const onMouseLeave = () => {
    cursor.value = { x: 9999, y: 9999 };
  };

  onMounted(async () => {
    await nextTick();
    await initCanvas();

    if (canvas.value) {
      canvas.value.addEventListener("mousemove", onMouseMove);
      canvas.value.addEventListener("mouseleave", onMouseLeave);
      animate();
    }
  });

  onUnmounted(() => {
    if (animationFrameId) {
      cancelAnimationFrame(animationFrameId);
    }
    if (canvas.value) {
      canvas.value.removeEventListener("mousemove", onMouseMove);
      canvas.value.removeEventListener("mouseleave", onMouseLeave);
    }
  });
</script>

<style scoped>
  .canvas-container {
    width: 100%;
    height: 200px;
    position: relative;
    margin: 2rem 0;
  }

  canvas {
    width: 100%;
    height: 100%;
    display: block;
  }

  @media (min-width: 1080px) {
    .canvas-container {
      height: 250px;
    }
  }
</style>
