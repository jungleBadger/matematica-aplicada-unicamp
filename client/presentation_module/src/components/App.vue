<!-- App.vue -->
<template>
  <div class="reveal" ref="mainSlides">
    <div class="slides">
      <SectionQRCode />
      <SectionWhoAmI />
      <SectionTwoWorlds />
      <SectionLabToMarket />
      <SectionVectorSearch />
      <SectionRAGDiagram />
      <SectionJuliaSnippet />
      <SectionPythonSnippet />
      <SectionBridgeChecklist />
      <SectionArchitectureOverview />
      <SectionImpactMetrics />
      <SectionIBMInternships />
      <SectionContact />
      <SectionThankYou />
    </div>
  </div>

  <!-- Status Badge -->
  <div class="status-badge" aria-live="polite">
    <div class="row">
      <span class="tag">{{ IS_MASTER ? 'MASTER' : 'AUDIENCE' }}</span>
      <span class="sep">•</span>
      <span class="room" title="Room">{{ ROOM }}</span>
    </div>
    <div class="row">
      <span class="dot" :class="isSocketConnected ? 'ok' : 'err'"></span>
      <span>Socket</span>
      <span class="sep">/</span>
      <span class="dot" :class="isGamepadConnected ? 'ok' : 'warn'"></span>
      <span>Gamepad</span>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";
import createDeck from "../configs/reveal";
import { store } from "../store/store.js";

/* Slides */
import SectionQRCode from "./SectionQRCode.vue";
import SectionWhoAmI from "./SectionWhoAmI.vue";
import SectionTwoWorlds from "./SectionTwoWorlds.vue";
import SectionLabToMarket from "./SectionLabToMarket.vue";
import SectionVectorSearch from "./SectionVectorSearch.vue";
import SectionRAGDiagram from "./SectionRAGDiagram.vue";
import SectionJuliaSnippet from "./SectionJuliaSnippet.vue";
import SectionPythonSnippet from "./SectionPythonSnippet.vue";
import SectionBridgeChecklist from "./SectionBridgeChecklist.vue";
import SectionArchitectureOverview from "./SectionArchitectureOverview.vue";
import SectionImpactMetrics from "./SectionImpactMetrics.vue";
import SectionIBMInternships from "./SectionIBMInternships.vue";
import SectionContact from "./SectionContact.vue";
import SectionThankYou from "./SectionThankYou.vue";

defineOptions({ name: "JPFApp" });

/* Deck ref */
const mainSlides = ref(null);
let deck;

/* ───── Query + loader ───── */
function qp(name, def = "") {
  const u = new URL(window.location.href);
  return u.searchParams.get(name) ?? def;
}
function loadScript(src) {
  return new Promise((resolve, reject) => {
    const s = document.createElement("script");
    s.src = src; s.async = true; s.onload = resolve; s.onerror = reject;
    document.head.appendChild(s);
  });
}

/* ───── Socket config (custom handler approach) ───── */
const SOCKET_ORIGIN = "https://d672f7fbb3ce.ngrok-free.app"; // ← replace
const ROOM = qp("room", "my-talk");
const IS_MASTER = qp("master", "") === "1";
const MASTER_KEY = qp("key", "");
let socket = null;

/* Badge reactive state */
const isSocketConnected = ref(false);
const isGamepadConnected = ref(false);

/* ───── Reveal handlers ───── */
function handleSlideChange(ev) {
  store.pageX = ev?.indexh ?? 0;
  store.pageY = ev?.indexv ?? 0;

  if (IS_MASTER && socket) {
    socket.emit("slidechanged", {
      key: MASTER_KEY, room: ROOM,
      h: ev?.indexh ?? 0, v: ev?.indexv ?? 0, f: ev?.indexf ?? 0
    });
  }
}
function handleFragmentShown(ev) {
  const el = ev?.fragment;
  if (el) { el.classList.remove("jpf-hidden"); el.classList.add("jpf-active"); }
  store.activeFragment = el ?? null;
  if (IS_MASTER && socket) socket.emit("fragmentshown", { key: MASTER_KEY, room: ROOM });
}
function handleFragmentHidden(ev) {
  const el = ev?.fragment;
  if (el) { el.classList.remove("jpf-active"); el.classList.add("jpf-hidden"); }
  store.activeFragment = null;
  if (IS_MASTER && socket) socket.emit("fragmenthidden", { key: MASTER_KEY, room: ROOM });
}

/* ───── Gamepad support ───── */
let gamepadRAF = null;
let lastButtons = [];
let lastAxes = [0, 0];
const DEADZONE = 0.35;
const REPEAT_COOLDOWN = 180;
let lastStickMoveAt = 0;

function pressedOnce(idx, gp) {
  const isPressed = !!(gp?.buttons?.[idx]?.pressed);
  const wasPressed = !!(lastButtons[idx]);
  lastButtons[idx] = isPressed;
  return isPressed && !wasPressed;
}
function axisValue(val) { return Math.abs(val) > DEADZONE ? Math.sign(val) : 0; }

function handleGamepad(gp) {
  if (!gp || !deck) return;
  // Set badge state (connected when any pad present)
  isGamepadConnected.value = true;

  // Face buttons: 0=Cross,1=Circle,2=Square,3=Triangle
  if (pressedOnce(3, gp)) deck.up();
  if (pressedOnce(1, gp)) deck.right();
  if (pressedOnce(0, gp)) deck.down();
  if (pressedOnce(2, gp)) deck.left();

  // D-pad: 12=Up, 13=Down, 14=Left, 15=Right
  if (pressedOnce(12, gp)) deck.up();
  if (pressedOnce(13, gp)) deck.down();
  if (pressedOnce(14, gp)) deck.left();
  if (pressedOnce(15, gp)) deck.right();

  // Left stick (axes[0]=x, axes[1]=y) with cooldown
  const axX = axisValue(gp.axes?.[0] ?? 0);
  const axY = axisValue(gp.axes?.[1] ?? 0);
  const now = performance.now();
  const changed = axX !== lastAxes[0] || axY !== lastAxes[1];
  if (changed && (now - lastStickMoveAt) > REPEAT_COOLDOWN) {
    if (axY < 0) deck.up();
    if (axY > 0) deck.down();
    if (axX < 0) deck.left();
    if (axX > 0) deck.right();
    lastStickMoveAt = now;
  }
  lastAxes[0] = axX; lastAxes[1] = axY;
}

function gamepadLoop() {
  const pads = navigator.getGamepads ? navigator.getGamepads() : [];
  const gp = pads && Array.from(pads).find(Boolean);
  if (gp) handleGamepad(gp);
  else isGamepadConnected.value = false; // none detected in this frame
  gamepadRAF = window.requestAnimationFrame(gamepadLoop);
}
function startGamepad() {
  lastButtons = new Array(32).fill(false);
  lastAxes = [0, 0];
  if (gamepadRAF == null) gamepadRAF = window.requestAnimationFrame(gamepadLoop);
}
function stopGamepad() {
  if (gamepadRAF != null) { window.cancelAnimationFrame(gamepadRAF); gamepadRAF = null; }
  isGamepadConnected.value = false;
}

/* ───── lifecycle ───── */
onMounted(async () => {
  // Reveal
  deck = createDeck(mainSlides.value);
  deck.on("slidechanged", handleSlideChange);
  deck.on("fragmentshown", handleFragmentShown);
  deck.on("fragmenthidden", handleFragmentHidden);

  // Theme toggle
  const THEMES = ["projector-theme", "light-theme", ""];
  let themeIndex = 0;
  const onKey = (e) => {
    if (e.key.toLowerCase() !== "t") return;
    document.documentElement.classList.remove("projector-theme", "light-theme");
    themeIndex = (themeIndex + 1) % THEMES.length;
    if (THEMES[themeIndex]) document.documentElement.classList.add(THEMES[themeIndex]);
  };
  window.addEventListener("keydown", onKey);

  await deck.initialize();

  // Socket.io client from your server (via ngrok)
  await loadScript(`${SOCKET_ORIGIN}/socket.io/socket.io.js`);
  // @ts-ignore provided by loaded script
  socket = window.io(SOCKET_ORIGIN, { path: "/socket.io", query: { room: ROOM } });
  socket.on("connect", () => { isSocketConnected.value = true; });
  socket.on("disconnect", () => { isSocketConnected.value = false; });

  if (!IS_MASTER) {
    socket.on("slidechanged", ({ h = 0, v = 0, f = 0 } = {}) => deck.slide(h, v, f));
    socket.on("fragmentshown", () => deck.next());
    socket.on("fragmenthidden", () => deck.prev());
  }

  // Gamepad listeners
  const onGamepadConnected = () => startGamepad();
  const onGamepadDisconnected = () => stopGamepad();
  window.addEventListener("gamepadconnected", onGamepadConnected);
  window.addEventListener("gamepaddisconnected", onGamepadDisconnected);
  if ([...navigator.getGamepads?.() || []].some(Boolean)) startGamepad();

  // Cleanup
  deck.on("destroy", () => {
    window.removeEventListener("keydown", onKey);
    window.removeEventListener("gamepadconnected", onGamepadConnected);
    window.removeEventListener("gamepaddisconnected", onGamepadDisconnected);
    try { socket && socket.disconnect(); } catch {}
    stopGamepad();
  });
});

onBeforeUnmount(() => {
  stopGamepad();
  try { socket && socket.disconnect(); } catch {}
  if (!deck) return;
  deck.off("slidechanged", handleSlideChange);
  deck.off("fragmentshown", handleFragmentShown);
  deck.off("fragmenthidden", handleFragmentHidden);
  deck.destroy();
});
</script>

<style scoped lang="scss">
.status-badge {
  position: fixed;
  top: 12px;
  right: 12px;
  z-index: 9999;
  background: var(--panel, rgba(17, 24, 39, .92));
  color: var(--ink, #fff);
  border: var(--panel-border, 1px solid rgba(255,255,255,.12));
  border-radius: var(--radius, 12px);
  box-shadow: var(--shadow, 0 8px 18px rgba(0,0,0,.3));
  padding: .6rem .8rem;
  font-size: clamp(12px, 1.6vw, 14px);
  line-height: 1.2;
  backdrop-filter: blur(4px);
}
.status-badge .row { display: flex; align-items: center; gap: .45rem; white-space: nowrap; }
.status-badge .row + .row { margin-top: .25rem; opacity: .9; }
.status-badge .tag {
  font-weight: 800;
  letter-spacing: .02em;
}
.status-badge .room {
  max-width: 22ch;
  overflow: hidden;
  text-overflow: ellipsis;
}
.status-badge .sep { opacity: .6; }
.status-badge .dot {
  width: .6rem; height: .6rem; border-radius: 999px; display: inline-block;
  box-shadow: 0 0 0 2px rgba(0,0,0,.15) inset;
}
.status-badge .dot.ok  { background: #10b981; } /* green */
.status-badge .dot.err { background: #ef4444; } /* red */
.status-badge .dot.warn{ background: #9ca3af; } /* gray */
</style>
