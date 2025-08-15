<template>
  <section class="slide-container qr-slide">
    <h2 class="slide-title">Acompanhe em tempo real!</h2>

    <div class="qr-wrap">
      <div ref="qrBox" class="qr-box">
        <div class="qr-fallback">QR loading…</div>
      </div>

      <div class="qr-caption">
        <p>Leia o QR Code ou acesse:</p>
        <p>
          <a
              href="https://junglebadger.github.io/matematica-aplicada-unicamp?room=my-talk"
              target="_blank"
              rel="noopener noreferrer"
          >
            junglebadger.github.io/matematica-aplicada-unicamp?room=my-talk
          </a>
        </p>
      </div>
    </div>
  </section>
</template>

<script setup>
import { onMounted, ref } from "vue";
import { toString as qrToString } from "qrcode";

defineOptions({ name: "SectionQRCode" });

const qrBox = ref(null);
const QR_URL =
    "https://junglebadger.github.io/matematica-aplicada-unicamp?room=my-talk";

onMounted(async () => {
  const svg = await qrToString(QR_URL, {
    type: "svg",
    margin: 1,
    width: 420,
    color: { dark: "#000000", light: "#FFFFFF" },
  });

  if (!qrBox.value) return;
  qrBox.value.innerHTML = svg;

  const svgEl = qrBox.value.querySelector("svg");
  if (svgEl) {
    // Deixe o CSS controlar o tamanho
    svgEl.removeAttribute("width");
    svgEl.removeAttribute("height");
    svgEl.setAttribute("preserveAspectRatio", "xMidYMid meet");
    svgEl.setAttribute("role", "img");
    svgEl.setAttribute(
        "aria-label",
        "QR code para junglebadger.github.io/matematica-aplicada-unicamp?room=my-talk"
    );
  }
});
</script>

<style scoped>
.qr-slide .slide-title {
  margin-bottom: 1.25rem;
}

.qr-wrap {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.25rem;
}

.qr-box {
  display: grid;
  place-items: center;
  width: min(56vmin, 320px);
  max-width: 90vw;
  max-height: 60vh;
  aspect-ratio: 1 / 1;
  background: #fff;
  border-radius: var(--radius);
  border: var(--panel-border);
  box-shadow: var(--shadow);
  overflow: hidden; /* evita bleed */
}

/* ⬇️ Regras profundas: atingem o SVG INJETADO mesmo com <style scoped> */
:deep(.qr-box svg) {
  width: 100%;
  height: 100%;
  display: block;
}

/* opcional: cantos arredondados no próprio conteúdo */
:deep(.qr-box svg rect),
:deep(.qr-box svg path) {
  shape-rendering: crispEdges;
}

.qr-fallback {
  color: #111;
  font-weight: 700;
}

.qr-caption {
  text-align: center;
}

.qr-caption p {
  margin: 0.15rem 0;
  color: var(--ink);
  font-size: clamp(16px, 2.1vw, 22px);
}

.qr-caption a {
  font-weight: 700;
  font-size: clamp(18px, 2.3vw, 26px);
  word-break: break-all;
}
</style>
