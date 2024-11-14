<template>
  <section>
    <NuxtImg src="/images/hero.png" />
    <div class="particle-wrapper">
      <PomegranateParticles />
    </div>
    <div id="becariosActualesContent" class="becariosContainer activeContent">
      <div class="w-100 titleContent columnAlignCenter">
        <h2 class="w-100 titleUnderline">
          <span>Residencia Oncológica Clínica</span>
          <span class="greenUnderlineTitle"></span>
        </h2>
        <div class="containerAnios columnAlignCenter">
          <h3>Primer Año</h3>
          <div class="containerBecariosCards columnAlignCenter">
            <div class="becarioCard columnAlignCenter">
              <div class="becarioInfo columnAlignCenter">
                <p class="nombre">Dr. Gisela Vargas</p>
                <p class="anios">2022-2026</p>
                <p class="lugar">Instituto Alexander Fleming, CABA</p>
              </div>
            </div>
            <div class="becarioCard columnAlignCenter">
              <div class="becarioInfo columnAlignCenter">
                <p class="nombre">Dra. Dulce Alzugaray</p>
                <p class="anios">2022-2026</p>
              </div>
            </div>
          </div>
        </div>
        <div class="containerAnios columnAlignCenter">
          <h3>Segundo Año</h3>
          <div class="containerBecariosCards columnAlignCenter">
            <div class="becarioCard columnAlignCenter">
              <div class="becarioInfo columnAlignCenter">
                <p class="nombre">Dr. Carlos Ramirez</p>
                <p class="anios">2022-2026</p>
              </div>
            </div>
            <div class="becarioCard columnAlignCenter">
              <div class="becarioInfo columnAlignCenter">
                <p class="nombre">Dra. Romina Ferrari</p>
                <p class="anios">2022-2026</p>
                <p class="lugar">Instituto Alexander Fleming, CABA</p>
              </div>
            </div>
            <div class="becarioCard columnAlignCenter">
              <div class="becarioInfo columnAlignCenter">
                <p class="nombre">Dr. Carlos Palmés</p>
                <p class="anios">2022-2026</p>
                <p class="lugar">Hospital Alta Complejidad</p>
              </div>
            </div>
          </div>
        </div>
        <div class="containerAnios columnAlignCenter">
          <h3>Tercer Año</h3>
          <div class="containerBecariosCards columnAlignCenter">
            <div class="becarioCard columnAlignCenter">
              <div class="becarioInfo columnAlignCenter">
                <p class="nombre">Dra. Dana Narvaez</p>
                <p class="anios">2022-2026</p>
                <p class="lugar">Hospital Alta Complejidad</p>
              </div>
            </div>
            <div class="becarioCard columnAlignCenter">
              <div class="becarioInfo columnAlignCenter">
                <p class="nombre">Dr. Marcos Daniel Bortz</p>
                <p class="anios">2022-2026</p>
              </div>
            </div>
            <div class="becarioCard columnAlignCenter">
              <div class="becarioInfo columnAlignCenter">
                <p class="nombre">Dra. Greta Catani</p>
                <p class="anios">2022-2026</p>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="w-100 titleContent columnAlignCenter">
        <h2 class="w-100 titleUnderline">
          <span>Residencia en Mastología</span>
          <span class="greenUnderlineTitle"></span>
        </h2>
        <div class="containerBecariosCards columnAlignCenter">
          <div class="becarioCard columnAlignCenter">
            <div class="becarioInfo columnAlignCenter">
              <p class="nombre">Dra. María Belén Ancao</p>
              <p class="anios">2023-2025</p>
            </div>
          </div>
        </div>
      </div>
      <div class="w-100 titleContent columnAlignCenter">
        <h2 class="w-100 titleUnderline">
          <span>Fellowship en Onco-Hematología</span>
          <span class="greenUnderlineTitle"></span>
        </h2>
        <div class="containerBecariosCards columnAlignCenter">
          <div class="becarioCard columnAlignCenter">
            <div class="becarioInfo columnAlignCenter">
              <p class="nombre">Dra. Josefina Troccoli</p>
              <p class="anios">2023-2025</p>
            </div>
          </div>
          <div class="becarioCard columnAlignCenter">
            <div class="becarioInfo columnAlignCenter">
              <p class="nombre">Dra. Andrea Carolina Muñoz</p>
              <p class="anios">2023-2025</p>
            </div>
          </div>
        </div>
      </div>
      <div class="w-100 titleContent columnAlignCenter">
        <h2 class="w-100 titleUnderline">
          <span>Fellowship en Cirugía Torácica</span>
          <span class="greenUnderlineTitle"></span>
        </h2>
        <div class="containerBecariosCards columnAlignCenter">
          <div class="becarioCard columnAlignCenter">
            <div class="becarioInfo columnAlignCenter">
              <p class="nombre">Dr. Miguel Ángel Lima</p>
              <p class="anios">2023-2025</p>
            </div>
          </div>
        </div>
      </div>
      <div class="w-100 titleContent columnAlignCenter">
        <h2 class="w-100 titleUnderline">
          <span>Fellowship en Urología Oncológica</span>
          <span class="greenUnderlineTitle"></span>
        </h2>
        <div class="containerBecariosCards columnAlignCenter">
          <div class="becarioCard columnAlignCenter">
            <div class="becarioInfo columnAlignCenter">
              <p class="nombre">Dra. Tatiana Ailen Letowski</p>
              <p class="anios">2023-2024</p>
            </div>
          </div>
          <div class="becarioCard columnAlignCenter">
            <div class="becarioInfo columnAlignCenter">
              <p class="nombre">Dr. Gastón Fernández Candia</p>
              <p class="anios">2023-2024</p>
            </div>
          </div>
        </div>
      </div>
      <div class="w-100 titleContent columnAlignCenter">
        <h2 class="w-100 titleUnderline">
          <span>Fellowship en Ortopedia Oncológica</span>
          <span class="greenUnderlineTitle"></span>
        </h2>
        <div class="containerBecariosCards columnAlignCenter">
          <div class="becarioCard columnAlignCenter">
            <div class="becarioInfo columnAlignCenter">
              <p class="nombre">Dr. Miguel Alfredo Beleño Cruz</p>
              <p class="anios">2022-2024</p>
            </div>
          </div>
        </div>
      </div>
    </div>
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
  }

  .particle-wrapper {
    width: 100%;
    padding: 1.125rem 1.25rem;
  }
</style>
