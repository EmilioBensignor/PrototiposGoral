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

  // Carga la imagen del aril
  const loadArilImage = () => {
    return new Promise((resolve) => {
      const img = new Image();
      img.src = "/images/arilo.png";
      img.onload = () => resolve(img);
    });
  };

  class Particle {
    constructor({ x, y, radius = 10 }) {
      this.x = x;
      this.y = y;
      this.ax = 0;
      this.ay = 0;
      this.vx = 0;
      this.vy = 0;
      this.ix = x;
      this.iy = y;
      this.rotation = random.range(0, Math.PI * 2);

      this.radius = radius * 1.1;
      this.minDist = random.range(100, 200);
      this.pushFactor = random.range(0.003, 0.005);
      this.pullFactor = random.range(0.002, 0.006);
      this.dampFactor = random.range(0.9, 0.95);
      this.stabilityFactor = 0.000002;
      this.minVelocityThreshold = 0.005;

      // Escala aleatoria para variedad visual
      this.scale = random.range(0.8, 1.2);
    }

    update() {
      // [El código de update permanece igual]
      let dx, dy, dd, distDelta;

      dx = this.ix - this.x;
      dy = this.iy - this.y;

      this.ax = dx * this.pullFactor;
      this.ay = dy * this.pullFactor;

      dx = this.x - cursor.value.x;
      dy = this.y - cursor.value.y;
      dd = Math.sqrt(dx * dx + dy * dy);

      distDelta = this.minDist - dd;

      if (dd < this.minDist) {
        this.ax += (dx / dd) * distDelta * this.pushFactor;
        this.ay += (dy / dd) * distDelta * this.pushFactor;
      }

      const time = performance.now() * 0.001;
      this.ax += Math.sin(time + this.rotation) * this.stabilityFactor;
      this.ay += Math.cos(time + this.rotation) * this.stabilityFactor;

      this.vx += this.ax;
      this.vy += this.ay;

      this.vx *= this.dampFactor;
      this.vy *= this.dampFactor;

      if (
        Math.abs(this.vx) < this.minVelocityThreshold &&
        Math.abs(this.vy) < this.minVelocityThreshold
      ) {
        this.vx = 0;
        this.vy = 0;
      }

      this.x += this.vx;
      this.y += this.vy;

      // Límites del canvas
      const canvas = document.querySelector("canvas");
      const bounds = {
        left: this.radius,
        right: canvas.width - this.radius,
        top: this.radius,
        bottom: canvas.height - this.radius,
      };

      if (this.x < bounds.left) {
        this.x = bounds.left;
        this.vx *= -0.7;
      } else if (this.x > bounds.right) {
        this.x = bounds.right;
        this.vx *= -0.7;
      }

      if (this.y < bounds.top) {
        this.y = bounds.top;
        this.vy *= -0.7;
      } else if (this.y > bounds.bottom) {
        this.y = bounds.bottom;
        this.vy *= -0.7;
      }

      this.rotation += this.vx * 0.05;
    }

    draw(context) {
      if (!arilImage) return;

      context.save();
      context.translate(this.x, this.y);
      context.rotate(this.rotation);

      // Calcula dimensiones para mantener la proporción de la imagen
      const width = this.radius * 2 * this.scale;
      const height = width * (arilImage.height / arilImage.width);

      // Dibuja la imagen centrada
      context.drawImage(arilImage, -width / 2, -height / 2, width, height);

      context.restore();
    }
  }

  const initCanvas = async () => {
    if (!canvas.value || !canvasContainer.value) return;

    const canvasEl = canvas.value;
    ctx = canvasEl.getContext("2d");

    // Carga la imagen antes de inicializar
    arilImage = await loadArilImage();

    const resize = () => {
      if (!canvasContainer.value) return;
      const container = canvasContainer.value;
      const { width, height } = container.getBoundingClientRect();
      canvasEl.width = width;
      canvasEl.height = height;
      initParticles(width, height);
    };

    resize();
    window.addEventListener("resize", resize);
    return () => window.removeEventListener("resize", resize);
  };

  const initParticles = (width, height) => {
    particles.value = [];

    const maxRadius = 18;
    const minRadius = 12;
    const margin = maxRadius * 2; // Aumentamos el margen para que no toque los bordes

    const bandHeight = height * 1; // Reducimos la altura de la banda
    const bandTop = (height - bandHeight) / 2;

    const createCluster = (centerX, centerY, numParticles) => {
      const clusterRadius = {
        x: 40, // Más compacto horizontalmente
        y: 80, // Más compacto verticalmente
      };

      for (let i = 0; i < numParticles; i++) {
        const angle = random.range(0, Math.PI * 2);
        const x = centerX + Math.cos(angle) * random.range(0, clusterRadius.x);
        const y = centerY + Math.sin(angle) * random.range(0, clusterRadius.y);

        const safeX = Math.max(margin, Math.min(width - margin, x));
        const safeY = Math.max(
          bandTop + margin,
          Math.min(bandTop + bandHeight - margin, y)
        );

        const radius = random.range(minRadius, maxRadius);
        const particle = new Particle({ x: safeX, y: safeY, radius });
        particles.value.push(particle);
      }
    };

    const numClusters = 5; // Más grupos para cubrir mejor el espacio
    const spacing = (width - 2 * margin) / (numClusters - 1);

    for (let i = 0; i < numClusters; i++) {
      const centerX = margin + spacing * i;
      const baseY = height * 0.5;

      const verticalOffset = i % 2 === 0 ? -10 : 10;
      const centerY = baseY + verticalOffset;

      const clusterSize = Math.ceil(
        random.range(
          i === 0 || i === numClusters - 1 ? 3 : 4,
          i === 0 || i === numClusters - 1 ? 5 : 7
        )
      );

      createCluster(centerX, centerY, clusterSize);
    }
  };

  const animate = () => {
    if (!canvas.value || !ctx) return;

    const canvasEl = canvas.value;
    ctx.fillStyle = "white";
    ctx.fillRect(0, 0, canvasEl.width, canvasEl.height);

    particles.value.forEach((particle) => {
      particle.update();
      particle.draw(ctx);
    });

    animationFrameId = requestAnimationFrame(animate);
  };

  const onMouseMove = (e) => {
    if (!canvas.value) return;

    const canvasEl = canvas.value;
    const rect = canvasEl.getBoundingClientRect();
    const x = ((e.clientX - rect.left) / rect.width) * canvasEl.width;
    const y = ((e.clientY - rect.top) / rect.height) * canvasEl.height;

    cursor.value = { x, y };
  };

  const onMouseLeave = () => {
    cursor.value = { x: 9999, y: 9999 };
  };

  onMounted(() => {
    nextTick(async () => {
      const cleanup = await initCanvas();
      if (canvas.value) {
        canvas.value.addEventListener("mousemove", onMouseMove);
        canvas.value.addEventListener("mouseleave", onMouseLeave);
        animate();
      }
    });
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
</style>
