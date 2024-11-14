<!-- components/PomegranateParticles.vue -->
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
      this.stabilityFactor = 0.000002; // Casi sin movimiento aleatorio

      this.minVelocityThreshold = 0.005;

      this.baseHue = random.range(0, 5); // Rojo más brillante
      this.baseSaturation = random.range(90, 100);
      this.baseLightness = random.range(55, 65);
      this.timeOffset = random.range(0, Math.PI * 2);
    }

    update() {
      let dx, dy, dd, distDelta;

      // Fuerza que atrae hacia la posición inicial
      dx = this.ix - this.x;
      dy = this.iy - this.y;

      this.ax = dx * this.pullFactor;
      this.ay = dy * this.pullFactor;

      // Fuerza que repele del cursor
      dx = this.x - cursor.value.x;
      dy = this.y - cursor.value.y;
      dd = Math.sqrt(dx * dx + dy * dy);

      distDelta = this.minDist - dd;

      if (dd < this.minDist) {
        this.ax += (dx / dd) * distDelta * this.pushFactor;
        this.ay += (dy / dd) * distDelta * this.pushFactor;
      }

      // Movimiento aleatorio suave
      const time = performance.now() * 0.001;
      this.ax += Math.sin(time + this.timeOffset) * this.stabilityFactor;
      this.ay += Math.cos(time + this.timeOffset) * this.stabilityFactor;

      // Actualiza velocidad y aplica fricción
      this.vx += this.ax;
      this.vy += this.ay;

      this.vx *= this.dampFactor;
      this.vy *= this.dampFactor;

      // Detiene el movimiento si es muy lento
      if (
        Math.abs(this.vx) < this.minVelocityThreshold &&
        Math.abs(this.vy) < this.minVelocityThreshold
      ) {
        this.vx = 0;
        this.vy = 0;
      }

      // Actualiza posición
      this.x += this.vx;
      this.y += this.vy;

      // Obtener dimensiones del canvas
      const canvas = document.querySelector("canvas");
      const bounds = {
        left: this.radius,
        right: canvas.width - this.radius,
        top: this.radius,
        bottom: canvas.height - this.radius,
      };

      // Rebote en los bordes horizontales
      if (this.x < bounds.left) {
        this.x = bounds.left;
        this.vx *= -0.7; // Rebote con pérdida de energía
      } else if (this.x > bounds.right) {
        this.x = bounds.right;
        this.vx *= -0.7;
      }

      // Rebote en los bordes verticales
      if (this.y < bounds.top) {
        this.y = bounds.top;
        this.vy *= -0.7;
      } else if (this.y > bounds.bottom) {
        this.y = bounds.bottom;
        this.vy *= -0.7;
      }

      const maxOffset = 50;
      if (Math.abs(this.y - this.iy) > maxOffset) {
        const direction = this.y > this.iy ? 1 : -1;
        this.y = this.iy + maxOffset * direction;
        this.vy *= -0.5;
      }

      this.rotation += this.vx * 0.05;
    }

    draw(context) {
      context.save();
      context.translate(this.x, this.y);
      context.rotate(this.rotation);

      // Color base más profundo con variaciones sutiles
      const hue = 350 + random.range(-5, 5);
      const mainColor = `hsla(${hue}, 95%, 45%, 0.95)`;
      const highlightColor = `hsla(${hue}, 90%, 65%, 0.95)`;
      const shadowColor = `hsla(${hue}, 95%, 35%, 0.95)`;

      context.beginPath();

      // Forma más orgánica para los arilos
      const numPoints = 6; // Menos puntos para forma más definida
      const angleOffset = this.timeOffset;

      // Primera capa: forma base
      for (let i = 0; i < numPoints; i++) {
        const angle = (i / numPoints) * Math.PI * 2 + angleOffset;
        const radiusVariation = 1 + Math.sin(angle * 3 + this.timeOffset) * 0.2;
        const currentRadius = this.radius * radiusVariation;

        const x = Math.cos(angle) * currentRadius;
        const y = Math.sin(angle) * currentRadius;

        if (i === 0) {
          context.moveTo(x, y);
        } else {
          // Usando curvas bezier para forma más orgánica
          const prevAngle = ((i - 1) / numPoints) * Math.PI * 2 + angleOffset;
          const cpRadius = this.radius * 1.3;
          const cp1x = Math.cos(prevAngle + 0.5) * cpRadius;
          const cp1y = Math.sin(prevAngle + 0.5) * cpRadius;
          const cp2x = Math.cos(angle - 0.5) * cpRadius;
          const cp2y = Math.sin(angle - 0.5) * cpRadius;
          context.bezierCurveTo(cp1x, cp1y, cp2x, cp2y, x, y);
        }
      }

      context.closePath();

      // Sombra sutil
      context.shadowColor = shadowColor;
      context.shadowBlur = 5;
      context.shadowOffsetX = 2;
      context.shadowOffsetY = 2;

      // Color base
      context.fillStyle = mainColor;
      context.fill();

      // Brillo principal
      const gradient = context.createRadialGradient(
        -this.radius * 0.2,
        -this.radius * 0.2,
        0,
        0,
        0,
        this.radius * 1.2
      );
      gradient.addColorStop(0, highlightColor);
      gradient.addColorStop(0.5, "transparent");
      gradient.addColorStop(1, shadowColor);

      context.shadowColor = "transparent";
      context.fillStyle = gradient;
      context.fill();

      // Brillo adicional para efecto cristalino
      context.beginPath();
      context.arc(
        -this.radius * 0.2,
        -this.radius * 0.2,
        this.radius * 0.3,
        0,
        Math.PI * 2
      );
      context.fillStyle = `hsla(${hue}, 90%, 75%, 0.2)`;
      context.fill();

      context.restore();
    }
  }

  const initCanvas = () => {
    if (!canvas.value || !canvasContainer.value) return;

    const canvasEl = canvas.value;
    ctx = canvasEl.getContext("2d");

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
    nextTick(() => {
      const cleanup = initCanvas();
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
