<template>
  <div class="reveal" ref="mainSlides">
    <div class="slides">
      <SectionWhoAmI />
      <SectionTwoWorlds />
      <SectionLabToMarket />
      <SectionVectorSearch />
      <sectionRAGDiagram />
      <SectionJuliaSnippet />
      <SectionPythonSnippet />
      <SectionBridgeChecklist />
      <SectionArchitectureOverview />
      <SectionImpactMetrics />
      <SectionContact />
      <SectionThankYou />

    </div>
  </div>
</template>

<script setup>

import SectionImpactMetrics from "./SectionImpactMetrics.vue";

defineOptions({ name: "JPFApp" });

import { ref, onMounted, onBeforeUnmount } from "vue";
import createDeck from "../configs/reveal";
import { store } from "../store/store.js";

import SectionWhoAmI from "./SectionWhoAmI.vue";
import SectionThankYou from "./SectionThankYou.vue";
import SectionContact from "./SectionContact.vue";
import SectionVectorSearch from "./SectionVectorSearch.vue";
import SectionJuliaSnippet from "./SectionJuliaSnippet.vue";
import SectionPythonSnippet from "./SectionPythonSnippet.vue";
import SectionEmbeddingBestPractices from "./SectionEmbeddingBestPractices.vue";
import SectionVectorIndex from "./SectionVectorIndex.vue";
import SectionTwoWorlds from "./SectionTwoWorlds.vue";
import SectionLabToMarket from "./SectionLabToMarket.vue";
import SectionArchitectureOverview from "./SectionArchitectureOverview.vue";
import SectionBridgeChecklist from "./SectionBridgeChecklist.vue";
import SectionRAGDiagram from "./SectionRAGDiagram.vue";

const mainSlides = ref(null);
let deck;

/* handlers */
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

/* lifecycle */
onMounted(async () => {
  deck = createDeck(mainSlides.value);
  deck.on("slidechanged", handleSlideChange);
  deck.on("fragmentshown", handleFragmentShown);
  deck.on("fragmenthidden", handleFragmentHidden);
  const THEMES = ['projector-theme', 'light-theme', '']; // '' = default
  let i = 0;
  window.addEventListener('keydown', e => {
    if (e.key.toLowerCase() !== 't') return;
    document.documentElement.classList.remove('projector-theme', 'light-theme');
    i = (i + 1) % THEMES.length;
    if (THEMES[i]) document.documentElement.classList.add(THEMES[i]);
  });
  await deck.initialize();
  // if you add slides dynamically later: deck.sync();
});

onBeforeUnmount(() => {
  if (!deck) return;
  deck.off("slidechanged", handleSlideChange);
  deck.off("fragmentshown", handleFragmentShown);
  deck.off("fragmenthidden", handleFragmentHidden);
  deck.destroy();
});
</script>

<style scoped lang="scss">
/* keep empty or add tiny local tweaks; Reveal’s CSS handles layout */
</style>
