<template>
  <section>
    <NuxtImg src="/images/hero.png" />
    <div class="particle-wrapper">
      <PomegranateParticles />
    </div>
    <NuxtImg src="/images/lower.png" />
  </section>
</template>

<style scoped>
  section {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  h1,
  p {
    text-align: center;
  }

  img {
    width: 100%;
    max-width: 320px;
    margin: 0 auto;
  }

  .particle-wrapper {
    width: 100%;
    max-width: 320px;
    padding: 1.125rem 1.25rem;
    margin: 0 auto;
  }
</style>
