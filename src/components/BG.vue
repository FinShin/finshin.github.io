<template>
  <div class="bg-wrapper">
    <div class="hacker-layer">
      <div
        v-for="block in blocks"
        :key="block.id"
        class="code-block"
        :class="{ 'is-fading': block.state === 'fading' }"
        :style="{
          left: `${block.x}%`,
          top: `${block.y}%`,
          opacity: block.opacity,
        }"
      >
        <span v-html="highlight(block.text)"></span>
        <span v-show="block.state === 'typing'" class="cursor">█</span>
      </div>
    </div>

    <div class="vignette-overlay"></div>

    <div class="content-container">
      <slot />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";

const snippets = [
  "import { ref, computed } from 'vue';\nconst isActive = ref(false);",
  "SELECT id, hash, last_login\nFROM sys_users\nWHERE access_level > 9000;",
  "0x7FFF8000:  movq   %rsp, %rbp\n0x7FFF8003:  popq   %rbp",
  "sudo systemctl restart nginx\n[OK] Service restarted.",
  "function allocate_memory(size) {\n  let ptr = malloc(size);\n  return ptr;\n}",
  "git commit -m 'feat: neural network core'\ngit push origin main",
  "curl -X POST https://api.dev/v1/auth \\\n  -H 'Authorization: Bearer token'",
  "// Initializing core systems...\nSystem.boot({ secure: true });",
  "<!-- Layout Wrapper -->\n<div class='flex flex-col h-screen'>",
  "while (thread.isAlive()) {\n  await processQueue();\n}",
];

const reEscape = /[&<>]/g;
const mapEscape = { "&": "&amp;", "<": "&lt;", ">": "&gt;" };
const reComments = /(\/\/.*|&lt;!--.*?--&gt;)/g;
const reStrings = /('.*?'|".*?")/g;
const reKeywords =
  /\b(import|from|const|function|let|return|SELECT|FROM|WHERE|sudo|git|curl|while|await)\b/g;
const reFuncs =
  /\b(ref|computed|isActive|malloc|systemctl|thread|processQueue|System|boot|id|hash|last_login|access_level)\b/g;
const reNumbers = /\b(0x[0-9A-F]+|\d+)\b/g;
const reGlitch = /[a-zA-Z0-9_]/g;

const highlight = (text) => {
  if (!text) return "";
  return text
    .replace(reEscape, (m) => mapEscape[m])
    .replace(reComments, '<span class="hl-comment">$1</span>')
    .replace(reStrings, '<span class="hl-string">$1</span>')
    .replace(reKeywords, '<span class="hl-keyword">$1</span>')
    .replace(reFuncs, '<span class="hl-func">$1</span>')
    .replace(reNumbers, '<span class="hl-number">$1</span>');
};

const MAX_BLOCKS = 6;
const blocks = ref(
  Array.from({ length: MAX_BLOCKS }, (_, i) => ({
    id: i,
    text: "",
    fullText: "",
    x: 0,
    y: 0,
    opacity: 0,
    state: "waiting",
    timer: Math.random() * 3000,
    charIndex: 0,
    glitchTicks: 0,
  })),
);

let animationFrameId;
let lastTime = performance.now();

const TARGET_FPS = 30;
const frameDelay = 1000 / TARGET_FPS;

const updateLoop = (time) => {
  animationFrameId = requestAnimationFrame(updateLoop);

  const dt = time - lastTime;
  if (dt < frameDelay) return;

  lastTime = time - (dt % frameDelay);

  blocks.value.forEach((block) => {
    block.timer -= dt;

    if (block.timer <= 0) {
      switch (block.state) {
        case "waiting":
          block.fullText =
            snippets[Math.floor(Math.random() * snippets.length)];
          block.text = "";
          block.charIndex = 0;

          // Spatial distribution
          const col = block.id % 2;
          const row = Math.floor(block.id / 2);
          const minX = col === 0 ? 5 : 60;
          const maxX = col === 0 ? 25 : 80;
          const minY = row === 0 ? 5 : row === 1 ? 40 : 75;
          const maxY = row === 0 ? 15 : row === 1 ? 50 : 85;

          block.x = minX + Math.random() * (maxX - minX);
          block.y = minY + Math.random() * (maxY - minY);

          block.opacity = 1;
          block.state = "typing";
          block.timer = 20 + Math.random() * 40;
          break;

        case "typing":
          block.charIndex += 1;
          block.text = block.fullText.slice(0, block.charIndex);
          if (block.charIndex >= block.fullText.length) {
            block.state = "code_idle";
            block.timer = 3000 + Math.random() * 4000;
          } else {
            block.timer = 15 + Math.random() * 30;
          }
          break;

        case "code_idle":
          block.state = "glitching";
          block.glitchTicks = 0;
          block.timer = 40;
          break;

        case "glitching":
          block.glitchTicks++;
          if (block.glitchTicks > 12) {
            block.text = block.fullText.replace(/[^\s\n]/g, () =>
              Math.random() > 0.5 ? "1" : "0",
            );
            block.state = "binary_idle";
            block.timer = 1500 + Math.random() * 2000;
          } else {
            block.text = block.fullText.replace(reGlitch, (char) =>
              Math.random() > 0.7 ? (Math.random() > 0.5 ? "1" : "0") : char,
            );
            block.timer = 60;
          }
          break;

        case "binary_idle":
          block.opacity = 0;
          block.state = "fading";
          block.timer = 2000;
          break;

        case "fading":
          block.state = "waiting";
          block.timer = 1000 + Math.random() * 3000;
          break;
      }
    }
  });
};

onMounted(() => {
  lastTime = performance.now();
  animationFrameId = requestAnimationFrame(updateLoop);
});

onBeforeUnmount(() => {
  cancelAnimationFrame(animationFrameId);
});
</script>

<style>
.bg-wrapper {
  position: relative;
  width: 100%;
  min-height: 100vh;
  background-color: #101010;
  overflow: hidden;
}

.hacker-layer {
  position: absolute;
  inset: 0;
  z-index: 0;
  pointer-events: none;
  filter: blur(2.5px);
  opacity: 0.85;
}

.code-block {
  position: absolute;
  font-family: "Fira Code", "Courier New", Courier, monospace;
  font-size: 1.5rem;
  line-height: 1.5;
  color: #f8fafc;
  white-space: pre-wrap;
  user-select: none;
  opacity: 1;
}

.code-block.is-fading {
  transition: opacity 2s ease-out;
}

.hl-keyword {
  color: #f472b6;
  text-shadow: 0 0 5px rgba(244, 114, 182, 0.4);
}
.hl-string {
  color: #fde047;
  text-shadow: 0 0 5px rgba(253, 224, 71, 0.4);
}
.hl-func {
  color: #4ade80;
  text-shadow: 0 0 5px rgba(74, 222, 128, 0.4);
}
.hl-number {
  color: #38bdf8;
  text-shadow: 0 0 5px rgba(56, 189, 248, 0.4);
}
.hl-comment {
  color: #94a3b8;
  font-style: italic;
}

.cursor {
  display: inline-block;
  width: 8px;
  background-color: #38bdf8;
  box-shadow: 0 0 5px #38bdf8;
  animation: blink 0.8s steps(2, start) infinite;
  vertical-align: text-bottom;
}

@keyframes blink {
  to {
    visibility: hidden;
  }
}

.vignette-overlay {
  position: absolute;
  inset: 0;
  background: radial-gradient(
    circle at center,
    transparent 10%,
    rgba(16, 16, 16, 0.8) 85%,
    rgba(16, 16, 16, 1) 100%
  );
  z-index: 1;
  pointer-events: none;
}

.content-container {
  position: relative;
  z-index: 2;
  width: 100%;
  min-height: 100vh;
}
</style>
