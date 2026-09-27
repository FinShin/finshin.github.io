<template>
  <Teleport to="body">
    <Transition name="fade-popup">
      <div
        v-if="isPopupOpen"
        class="popup-overlay"
        @click.self="isPopupOpen = false"
      >
        <button
          class="popup-close select-none"
          @click="isPopupOpen = false"
          aria-label="Close"
        >
          &times;
        </button>
        <img
          :src="popupImageSrc"
          class="popup-img select-none"
          loading="lazy"
          decoding="async"
        />
      </div>
    </Transition>
  </Teleport>

  <div
    ref="rootRef"
    class="depth-carousel"
    :class="[className, { 'is-dragging': isDragging }]"
    :style="{ '--dc-perspective': `${perspective}px` }"
    role="group"
    aria-roledescription="carousel"
    aria-label="Depth carousel"
    tabindex="0"
    @pointerdown="onPointerDown"
    @pointermove="onPointerMove"
    @pointerup="onPointerEnd"
    @pointercancel="onPointerEnd"
    @keydown="onKeyDown"
  >
    <div class="depth-carousel__stage">
      <div
        v-for="(item, i) in normalizedData"
        :key="i"
        class="depth-carousel__card"
        :ref="(el) => setCardRef(el, i)"
        :style="{
          width: `${displayCardWidth}px`,
          aspectRatio: `${displayCardWidth} / ${displayCardHeight}`,
          borderRadius: `${props.radius}px`,
        }"
        aria-roledescription="slide"
        :aria-label="`${i + 1} of ${count}`"
        :aria-hidden="active !== i"
        @click="onCardClick(i)"
      >
        <img
          class="depth-carousel__img"
          :src="item.image"
          :alt="item.alt || ''"
          draggable="false"
          decoding="async"
          loading="lazy"
        />
        <span
          class="depth-carousel__tint"
          :ref="(el) => setOverlayRef(el, i)"
          :style="{ background: tint }"
        ></span>
      </div>
    </div>

    <!-- Arrow Controls -->
    <template v-if="showControls && count > 1">
      <button
        type="button"
        class="depth-carousel__arrow depth-carousel__arrow--prev"
        aria-label="Previous slide"
        @click="navigateBy(-1)"
      >
        <svg viewBox="0 0 24 24" width="20" height="20" aria-hidden="true">
          <path
            d="M15 5l-7 7 7 7"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          />
        </svg>
      </button>
      <button
        type="button"
        class="depth-carousel__arrow depth-carousel__arrow--next"
        aria-label="Next slide"
        @click="navigateBy(1)"
      >
        <svg viewBox="0 0 24 24" width="20" height="20" aria-hidden="true">
          <path
            d="M9 5l7 7-7 7"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          />
        </svg>
      </button>
    </template>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount, watch } from "vue";

export type DepthCarouselItem = string | { image: string; alt?: string };
type TiltDirection = "left" | "right";

const isPopupOpen = ref(false);
const popupImageSrc = ref("");

const openPopup = (imgSrc: string) => {
  popupImageSrc.value = imgSrc;
  isPopupOpen.value = true;
};

export interface DepthCarouselProps {
  items?: DepthCarouselItem[];
  cardWidth?: number;
  cardHeight?: number;
  radius?: number;
  tint?: string;
  depth?: number;
  spread?: number;
  tilt?: number;
  tiltDirection?: TiltDirection;
  perspective?: number;
  visibleCards?: number;
  falloff?: number;
  blur?: number;
  duration?: number;
  autoplay?: boolean;
  autoplayDelay?: number;
  loop?: boolean;
  showControls?: boolean;
  showIndicators?: boolean;
  className?: string;
}

import embedded from "@/assets/exp/embedded_thesis.jpeg";
import ims1 from "@/assets/exp/immersion1.jpeg";
import ims2 from "@/assets/exp/immersion2.jpeg";
import ite1 from "@/assets/exp/ite1.jpg";
import ite2 from "@/assets/exp/ite2.jpeg";
import ite3 from "@/assets/exp/ite3.jpeg";
import ite4 from "@/assets/exp/ite4.jpeg";
import ite5 from "@/assets/exp/ite5.jpeg";
import ite6 from "@/assets/exp/ite6.jpeg";
import mis from "@/assets/exp/mis.jpg";
import durian from "@/assets/exp/durian.jpeg";
import codechum from "@/assets/exp/codechum_nat.jpg";

const props = withDefaults(defineProps<DepthCarouselProps>(), {
  items: () => [
    { image: embedded, alt: "Slide 1" },
    { image: ims1, alt: "Slide 2" },
    { image: ims2, alt: "Slide 3" },
    { image: ite1, alt: "Slide 4" },
    { image: ite2, alt: "Slide 5" },
    { image: ite3, alt: "Slide 6" },
    { image: ite4, alt: "Slide 7" },
    { image: ite5, alt: "Slide 8" },
    { image: ite6, alt: "Slide 9" },
    { image: mis, alt: "10" },
    { image: durian, alt: "Slide 11" },
    { image: codechum, alt: "Slide 12" },
  ],
  cardWidth: 300,
  cardHeight: 380,
  radius: 18,
  tint: "#05060a",
  depth: 220,
  spread: 90,
  tilt: 22,
  tiltDirection: "right",
  perspective: 1400,
  visibleCards: 4,
  falloff: 0.2,
  blur: 6,
  duration: 700,
  autoplay: false,
  autoplayDelay: 3200,
  loop: true,
  showControls: true,
  showIndicators: true,
  className: "",
});

const emit = defineEmits<{
  (e: "change", index: number, item: { image: string; alt?: string }): void;
}>();

const clamp = (v: number, min: number, max: number) =>
  Math.min(Math.max(v, min), max);

const easePower3Out = (t: number) => {
  const f = 1 - t;
  return 1 - f * f * f;
};

const normalizeItem = (it: DepthCarouselItem) =>
  typeof it === "string" ? { image: it, alt: "" } : it;

const normalizedData = computed(() =>
  (Array.isArray(props.items) ? props.items : []).map(normalizeItem),
);

const count = computed(() => normalizedData.value.length);
const isMobile = ref(
  typeof window !== "undefined" &&
    window.matchMedia("(max-width: 767px)").matches,
);

const displayCardWidth = computed(() =>
  isMobile.value ? props.cardWidth : props.cardWidth * 2,
);
const displayCardHeight = computed(() =>
  isMobile.value ? props.cardHeight : 450,
);

const containerWidth = ref(0);
const containerHeight = ref(0);
const active = ref(0);

const rootRef = ref<HTMLDivElement | null>(null);
const cardRefs = ref<(HTMLDivElement | null)[]>([]);
const overlayRefs = ref<(HTMLSpanElement | null)[]>([]);

const setCardRef = (el: any, index: number) => {
  if (el) cardRefs.value[index] = el;
};
const setOverlayRef = (el: any, index: number) => {
  if (el) overlayRefs.value[index] = el;
};

let pos = 0;
let focus = 0;
let animFrameId: number | null = null;
let layoutFrameId: number | null = null;
let scale = 1;

const isLowPowerDevice =
  typeof navigator !== "undefined" &&
  ((navigator.hardwareConcurrency || 4) <= 4 ||
    ((navigator as Navigator & { deviceMemory?: number }).deviceMemory || 8) <=
      4);

const updateScale = (width: number, height: number) => {
  containerWidth.value = width;
  containerHeight.value = height;

  if (!width || !height || !count.value) return;

  const viewportWidth =
    typeof window !== "undefined" ? window.innerWidth : width;
  const viewportHeight =
    typeof window !== "undefined" ? window.innerHeight : height;

  let targetScale;

  if (isMobile.value) {
    const mobileTargetScaleX =
      (viewportWidth * 0.65) / Math.max(1, displayCardWidth.value);
    const mobileTargetScaleY =
      (viewportHeight * 0.6) / Math.max(1, displayCardHeight.value);
    targetScale = Math.min(mobileTargetScaleX, mobileTargetScaleY);
  } else {
    targetScale = (viewportWidth * 0.6) / Math.max(1, displayCardWidth.value);
  }

  const availableWidth = Math.max(width - 100, 1);
  const availableHeight = Math.max(height - 32, 1);
  const widthScale = availableWidth / Math.max(1, displayCardWidth.value);
  const heightScale = availableHeight / Math.max(1, displayCardHeight.value);

  scale = Math.max(0.01, Math.min(targetScale, widthScale, heightScale));
};

const isDragging = ref(false);
const dragState = {
  active: false,
  x: 0,
  startPos: 0,
  lastX: 0,
  lastT: 0,
  v: 0,
  moved: false,
  id: 0,
  stepPx: 100,
};

let autoTimer: ReturnType<typeof setInterval> | null = null;
let mobileMediaQuery: MediaQueryList | null = null;
let reduced = false;
let resizeObserver: ResizeObserver | null = null;
let isHovered = false;
let isFocused = false;

const onMouseEnter = () => (isHovered = true);
const onMouseLeave = () => (isHovered = false);
const onFocusIn = () => (isFocused = true);
const onFocusOut = () => (isFocused = false);

const onMediaChange = (e: MediaQueryListEvent) => {
  isMobile.value = e.matches;
};

const killTween = () => {
  if (animFrameId !== null) {
    cancelAnimationFrame(animFrameId);
    animFrameId = null;
  }
};

const scheduleLayout = () => {
  if (layoutFrameId !== null) return;
  layoutFrameId = requestAnimationFrame(() => {
    layoutFrameId = null;
    layout(pos);
  });
};

const cancelScheduledLayout = () => {
  if (layoutFrameId !== null) {
    cancelAnimationFrame(layoutFrameId);
    layoutFrameId = null;
  }
};

const layout = (currentPos: number) => {
  const n = count.value;
  if (!n) return;

  const dir = props.tiltDirection === "left" ? -1 : 1;
  const sc = scale;
  const configuredVisibleLimit = props.visibleCards + 0.5;
  const invVisibleCards = 1 / Math.max(1, props.visibleCards);

  const scaledCardWidth = displayCardWidth.value * sc;
  const availableHalfWidth = Math.max(
    0,
    (containerWidth.value - 100) / 2 - scaledCardWidth / 2,
  );
  const desiredSpreadPx = Math.abs(props.spread) * sc;
  const maxSpreadPx =
    configuredVisibleLimit > 0
      ? availableHalfWidth / configuredVisibleLimit
      : desiredSpreadPx;

  const minSpread = isMobile.value ? 25 * sc : 0;
  const effectiveSpread =
    desiredSpreadPx > 0
      ? Math.max(minSpread, Math.min(desiredSpreadPx, maxSpreadPx)) /
        Math.max(sc, 0.0001)
      : 0;

  const visibleLimit = configuredVisibleLimit;

  for (let i = 0; i < n; i++) {
    const el = cardRefs.value[i];
    if (!el) continue;

    let d = i - currentPos;
    if (props.loop && n > 1) {
      d = ((d % n) + n) % n;
      if (d > n / 2) d -= n;
    }

    const back = Math.max(0, d);
    const az = Math.abs(d);
    const shown = az <= visibleLimit;

    if (!shown) {
      el.style.opacity = "0";
      el.style.visibility = "hidden";
      el.style.pointerEvents = "none";
      const ov = overlayRefs.value[i];
      if (ov) ov.style.opacity = "0";
      continue;
    }

    const tz = -props.depth * d;
    const tx = dir * effectiveSpread * d;
    const ry = dir * props.tilt * clamp(d, 0, 1);
    const opacity = d < 0 ? Math.max(0, 1 + d) : 1;

    const simulatedDarkness = clamp(back * props.falloff * 1.5, 0, 0.85);
    const blurPx =
      !isLowPowerDevice && props.blur > 0
        ? Math.min(props.blur, back * invVisibleCards * props.blur)
        : 0;
    const zi = Math.round(2000 - d * 20);

    el.style.transform = `translate(-50%, -50%) scale(${sc}) translate3d(${tx.toFixed(2)}px, 0px, ${tz.toFixed(2)}px) rotateY(${ry.toFixed(3)}deg)`;
    el.style.opacity = opacity.toFixed(3);
    el.style.visibility = "visible";
    el.style.zIndex = String(zi);
    el.style.pointerEvents = shown && opacity > 0.05 ? "auto" : "none";
    el.style.filter = blurPx > 0 ? `blur(${blurPx.toFixed(2)}px)` : "none";

    const ov = overlayRefs.value[i];
    if (ov) ov.style.opacity = simulatedDarkness.toFixed(3);
  }
};

const notify = (idx: number) => {
  active.value = idx;
  const currentItem = normalizedData.value[idx] ?? { image: "", alt: "" };
  emit("change", idx, currentItem);
};

const tweenTo = (target: number, animate: boolean) => {
  killTween();
  const startPos = pos;
  const change = target - startPos;
  const dur = animate && !reduced ? props.duration : 0;

  if (dur <= 0) {
    pos = target;
    const n = count.value;
    if (n > 0) pos = ((pos % n) + n) % n;
    layout(pos);
    return;
  }

  let startTime: number | null = null;
  const step = (currentTime: number) => {
    if (!startTime) startTime = currentTime;
    const elapsed = currentTime - startTime;
    const progress = Math.min(elapsed / dur, 1);
    const easedProgress = easePower3Out(progress);

    pos = startPos + change * easedProgress;
    layout(pos);

    if (progress < 1) {
      animFrameId = requestAnimationFrame(step);
    } else {
      animFrameId = null;
      const n = count.value;
      if (n > 0) pos = ((pos % n) + n) % n;
      layout(pos);
    }
  };

  animFrameId = requestAnimationFrame(step);
};

const setFocus = (rawIndex: number, animate = true) => {
  const n = count.value;
  if (!n) return;

  const idx = props.loop ? ((rawIndex % n) + n) % n : clamp(rawIndex, 0, n - 1);
  let delta = idx - pos;

  if (props.loop && n > 1) {
    delta = ((delta % n) + n) % n;
    if (delta > n / 2) delta -= n;
  }

  tweenTo(pos + delta, animate);
  if (idx !== focus) {
    focus = idx;
    notify(idx);
  }
};

const navigateBy = (step: number) => setFocus(focus + step, true);

const onPointerDown = (e: PointerEvent) => {
  if (count.value < 2) return;
  killTween();
  cancelScheduledLayout();

  dragState.active = true;
  dragState.x = e.clientX;
  dragState.startPos = pos;
  dragState.lastX = e.clientX;
  dragState.lastT = performance.now();
  dragState.v = 0;
  dragState.moved = false;
  dragState.id = e.pointerId;

  dragState.stepPx = Math.max(displayCardWidth.value * 0.85 * scale, 60);
};

const onPointerMove = (e: PointerEvent) => {
  if (!dragState.active) return;

  const dx = e.clientX - dragState.x;

  if (!dragState.moved && Math.abs(dx) > 12) {
    dragState.moved = true;
    isDragging.value = true;
    rootRef.value?.setPointerCapture(dragState.id);
  }

  if (!dragState.moved) return;

  e.preventDefault();
  const now = performance.now();
  const dt = Math.max(now - dragState.lastT, 1);

  dragState.v = (e.clientX - dragState.lastX) / dt;
  dragState.lastX = e.clientX;
  dragState.lastT = now;
  pos = dragState.startPos - dx / dragState.stepPx;
  scheduleLayout();
};

const onPointerEnd = () => {
  if (!dragState.active) return;
  dragState.active = false;
  isDragging.value = false;

  if (!dragState.moved) return;
  cancelScheduledLayout();

  const projected = pos - (dragState.v * 150) / dragState.stepPx;
  setFocus(Math.round(projected), true);
};

const onKeyDown = (e: KeyboardEvent) => {
  if (e.key === "ArrowLeft") {
    e.preventDefault();
    navigateBy(-1);
  } else if (e.key === "ArrowRight") {
    e.preventDefault();
    navigateBy(1);
  }
};

const onCardClick = (index: number) => {
  if (dragState.moved) return;

  if (focus === index) {
    const targetImage = normalizedData.value[index]?.image;
    if (targetImage) {
      openPopup(targetImage);
    }
  } else {
    setFocus(index, true);
  }
};

const stopAutoplay = () => {
  if (autoTimer) clearInterval(autoTimer);
  autoTimer = null;
};

const startAutoplay = () => {
  stopAutoplay();
  autoTimer = setInterval(
    () => {
      if (!isHovered && !isFocused && !isPopupOpen.value) navigateBy(1);
    },
    Math.max(props.autoplayDelay, 600),
  );
};

onMounted(() => {
  const root = rootRef.value;
  if (!root) return;

  reduced =
    typeof window !== "undefined" &&
    window.matchMedia("(prefers-reduced-motion: reduce)").matches;

  mobileMediaQuery = window.matchMedia("(max-width: 767px)");
  if (mobileMediaQuery.addEventListener) {
    mobileMediaQuery.addEventListener("change", onMediaChange);
  } else if ((mobileMediaQuery as any).addListener) {
    (mobileMediaQuery as any).addListener(onMediaChange);
  }

  resizeObserver = new ResizeObserver((entries) => {
    const entry = entries[0];
    if (!entry) return;
    const { width, height } = entry.contentRect;
    updateScale(width, height);
    layout(pos);
  });

  resizeObserver.observe(root);

  if (props.autoplay && !reduced && count.value > 1) {
    root.addEventListener("mouseenter", onMouseEnter, { passive: true });
    root.addEventListener("mouseleave", onMouseLeave, { passive: true });
    root.addEventListener("focusin", onFocusIn, { passive: true });
    root.addEventListener("focusout", onFocusOut, { passive: true });
    startAutoplay();
  }

  updateScale(root.clientWidth, root.clientHeight);
  layout(pos);
});

onBeforeUnmount(() => {
  killTween();
  cancelScheduledLayout();
  stopAutoplay();
  if (mobileMediaQuery) {
    if (mobileMediaQuery.removeEventListener) {
      mobileMediaQuery.removeEventListener("change", onMediaChange);
    } else if ((mobileMediaQuery as any).removeListener) {
      (mobileMediaQuery as any).removeListener(onMediaChange);
    }
  }
  rootRef.value?.removeEventListener("mouseenter", onMouseEnter);
  rootRef.value?.removeEventListener("mouseleave", onMouseLeave);
  rootRef.value?.removeEventListener("focusin", onFocusIn);
  rootRef.value?.removeEventListener("focusout", onFocusOut);
  if (resizeObserver) resizeObserver.disconnect();
});

watch(
  [
    () => props.depth,
    () => props.spread,
    () => props.tilt,
    () => props.tiltDirection,
    () => props.visibleCards,
    () => props.falloff,
    () => props.blur,
    () => props.cardWidth,
    () => props.cardHeight,
    () => props.radius,
    () => isMobile.value,
    () => props.items,
  ],
  () => layout(pos),
);
</script>

<style scoped>
.depth-carousel {
  position: relative;
  width: 100%;
  height: 100%;
  min-height: 320px;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  perspective: var(--dc-perspective, 1400px);
  perspective-origin: 50% 50%;
  touch-action: pan-y;
  outline: none;
  user-select: none;
  -webkit-user-select: none;
  isolation: isolate;
  cursor: default;
}

.depth-carousel.is-dragging {
  cursor: grabbing;
}

.depth-carousel__stage {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 5%;
  width: 90%;
  transform-style: preserve-3d;
  overflow: visible;
}

.depth-carousel__card {
  position: absolute;
  top: 50%;
  left: 50%;
  transform-origin: center center;
  overflow: hidden;
  background: #0b0d12;
  box-shadow:
    0 30px 60px -20px rgba(0, 0, 0, 0.65),
    0 8px 20px -10px rgba(0, 0, 0, 0.5);
  will-change: transform, opacity;
  backface-visibility: hidden;
  cursor: grab;
  transform: translate(-50%, -50%);
}

.depth-carousel.is-dragging .depth-carousel__card {
  cursor: grabbing;
}

.depth-carousel__img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
  pointer-events: none;
  -webkit-user-drag: none;
}

.depth-carousel__tint {
  position: absolute;
  inset: 0;
  opacity: 0;
  pointer-events: none;
  mix-blend-mode: multiply;
  will-change: opacity;
}

.depth-carousel__arrow {
  position: absolute;
  top: 50%;
  z-index: 3000;
  width: 42px;
  height: 42px;
  display: grid;
  place-items: center;
  border: 1px solid rgba(255, 255, 255, 0.18);
  border-radius: 999px;
  background: rgba(18, 20, 26, 0.55);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  color: #fff;
  cursor: pointer;
  transition:
    background 0.2s ease,
    border-color 0.2s ease,
    transform 0.2s ease;
}

.depth-carousel__arrow:hover {
  background: rgba(28, 31, 40, 0.85);
  border-color: rgba(255, 255, 255, 0.4);
}

.depth-carousel__arrow--prev {
  left: max(16px, 3%);
  transform: translateY(-50%);
}

.depth-carousel__arrow--next {
  right: max(16px, 3%);
  transform: translateY(-50%);
}

.depth-carousel__arrow--prev:active,
.depth-carousel__arrow--next:active {
  transform: translateY(-50%) scale(0.94);
}

.popup-overlay {
  position: fixed;
  inset: 0;
  z-index: 99999;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: rgba(0, 0, 0, 0.9);
  padding: 1rem;
}

.popup-close {
  position: absolute;
  top: 1rem;
  right: 1rem;
  color: white;
  font-size: 2rem;
  line-height: 1;
  font-weight: bold;
  cursor: pointer;
  background: transparent;
  border: none;
  padding: 0.5rem;
}

.popup-img {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.5);
  border-radius: 8px;
}

.fade-popup-enter-active,
.fade-popup-leave-active {
  transition: opacity 0.3s ease;
}
.fade-popup-enter-from,
.fade-popup-leave-to {
  opacity: 0;
}

@media (min-width: 768px) {
  .depth-carousel {
    height: calc(40vw + 120px);
    min-height: 600px;
  }
}

@media (max-width: 767px) {
  .depth-carousel {
    height: 80vh;
    min-height: 420px;
  }
}

@media (prefers-reduced-motion: reduce) {
  .depth-carousel__card {
    will-change: auto;
  }
  .depth-carousel__arrow {
    transition: none;
  }
}
</style>
