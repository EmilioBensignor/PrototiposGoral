<template>
  <div>
    <section class="mobile">
      <div class="particle-wrapper">
        <PomegranateParticles />
      </div>
      <PomegranateViewer1 />
    </section>
    <!-- <section class="desktop">
      <NuxtImg src="/images/heroDesktop.png" alt="hero" />
      <div class="particle-wrapper">
        <PomegranateParticles />
      </div>
      <div class="flex">
        <PomegranateViewer />
        <NuxtImg class="side" src="/images/sideDesktop.png" alt="lower" />
      </div>
    </section> -->
  </div>
</template>

<style scoped>
  .desktop {
    display: none;
  }

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

  @media (width >= 1080px) {
    .mobile {
      display: none;
    }

    .desktop {
      display: block;
    }

    img, .particle-wrapper {
      max-width: 1080px;
    }

    .side {
      width: 70%;
    }
  }
</style>
