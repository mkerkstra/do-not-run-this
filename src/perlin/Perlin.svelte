<script lang="ts" context="module">
  function generateRandomVector(): Vector {
    const t = Math.random() * 2 * Math.PI;
    return [Math.cos(t), Math.sin(t)];
  }

  /**
   * {@link https://stackoverflow.com/a/45121786/6744229}
   */
  function smoother(x: number) {
    return 6 * x ** 5 - 15 * x ** 4 + 10 * x ** 3;
  }

  function interpolation(distance: number, a: number, b: number) {
    return a + smoother(distance) * (b - a);
  }

  type Vector = [number, number];

  export function generatePerlin(
    seed: number,
    nodes: number,
    canvasOpts: {
      canvas: HTMLCanvasElement;
      canvasContext: CanvasRenderingContext2D;
      resolution?: number;
      color?: number;
    }
  ) {
    const pixelSize = canvasOpts.canvas.width / (canvasOpts.resolution || 128);
    const pixelAmount = nodes / (canvasOpts.resolution || 128);
    const nodeSteps = nodes * canvasOpts.canvas.width;
    let gradients: Vector[][] = [[]];

    function dotProd(x: number, y: number, vectorX: number, vectorY: number) {
      const distance = [x - vectorX, y - vectorY];
      // if this gradient is already in the array, use it
      const newGradient =
        (gradients && gradients[vectorX] && gradients[vectorX][vectorY]) ||
        generateRandomVector();
      // add gradient to array
      const that = gradients[vectorX];
      // return the dot product of the gradient and the distance
      return distance[0] * newGradient[0] + distance[1] * newGradient[1];
    }

    for (let x = 0; x < nodes; x += pixelAmount) {
      // determine grid cell coordinaates
      const xF = Math.floor(x);
      for (let y = 0; y < nodes; y += pixelAmount) {
        const yF = Math.floor(y);
        // dot products
        const xA = dotProd(x, y, xF, yF);
        const xB = dotProd(x, y, xF + 1, yF);
        const yA = dotProd(x, y, xF, yF + 1);
        const yB = dotProd(x, y, xF + 1, yF + 1);
        // interpolations
        const interpX1 = interpolation(x - xF, xA, xB);
        const interpX2 = interpolation(x - xF, yA, yB);
        const val = interpolation(y - yF, interpX1, interpX2);
        // draw squaare
        canvasOpts.canvasContext.fillStyle = `hsl(${
          val * (canvasOpts?.color || 250)
        }, 50%, 50%)`;
        canvasOpts.canvasContext.fillRect(
          x / nodeSteps,
          y / nodeSteps,
          pixelSize,
          pixelSize
        );
      }
    }
  }
</script>
