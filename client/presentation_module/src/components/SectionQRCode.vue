<template>
  <section class="slide-container">
    <h2 class="slide-title">Acompanhe em tempo real!</h2>

    <div style="display:flex; flex-direction:column; align-items:center; gap:1.25rem;">
      <!-- QR area: we render a proper SVG here; fallback SVG stays if fetch fails -->
      <div ref="qrBox" style="display:flex; justify-content:center;">
        <!-- Fallback inline SVG (kept from your version) -->
        <svg
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 41 41"
            shape-rendering="crispEdges"
            role="img"
            aria-label="QR code"
            style="width:min(56vmin, 320px); height:auto; box-shadow: var(--shadow); border: var(--panel-border); border-radius: var(--radius); background:#fff;"
        >
          <path fill="#ffffff" d="M0 0h41v41H0z"/>
          <path stroke="#000000" d="M4 4.5h7m3 0h3m3 0h1m1 0h4m1 0h1m2 0h7M4 5.5h1m5 0h1m2 0h2m1 0h1m2 0h2m1 0h1m2 0h2m1 0h1m1 0h1m5 0h1M4 6.5h1m1 0h3m1 0h1m1 0h4m2 0h2m3 0h3m4 0h1m1 0h3m1 0h1M4 7.5h1m1 0h3m1 0h1m1 0h2m8 0h2m2 0h2m2 0h1m1 0h3m1 0h1M4 8.5h1m1 0h3m1 0h1m1 0h2m2 0h5m3 0h2m1 0h1m2 0h1m1 0h3m1 0h1M4 9.5h1m5 0h1m1 0h3m2 0h2m2 0h1m1 0h1m1 0h2m3 0h1m5 0h1M4 10.5h7m1 0h1m1 0h1m1 0h1m1 0h1m1 0h1m1 0h1m1 0h1m1 0h1m1 0h1m1 0h7M12 11.5h2m4 0h1m1 0h1m2 0h4m1 0h1M4 12.5h1m1 0h5m3 0h4m3 0h1m1 0h1m2 0h1m1 0h1m1 0h5M5 13.5h2m1 0h2m2 0h4m1 0h1m2 0h4m3 0h1m2 0h2m1 0h4M4 14.5h1m4 0h2m2 0h7m1 0h1m3 0h1m3 0h1m2 0h1m1 0h2M6 15.5h2m1 0h1m1 0h1m2 0h2m1 0h1m2 0h4m1 0h3m1 0h1m2 0h4M5 16.5h1m2 0h1m1 0h3m1 0h1m1 0h1m1 0h1m1 0h1m3 0h1m1 0h1m1 0h2m1 0h3m1 0h1M4 17.5h2m2 0h2m1 0h1m1 0h2m1 0h2m1 0h1m1 0h2m2 0h1m1 0h1m1 0h2m2 0h4M4 18.5h2m4 0h1m2 0h8m1 0h1m1 0h1m3 0h5m1 0h2M5 19.5h2m1 0h1m3 0h2m2 0h1m1 0h2m5 0h1m3 0h3m1 0h2M4 20.5h1m1 0h2m2 0h1m1 0h3m1 0h4m1 0h1m2 0h1m1 0h1m1 0h2m1 0h2m3 0h1M5 21.5h1m2 0h2m1 0h2m1 0h2m1 0h1m5 0h2m2 0h1m2 0h2m1 0h2m1 0h1M6 22.5h1m2 0h3m3 0h1m1 0h1m1 0h1m2 0h1m1 0h1m1 0h1m1 0h1m2 0h2m1 0h2M4 23.5h6m1 0h2m2 0h2m1 0h1m5 0h4m1 0h8M10 24.5h2m1 0h1m5 0h1m2 0h1m3 0h2m1 0h1m2 0h2M4 25.5h1m1 0h2m1 0h1m1 0h10m3 0h2m1 0h1m2 0h1m5 0h1M4 26.5h1m2 0h1m1 0h4m3 0h2m5 0h1m1 0h1m2 0h1m1 0h2m1 0h3M4 27.5h1m1 0h1m1 0h1m2 0h2m2 0h1m4 0h1m1 0h6m2 0h2m1 0h3M4 28.5h1m5 0h2m2 0h5m2 0h1m1 0h1m2 0h9M12 29.5h2m3 0h1m2 0h5m3 0h1m3 0h1m1 0h3M4 30.5h7m2 0h1m1 0h2m2 0h3m2 0h1m1 0h3m1 0h1m1 0h1m1 0h2M4 31.5h1m5 0h1m1 0h1m3 0h2m1 0h7m1 0h2m3 0h5M4 32.5h1m1 0h3m1 0h1m1 0h3m4 0h2m7 0h6m1 0h2M4 33.5h1m1 0h3m1 0h1m1 0h4m2 0h2m1 0h2m2 0h1m1 0h3m2 0h2m1 0h2M4 34.5h1m1 0h3m1 0h1m1 0h1m1 0h2m3 0h2m1 0h1m1 0h8m1 0h1M4 35.5h1m5 0h1m2 0h3m1 0h3m2 0h4m3 0h1m2 0h3M4 36.5h7m1 0h2m1 0h1m1 0h5m1 0h1m2 0h4m1 0h1m1 0h1m1 0h1"/>
        </svg>
      </div>

      <div style="text-align:center;">
        <p style="font-size:clamp(18px,2.2vw,24px); color:var(--ink); margin:0;">
          Leia o QR Code ou acesse:
        </p>
        <p style="margin:.25rem 0 0;">
          <a
              href="https://junglebadger.github.io/matematica-aplicada-unicamp?room=my-talk"
              target="_blank"
              rel="noopener noreferrer"
              style="font-weight:700; font-size:clamp(18px,2.4vw,26px);"
          >
            junglebadger.github.io/matematica-aplicada-unicamp
          </a>
        </p>
      </div>
    </div>
  </section>
</template>

<script setup>
import { onMounted, ref } from "vue";
defineOptions({ name: "SectionQRCode" });

const qrBox = ref(null);
const QR_URL =
    "https://junglebadger.github.io/matematica-aplicada-unicamp?room=my-talk";

// Replace fallback with a live SVG QR (keeps everything inline in the DOM)
onMounted(async () => {
  try {
    const resp = await fetch(
        `https://api.qrserver.com/v1/create-qr-code/?size=420x420&format=svg&data=${encodeURIComponent(
            QR_URL
        )}`
    );
    if (!resp.ok) throw new Error("QR fetch failed");
    const svgText = await resp.text();
    if (svgText && qrBox.value) {
      qrBox.value.innerHTML = svgText;
      const svg = qrBox.value.querySelector("svg");
      if (svg) {
        svg.setAttribute(
            "style",
            "width:min(56vmin, 320px); height:auto; box-shadow: var(--shadow); border: var(--panel-border); border-radius: var(--radius); background:#fff;"
        );
        svg.setAttribute("role", "img");
        svg.setAttribute("aria-label", "QR code para acesso rápido");
      }
    }
  } catch {
    // fallback SVG remains
  }
});
</script>
