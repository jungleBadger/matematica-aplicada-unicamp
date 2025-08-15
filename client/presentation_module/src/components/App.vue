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

/* ============== Reveal event handlers ============== */
function handleSlideChange(ev) {
  store.pageX = ev?.indexh ?? 0;
  store.pageY = ev?.indexv ?? 0;
}

function handleFragmentShown(ev) {
  const el = ev?.fragment;
  if (el) {
    el.classList.remove("jpf-hidden");
    el.classList.add("jpf-active");
  }
  store.activeFragment = el ?? null;
}

function handleFragmentHidden(ev) {
  const el = ev?.fragment;
  if (el) {
    el.classList.remove("jpf-active");
    el.classList.add("jpf-hidden");
  }
  store.activeFragment = null;
}

/* ============== Gamepad support ============== */
/** rAF handle */
let gamepadRAF = null;
/** previous buttons pressed state to detect rising edges */
let lastButtons = [];
/** last axes directions (quantized) to throttle repeats */
let lastAxes = [0, 0];
/** stick params */
const DEADZONE = 0.35;            // analog deadzone
const REPEAT_COOLDOWN = 180;      // ms between stick moves
let lastStickMoveAt = 0;

function pressedOnce(idx, gp) {
  const isPressed = !!(gp?.buttons?.[idx]?.pressed);
  const wasPressed = !!(lastButtons[idx]);
  lastButtons[idx] = isPressed;
  return isPressed && !wasPressed;
}

/** quantize axis using deadzone -> -1,0,1 */
function axisValue(val) {
  return Math.abs(val) > DEADZONE ? Math.sign(val) : 0;
}

function handleGamepad(gp) {
  if (!gp || !deck) return;

  // Face buttons (PS layout): 0=Cross,1=Circle,2=Square,3=Triangle
  if (pressedOnce(3, gp)) deck.up();    // Triangle
  if (pressedOnce(1, gp)) deck.right(); // Circle
  if (pressedOnce(0, gp)) deck.down();  // Cross
  if (pressedOnce(2, gp)) deck.left();  // Square

  // D-pad: 12=Up, 13=Down, 14=Left, 15=Right
  if (pressedOnce(12, gp)) deck.up();
  if (pressedOnce(13, gp)) deck.down();
  if (pressedOnce(14, gp)) deck.left();
  if (pressedOnce(15, gp)) deck.right();

  // Left stick: axes[0]=X, axes[1]=Y (with cooldown)
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
  lastAxes[0] = axX;
  lastAxes[1] = axY;
}

function gamepadLoop() {
  const pads = navigator.getGamepads ? navigator.getGamepads() : [];
  // pick first connected
  const gp = pads && Array.from(pads).find(Boolean);
  if (gp) handleGamepad(gp);
  gamepadRAF = window.requestAnimationFrame(gamepadLoop);
}

function startGamepad() {
  lastButtons = new Array(32).fill(false);
  lastAxes = [0, 0];
  if (gamepadRAF == null) gamepadRAF = window.requestAnimationFrame(gamepadLoop);
}

function stopGamepad() {
  if (gamepadRAF != null) {
    window.cancelAnimationFrame(gamepadRAF);
    gamepadRAF = null;
  }
}

/* ============== lifecycle ============== */
onMounted(async () => {
  deck = createDeck(mainSlides.value);
  deck.on("slidechanged", handleSlideChange);
  deck.on("fragmentshown", handleFragmentShown);
  deck.on("fragmenthidden", handleFragmentHidden);

  // Theme toggle with 'T'
  const THEMES = ["projector-theme", "light-theme", ""]; // '' = default
  let themeIndex = 0;
  const onKey = (e) => {
    if (e.key.toLowerCase() !== "t") return;
    document.documentElement.classList.remove("projector-theme", "light-theme");
    themeIndex = (themeIndex + 1) % THEMES.length;
    if (THEMES[themeIndex]) document.documentElement.classList.add(THEMES[themeIndex]);
  };
  window.addEventListener("keydown", onKey);

  await deck.initialize();

  // Gamepad events
  const onGamepadConnected = () => {
    // some browsers require user gesture; ensure loop is running
    startGamepad();
  };
  const onGamepadDisconnected = () => {
    stopGamepad();
  };

  window.addEventListener("gamepadconnected", onGamepadConnected);
  window.addEventListener("gamepaddisconnected", onGamepadDisconnected);

  // If already connected (refresh), kick the loop
  if ([...navigator.getGamepads?.() || []].some(Boolean)) startGamepad();

  // Cleanup when Reveal destroys
  deck.on("destroy", () => {
    window.removeEventListener("keydown", onKey);
    window.removeEventListener("gamepadconnected", onGamepadConnected);
    window.removeEventListener("gamepaddisconnected", onGamepadDisconnected);
    stopGamepad();
  });
});

onBeforeUnmount(() => {
  stopGamepad();
  if (!deck) return;
  deck.off("slidechanged", handleSlideChange);
  deck.off("fragmentshown", handleFragmentShown);
  deck.off("fragmenthidden", handleFragmentHidden);
  deck.destroy();
});
</script>

<style scoped lang="scss">
/* Keep empty or add tiny local tweaks; Reveal handles layout */
</style>
