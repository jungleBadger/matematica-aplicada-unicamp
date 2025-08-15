<template>
  <section class="slide-container qr-slide">
    <h2 class="slide-title">Acompanhe em tempo real!</h2>

    <div class="qr-wrap">
      <!-- The SVG is injected here -->
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
  try {
    const svg = await qrToString(QR_URL, {
      type: "svg",
      margin: 1,
      width: 420, // we’ll override via CSS; this just sets a base viewBox
      color: { dark: "#000000", light: "#FFFFFF" },
    });

    if (!qrBox.value) return;
    qrBox.value.innerHTML = svg;

    const svgEl = qrBox.value.querySelector("svg");
    if (svgEl) {
      // Make sizing 100% controlled by CSS
      svgEl.removeAttribute("width");
      svgEl.removeAttribute("height");
      svgEl.setAttribute("preserveAspectRatio", "xMidYMid meet");
      svgEl.classList.add("qr-svg");
      svgEl.setAttribute("role", "img");
      svgEl.setAttribute(
          "aria-label",
          "QR code para junglebadger.github.io/matematica-aplicada-unicamp?room=my-talk"
      );
    }
  } catch (e) {
    console.warn("QR generation failed:", e);
  }
});
</script>

<style scoped>
.qr-slide .slide-title {
  /* Give the QR a bit more room on small screens */
  margin-bottom: 1.25rem;
}

/* Layout container for QR + caption */
.qr-wrap {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1.25rem;
}

/* Box that constrains the SVG size */
.qr-box {
  display: grid;
  place-items: center;
  /* Responsive width; prevents projector overflow */
  width: min(56vmin, 320px);
  max-width: 90vw;
  /* Also cap vertical growth so it never overflows horizontally */
  max-height: 60vh;
  aspect-ratio: 1 / 1;
  background: #fff;
  border-radius: var(--radius);
  border: var(--panel-border);
  box-shadow: var(--shadow);
  overflow: hidden; /* hard-stop overflow */
}

/* The injected SVG obeys the box */
.qr-svg {
  width: 100%;
  height: 100%;
  display: block;
}

/* Fallback text while loading or on error */
.qr-fallback {
  color: #111;
  font-weight: 700;
}

/* Caption */
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
}
</style>
