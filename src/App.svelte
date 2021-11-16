<script lang="ts">
  import { onMount } from "svelte";
  import { generatePerlin } from "./perlin/Perlin.svelte";

  let canvas: HTMLCanvasElement;

  onMount(() => {
    const canvasContext = canvas.getContext("2d");
    let frame: number;

    canvasContext.beginPath();
    generatePerlin(1, 8, {
      canvas,
      canvasContext,
    });

    (function loop() {
      frame = requestAnimationFrame(loop);

      const imageData = canvasContext.getImageData(
        0,
        0,
        canvas.width,
        canvas.height
      );

      for (let p = 0; p < imageData.data.length; p += 4) {
        const i = p / 4;
        const x = i % canvas.width;
        const y = (i / canvas.height) >>> 0;

        const t = window.performance.now();

        const r = 64 + (128 * x) / canvas.width + 64 * Math.sin(t / 1000);
        const g = 64 + (128 * y) / canvas.height + 64 * Math.cos(t / 1400);
        const b = 128;

        imageData.data[p + 0] = r;
        imageData.data[p + 1] = g;
        imageData.data[p + 2] = b;
        imageData.data[p + 3] = 255;
      }

      canvasContext.putImageData(imageData, 0, 0);
    })();

    return () => {
      cancelAnimationFrame(frame);
    };
  });
</script>

<canvas bind:this={canvas} width={32} height={32} />

<style>
  canvas {
    width: 100%;
    height: 100%;
    /* background-color: #666;
    -webkit-mask: url(logo.png) 50% 50% no-repeat;
    mask: url(logo.png) 50% 50% no-repeat; */
  }
</style>
