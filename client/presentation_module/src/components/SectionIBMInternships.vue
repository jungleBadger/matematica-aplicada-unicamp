<!-- SectionIBMInternships.vue -->
<template>
  <section class="slide-container">
    <h2 class="slide-title">Vagas de Estágio — IBM</h2>

    <div class="cards-container">
      <!-- Vagas em destaque -->
      <div class="card fragment" data-fragment-index="1">
        <div class="icon" aria-hidden="true">💼</div>
        <h3>Vagas em destaque</h3>
        <ul>
          <li>
            <a href="https://ibmglobal.avature.net/pt_BR/careers/JobDetail?jobId=49337&source=WEB_Search_LA"
               target="_blank" rel="noopener noreferrer">
              Estágio em Suporte de Aplicativos de Negócios
            </a>
          </li>
          <li>
            <a href="https://ibmglobal.avature.net/pt_BR/careers/JobDetail?jobId=52832&source=WEB_Search_LA"
               target="_blank" rel="noopener noreferrer">
              Programa de Estágio — Desenvolvimento Front End
            </a>
          </li>
          <li>
            <a href="https://ibmglobal.avature.net/pt_BR/careers/JobDetail?jobId=50948&source=WEB_Search_LA"
               target="_blank" rel="noopener noreferrer">
              Programa de Estágio — Exclusivo para Pessoas com Deficiência
            </a>
          </li>
        </ul>
      </div>

      <!-- Onde buscar mais vagas -->
      <div class="card fragment" data-fragment-index="2">
        <div class="icon" aria-hidden="true">🔎</div>
        <h3>Onde buscar mais vagas</h3>
        <ul>
          <li>
            <a href="https://www.ibm.com/careers" target="_blank" rel="noopener noreferrer">
              IBM Carreiras — Página oficial
            </a>
          </li>
        </ul>
        <!-- QR Code -->

        <div style="margin-top: 1rem; text-align: center;">
            Escaneie para acessar
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
defineOptions({ name: "SectionIBMInternships" });
</script>
