<template>
  <canvas
    ref="canvasRef"
    :class="['size-full block rounded-[inherit]', !transparent && 'bg-black']"
    :style="{
      width: width ? `${width}px` : undefined,
      height: height ? `${height}px` : undefined,
    }"
  />
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeUnmount, watch } from "vue";

interface MatrixRainProps {
  variant?: "default" | "cyan" | "rainbow";
  width?: number;
  height?: number;
  fontSize?: number;
  speed?: number;
  fixedColor?: string;
  transparent?: boolean;
}

const props = withDefaults(defineProps<MatrixRainProps>(), {
  variant: "default",
  fontSize: 16,
  speed: 50,
  transparent: false,
});

const canvasRef = ref<HTMLCanvasElement | null>(null);

let rafId: number;
let resizeObserver: ResizeObserver | null = null;

let drops: number[] = [];
let lastDrawTime = 0;
const chars =
  "ｦｧｨｩｪｫｬｭｮｯｰｱｲｳｴｵｶｷｸｹｺｻｼｽｾｿﾀﾁﾂﾃﾄﾅﾆﾇﾈﾉﾊﾋﾌﾍﾎﾏﾐﾑﾒﾓﾔﾕﾖﾗﾘﾙﾚﾛﾜﾝ1234567890";
const charsLen = chars.length;

const activeCells = new Map<
  string,
  { col: number; row: number; char: string; life: number; baseColor: string }
>();
let lastCol = -1,
  lastRow = -1;

const addActiveCell = (col: number, row: number) => {
  const key = `${col}-${row}`;
  const cell = activeCells.get(key);
  if (cell) {
    cell.life = 1.0;
  } else {
    const isRainbow = props.variant === "rainbow" && !props.fixedColor;
    const baseColor = isRainbow
      ? `hsl(${(Date.now() / 20 + col * 10) % 360},100%,50%)`
      : props.fixedColor || (props.variant === "cyan" ? "#0FF" : "#0F0");
    activeCells.set(key, {
      col,
      row,
      char: chars.charAt((Math.random() * charsLen) | 0),
      life: 1.0,
      baseColor,
    });
  }
};

const onPointerMove = (e: MouseEvent | TouchEvent) => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  const clientX = "touches" in e ? e.touches[0]?.clientX : e.clientX;
  const clientY = "touches" in e ? e.touches[0]?.clientY : e.clientY;
  if (clientX === undefined || clientY === undefined) return;

  const rect = canvas.getBoundingClientRect();
  const x = clientX - rect.left;
  const y = clientY - rect.top;

  if (x < 0 || x > rect.width || y < 0 || y > rect.height) {
    lastCol = lastRow = -1;
    return;
  }

  const col = (x / props.fontSize) | 0;
  const row = (y / props.fontSize) | 0;

  if (lastCol !== -1 && lastRow !== -1) {
    let x0 = lastCol,
      y0 = lastRow;
    const dx = Math.abs(col - x0),
      dy = Math.abs(row - y0);
    const sx = x0 < col ? 1 : -1,
      sy = y0 < row ? 1 : -1;
    let err = dx - dy;

    while (true) {
      addActiveCell(x0, y0);
      if (x0 === col && y0 === row) break;
      const e2 = 2 * err;
      if (e2 > -dy) {
        err -= dy;
        x0 += sx;
      }
      if (e2 < dx) {
        err += dx;
        y0 += sy;
      }
    }
  } else {
    addActiveCell(col, row);
  }

  lastCol = col;
  lastRow = row;
};

const onPointerLeave = () => {
  lastCol = lastRow = -1;
};

const initMatrix = () => {
  const canvas = canvasRef.value;
  if (!canvas) return;

  canvas.width = props.width || canvas.offsetWidth;
  canvas.height = props.height || canvas.offsetHeight;

  drops = new Array((canvas.width / props.fontSize) | 0).fill(1);
  activeCells.clear();

  const ctx = canvas.getContext("2d");
  if (ctx && !props.transparent) {
    ctx.fillStyle = "#000000";
    ctx.fillRect(0, 0, canvas.width, canvas.height);
  }
};

const draw = (timestamp: number) => {
  rafId = requestAnimationFrame(draw);

  if (timestamp - lastDrawTime < props.speed) return;
  lastDrawTime = timestamp;

  const canvas = canvasRef.value;
  const ctx = canvas?.getContext("2d");
  if (!canvas || !ctx) return;

  const w = canvas.width,
    h = canvas.height,
    size = props.fontSize;

  const isRainbow = props.variant === "rainbow" && !props.fixedColor;
  const staticColor =
    props.fixedColor || (props.variant === "cyan" ? "#0FF" : "#0F0");

  ctx.globalCompositeOperation = props.transparent
    ? "destination-out"
    : "source-over";
  ctx.fillStyle = "rgba(0, 0, 0, 0.05)";
  ctx.fillRect(0, 0, w, h);
  ctx.globalCompositeOperation = "source-over";
  ctx.font = `${size}px monospace`;

  for (let i = 0; i < drops.length; i++) {
    const text = chars.charAt((Math.random() * charsLen) | 0);
    const drop = drops[i] as number;
    const y = drop * size;

    ctx.fillStyle = isRainbow
      ? `hsl(${(Date.now() / 20 + i * 10) % 360}, 100%, 50%)`
      : staticColor;
    ctx.fillText(text, i * size, y);

    drops[i] = y > h && Math.random() > 0.975 ? 0 : drop + 1;
  }

  for (const [key, cell] of activeCells.entries()) {
    const x = cell.col * size,
      rectY = cell.row * size,
      textY = rectY + size;

    ctx.globalCompositeOperation = props.transparent
      ? "destination-out"
      : "source-over";
    ctx.fillStyle = "#000000";
    ctx.fillRect(x, rectY, size, size);
    ctx.globalCompositeOperation = "source-over";

    ctx.fillStyle = cell.baseColor;
    ctx.fillText(cell.char, x, textY);
    ctx.fillStyle = `rgba(255, 255, 255, ${cell.life})`;
    ctx.fillText(cell.char, x, textY);

    if ((cell.life -= 0.05) <= 0) {
      activeCells.delete(key);
    }
  }
};

watch(
  () => [props.width, props.height, props.fontSize, props.transparent],
  initMatrix,
);

onMounted(() => {
  if (canvasRef.value) {
    resizeObserver = new ResizeObserver(() => {
      if (!props.width && !props.height) initMatrix();
    });
    resizeObserver.observe(canvasRef.value);
  }

  window.addEventListener("mousemove", onPointerMove);
  document.addEventListener("mouseleave", onPointerLeave);

  window.addEventListener("touchmove", onPointerMove, { passive: true });
  window.addEventListener("touchstart", onPointerMove, { passive: true });
  window.addEventListener("touchend", onPointerLeave);
  window.addEventListener("touchcancel", onPointerLeave);

  initMatrix();
  rafId = requestAnimationFrame(draw);
});

onBeforeUnmount(() => {
  cancelAnimationFrame(rafId);
  resizeObserver?.disconnect();

  window.removeEventListener("mousemove", onPointerMove);
  document.removeEventListener("mouseleave", onPointerLeave);

  window.removeEventListener("touchmove", onPointerMove);
  window.removeEventListener("touchstart", onPointerMove);
  window.removeEventListener("touchend", onPointerLeave);
  window.removeEventListener("touchcancel", onPointerLeave);
});
</script>
