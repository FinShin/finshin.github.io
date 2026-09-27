<template>
  <div class="social-container">
    <!-- TOP NOTIFICATION BUBBLE -->
    <Transition name="toast-pop">
      <div v-if="toast.visible" class="copy-toast">
        <svg viewBox="0 0 24 24" class="toast-icon">
          <path d="M9 16.17L4.83 12l-1.42 1.41L9 19 21 7l-1.41-1.41z" />
        </svg>
        <span>{{ toast.message }}</span>
      </div>
    </Transition>

    <div class="social-row">
      <a
        v-for="item in socialLinks"
        :key="item.id"
        :href="item.href || '#'"
        class="icon-btn"
        :data-brand="item.id"
        :aria-label="item.name"
        target="_blank"
        rel="noopener noreferrer"
        @click="handleClick($event, item)"
      >
        <div class="splash-ring"></div>

        <div
          v-for="(drop, index) in item.droplets"
          :key="index"
          class="paint-droplet"
          :style="{
            '--tx': `${drop.tx}px`,
            '--ty': `${drop.ty}px`,
            '--size': `${drop.size}px`,
            '--end-scale': drop.endScale,
          }"
        ></div>

        <svg viewBox="0 0 24 24" class="icon-svg">
          <path :d="copiedStates[item.id] ? checkSvgPath : item.svgPath" />
        </svg>
      </a>
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive } from "vue";

interface Droplet {
  tx: number;
  ty: number;
  size: number;
  endScale: number;
}

interface SocialLink {
  id: string;
  name: string;
  href?: string;
  textToCopy?: string;
  svgPath: string;
  droplets: Droplet[];
}

const generateDroplets = (num = 12): Droplet[] => {
  return Array.from({ length: num }).map((_, i) => {
    const angle = (i / num) * 360 + (Math.random() * 20 - 10);
    const distance = 40 + Math.random() * 50;
    const radians = angle * (Math.PI / 180);
    return {
      tx: Math.cos(radians) * distance,
      ty: Math.sin(radians) * distance,
      size: 4 + Math.random() * 8,
      endScale: 0.2 + Math.random() * 0.8,
    };
  });
};

const checkSvgPath =
  "M9 16.17L4.83 12l-1.42 1.41L9 19 21 7l-1.41-1.41L9 16.17z";

const socialLinks: SocialLink[] = [
  {
    id: "linkedin",
    name: "LinkedIn",
    href: "https://www.linkedin.com/in/jirho-enciso",
    svgPath:
      "M19 3a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h14m-.5 15.5v-5.3a3.26 3.26 0 0 0-3.26-3.26c-.85 0-1.84.52-2.28 1.3v-1.11h-2.79v8.37h2.79v-4.93c0-.77.62-1.4 1.39-1.4a1.4 1.4 0 0 1 1.4 1.4v4.93h2.75M6.88 8.56a1.68 1.68 0 0 0 1.68-1.68c0-.93-.75-1.69-1.68-1.69a1.69 1.69 0 0 0-1.69 1.69c0 .93.76 1.68 1.69 1.68m1.39 9.94v-8.37H5.5v8.37h2.77z",
    droplets: generateDroplets(),
  },
  {
    id: "facebook",
    name: "Facebook",
    href: "https://www.facebook.com/jirho.enciso.2025",
    svgPath:
      "M12 2.04C6.5 2.04 2 6.53 2 12.06C2 17.06 5.66 21.21 10.44 21.96V14.96H7.9V12.06H10.44V9.85C10.44 7.34 11.93 5.96 14.22 5.96C15.31 5.96 16.45 6.15 16.45 6.15V8.62H15.19C13.95 8.62 13.56 9.39 13.56 10.18V12.06H16.34L15.89 14.96H13.56V21.96A10 10 0 0 0 22 12.06C22 6.53 17.5 2.04 12 2.04Z",
    droplets: generateDroplets(),
  },
  {
    id: "github",
    name: "GitHub",
    href: "https://github.com/FinShin",
    svgPath:
      "M12 2A10 10 0 0 0 2 12c0 4.42 2.87 8.17 6.84 9.5.5.08.66-.23.66-.5v-1.69c-2.77.6-3.36-1.34-3.36-1.34-.46-1.16-1.11-1.47-1.11-1.47-.91-.62.07-.6.07-.6 1 .07 1.53 1.03 1.53 1.03.87 1.52 2.34 1.07 2.91.83.1-.65.35-1.09.63-1.34-2.22-.25-4.55-1.11-4.55-4.92 0-1.11.38-2 1.03-2.71-.1-.25-.45-1.29.1-2.64 0 0 .84-.27 2.75 1.02.79-.22 1.65-.33 2.5-.33.85 0 1.71.11 2.5.33 1.91-1.29 2.75-1.02 2.75-1.02.55 1.35.2 2.39.1 2.64.65.71 1.03 1.6 1.03 2.71 0 3.82-2.34 4.66-4.57 4.91.36.31.69.92.69 1.85V21c0 .27.16.59.67.5C19.14 20.16 22 16.42 22 12A10 10 0 0 0 12 2z",
    droplets: generateDroplets(),
  },
  {
    id: "gmail",
    name: "Gmail",
    textToCopy: "jirhoenciso1204@gmail.com",
    svgPath:
      "M20 4H4c-1.1 0-1.99.9-1.99 2L2 18c0 1.1.9 2 2 2h16c1.1 0 2-.9 2-2V6c0-1.1-.9-2-2-2zm0 4l-8 5-8-5V6l8 5 8-5v2z",
    droplets: generateDroplets(),
  },
  {
    id: "telegram",
    name: "Telegram",
    textToCopy: "@creepsandesu",
    svgPath:
      "M9.78 18.65l.28-4.28 7.72-6.97c.34-.31-.07-.49-.52-.19L7.74 13.3 3.64 12c-.88-.25-.89-.86.2-1.3l15.97-6.16c.73-.33 1.43.18 1.15 1.3l-2.72 12.81c-.19.91-.74 1.13-1.5.71L12.6 16.3l-1.99 1.93c-.23.23-.42.42-.83.42z",
    droplets: generateDroplets(),
  },
];

const copiedStates = reactive<Record<string, boolean>>({});

const toast = reactive({
  visible: false,
  message: "",
});

let toastTimer: ReturnType<typeof setTimeout> | null = null;
const iconTimers: Record<string, ReturnType<typeof setTimeout>> = {};
const splashTimers: Record<string, ReturnType<typeof setTimeout>> = {};

const copyToClipboard = async (text: string): Promise<boolean> => {
  if (navigator.clipboard && window.isSecureContext) {
    try {
      await navigator.clipboard.writeText(text);
      return true;
    } catch {}
  }

  try {
    const textArea = document.createElement("textarea");
    textArea.value = text;
    textArea.style.cssText =
      "position:fixed;top:-9999px;left:-9999px;opacity:0;";
    document.body.appendChild(textArea);
    textArea.focus();
    textArea.select();
    const success = document.execCommand("copy");
    document.body.removeChild(textArea);
    return success;
  } catch (err) {
    return false;
  }
};

const handleClick = async (event: Event, item: SocialLink) => {
  // Capture DOM target synchronously before any promises run
  const button = event.currentTarget as HTMLElement | null;

  if (item.textToCopy) {
    event.preventDefault();

    const success = await copyToClipboard(item.textToCopy);
    if (success) {
      copiedStates[item.id] = true;
      if (iconTimers[item.id]) clearTimeout(iconTimers[item.id]);

      iconTimers[item.id] = setTimeout(() => {
        copiedStates[item.id] = false;
      }, 2000);

      showToast(`Copied "${item.textToCopy}" to clipboard!`);
    }
  }

  if (button) {
    button.classList.remove("is-splashing");

    requestAnimationFrame(() => {
      requestAnimationFrame(() => {
        button.classList.add("is-splashing");
      });
    });

    if (splashTimers[item.id]) clearTimeout(splashTimers[item.id]);
    splashTimers[item.id] = setTimeout(() => {
      button.classList.remove("is-splashing");
    }, 600);
  }
};

const showToast = (msg: string) => {
  toast.message = msg;
  toast.visible = true;

  if (toastTimer) clearTimeout(toastTimer);
  toastTimer = setTimeout(() => {
    toast.visible = false;
  }, 3500);
};
</script>

<style scoped>
.social-container {
  position: relative;
  width: 100%;
  margin: 0;
  padding: 0;
}

.copy-toast {
  position: fixed;
  top: 10vh;
  left: 50%;
  transform: translateX(-50%);
  z-index: 9999;
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 20px;
  background: radial-gradient(
    150% 150% at 0% 0%,
    rgba(255, 255, 255, 0.18) 0%,
    rgba(255, 255, 255, 0.01) 40%,
    rgba(255, 255, 255, 0.08) 100%
  );
  -webkit-backdrop-filter: blur(12px) saturate(115%) contrast(105%);
  backdrop-filter: blur(12px) saturate(115%) contrast(105%);
  border: 1px solid rgba(255, 255, 255, 0.15);
  border-top-color: rgba(255, 255, 255, 0.45);
  border-left-color: rgba(255, 255, 255, 0.45);
  box-shadow:
    0 15px 35px -5px rgba(0, 0, 0, 0.25),
    inset 0 0 15px rgba(255, 255, 255, 0.08),
    inset 2px 2px 8px rgba(255, 255, 255, 0.35),
    inset -3px -3px 12px rgba(0, 0, 0, 0.2);
  border-radius: 10px;
  color: #ffffff;
  font-size: 0.875rem;
  font-weight: 500;
  pointer-events: none;
  max-width: 90vw;
  text-align: center;
  will-change: transform, opacity;
}

.toast-icon {
  width: 18px;
  height: 18px;
  flex-shrink: 0;
  fill: #34d399;
}

.toast-pop-enter-active {
  transition:
    transform 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275),
    opacity 0.3s ease;
}
.toast-pop-leave-active {
  transition:
    transform 0.3s ease,
    opacity 0.3s ease;
}
.toast-pop-enter-from {
  opacity: 0;
  transform: translate(-50%, -20px) scale(0.85);
}
.toast-pop-leave-to {
  opacity: 0;
  transform: translate(-50%, -20px) scale(0.9);
}

.social-row {
  display: flex;
  flex-direction: row;
  gap: 1.5rem;
  align-items: center;
  justify-content: center;
  padding: 1rem;
}

.icon-btn {
  position: relative;
  width: 10vw;
  height: 10vw;
  min-width: 2.25rem;
  min-height: 2.25rem;
  max-width: 3.25rem;
  max-height: 3.25rem;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  text-decoration: none;
  transition:
    color 0.3s ease,
    border-color 0.3s ease,
    transform 0.2s ease,
    box-shadow 0.3s ease;
  color: #ffffff;
  --brand-color: #ffffff;

  -webkit-tap-highlight-color: transparent;
  transform: translateZ(0);
}

.icon-svg {
  width: 1.5rem;
  height: 1.5rem;
  fill: currentColor;
  z-index: 2;
  transition: transform 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

.icon-btn[data-brand="linkedin"] {
  --brand-color: #0a66c2;
}
.icon-btn[data-brand="facebook"] {
  --brand-color: #1877f2;
}
.icon-btn[data-brand="github"] {
  --brand-color: #6e5494;
}
.icon-btn[data-brand="gmail"] {
  --brand-color: #ea4335;
}
.icon-btn[data-brand="telegram"] {
  --brand-color: #24a1de;
}

@media (hover: hover) {
  .icon-btn:hover {
    color: var(--brand-color);
    border-color: var(--brand-color);
    transform: translateY(-3px);
    box-shadow:
      0 0 20px rgba(255, 255, 255, 0.1),
      0 0 15px var(--brand-color);
  }
  .icon-btn:hover .icon-svg {
    transform: scale(1.15);
  }
}

.icon-btn:active {
  transform: scale(0.92);
}

.splash-ring {
  position: absolute;
  inset: -5px;
  border-radius: 50%;
  border: 3px solid var(--brand-color);
  pointer-events: none;
  opacity: 0;
  z-index: 1;
  will-change: transform, opacity, border-width;
}

.icon-btn.is-splashing .splash-ring {
  animation: paintRing 0.55s cubic-bezier(0.1, 0.8, 0.3, 1) forwards;
}

.paint-droplet {
  position: absolute;
  top: 50%;
  left: 50%;
  width: var(--size, 8px);
  height: var(--size, 8px);
  background-color: var(--brand-color);
  border-radius: 50%;
  pointer-events: none;
  z-index: 3;
  transform: translate(-50%, -50%) scale(0);
  opacity: 1;
  will-change: transform, opacity;
}

.icon-btn.is-splashing .paint-droplet {
  animation: stompSplash 0.5s cubic-bezier(0.08, 0.82, 0.17, 1) forwards;
}

/* Hardware Accelerated Keyframes */
@keyframes paintRing {
  0% {
    transform: scale(0.6);
    opacity: 0.9;
  }
  50% {
    opacity: 0.8;
  }
  100% {
    transform: scale(1.8);
    opacity: 0;
    border-width: 0px;
  }
}

@keyframes stompSplash {
  0% {
    transform: translate(-50%, -50%) scale(0.2);
    opacity: 1;
  }
  70% {
    opacity: 0.9;
  }
  100% {
    transform: translate(calc(-50% + var(--tx)), calc(-50% + var(--ty)))
      scale(var(--end-scale, 1));
    opacity: 0;
  }
}

@media (max-width: 500px) {
  .copy-toast {
    top: 3vh;
  }
}
</style>
