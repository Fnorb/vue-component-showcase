<template>
  <div class="animated-background">
    <canvas ref="animatedCanvas"></canvas>
  </div>
</template>

<script>
export default {
  name: 'AnimatedBackground',
  props: {
    trackingContainer: {
      type: String,
      default: ".animated-background"
    }
  },
  data() {
    return {
      canvas: null,
      ctx: null,
      listenersSet: false,
      tileMinSizePX: 30,
      tileMaxSizePX: 120,
      effectMax: 2,
      effectRange: 150,
      columnWidths: [],
      rowHeights: [],
      rectangleWidth: 200,  
      rectangleHeight: 10, 
      rectangleColor: 171,
      rectangleSaturation: 100,
      rectangleLightness: [],
      rectangleLightnessStart: 12,
      rectangleLightnessEnd: 15,
      rectangleLightnessStep: 0.1,
      rectangleLightnessSet: false,
      needsUpdate: false, // Flag für Berechnungen
      backgroundFade: "sinusoidal", //linear return random sinusoidal
      backgroundPattern: "random", //random line scattered
      rectanglePhases: [],
    };
  },
  mounted() {
    this.$nextTick(() => {
      const canvas = this.$refs.animatedCanvas;
      const context = canvas.getContext("2d", { willReadFrequently: false });
      this.canvas = canvas;
      this.ctx = context;

      if (!this.listenersSet) {
        let resizeTimer;
        window.addEventListener('resize', () => {
          clearTimeout(resizeTimer);
          resizeTimer = setTimeout(() => {
            this.$nextTick(() => {
              this.upgradeAnimatedBGDimensions();
              this.placeTiles();
            });
          }, 200);
        });
        this.listenersSet = true;
      }

      this.upgradeAnimatedBGDimensions();
      this.placeTiles();
      this.initiaLizeLightness();
      document.querySelector('.app-container').addEventListener('mousemove', this.handleMouseMove);

      this.animate(); // Startet die Animationsschleife
    });
  },
  methods: {
    initiaLizeLightness() {
      // Placeholder für zukünftige Implementierung
      const rows = this.rowHeights.length;
      const cols = this.columnWidths.length;

      this.rectanglePhases = Array.from({ length: rows }, () =>
        Array.from({ length: cols }, () => Math.random() * Math.PI * 2) // Random phase
      );
    },

    upgradeAnimatedBGDimensions() {
      this.canvas.width = 0;
      this.canvas.height = 0;
      const container = this.canvas.parentElement.parentElement;
      this.canvas.width = container.offsetWidth;
      this.canvas.height = container.offsetHeight;
    },

    placeTiles() {
  const canvasWidth = this.canvas.width;
  const canvasHeight = this.canvas.height;
  const tileBaseWidth = this.rectangleWidth;
  const tileBaseHeight = this.rectangleHeight;
  const columns = Math.ceil(canvasWidth / tileBaseWidth);
  const rows = Math.ceil(canvasHeight / tileBaseHeight);

  if (this.columnWidths.length === 0) this.columnWidths = new Array(columns).fill(1);
  if (this.rowHeights.length === 0) this.rowHeights = new Array(rows).fill(1);

  // Ensure columnWidths matches columns
  while (this.columnWidths.length < columns) this.columnWidths.push(1);
  while (this.columnWidths.length > columns) this.columnWidths.pop();

  // Ensure rowHeights matches rows
  while (this.rowHeights.length < rows) this.rowHeights.push(1);
  while (this.rowHeights.length > rows) this.rowHeights.pop();

  while (this.rectangleLightness.length < rows) {
    this.rectangleLightness.push([]);
  }
  while (this.rectangleLightness.length > rows) {
    this.rectangleLightness.pop();
  }

  if (this.backgroundFade === "sinusoidal" && this.rectanglePhases.length === 0) {
    this.rectanglePhases = Array.from({ length: rows }, () =>
      Array.from({ length: columns }, () => Math.random() * Math.PI * 2) // Random phase
    );
  }

  const time = performance.now() / 1000; // Time in seconds
  const frequency = this.rectangleLightnessStep * 10; 

  
  for (let row = 0; row < rows; row++) {
    while (this.rectangleLightness.length < rows) {
      this.rectangleLightness.push(new Array(columns).fill(null));
    }
    while (this.rectangleLightness.length > rows) {
      this.rectangleLightness.pop();
    }

    for (let col = 0; col < columns; col++) {
      if (this.rectangleLightness[row] === undefined) {
        this.rectangleLightness[row] = [];
      }

      if (this.backgroundFade === "sinusoidal") {
        const phase = this.rectanglePhases[row][col];
        const sinValue = Math.sin(time * frequency + phase);
        this.rectangleLightness[row][col] =
          this.rectangleLightnessStart +
          (this.rectangleLightnessEnd - this.rectangleLightnessStart) * ((sinValue + 1) / 2);
      } else {
        if (this.rectangleLightness[row][col] === undefined) {
          this.rectangleLightness[row][col] =
            this.rectangleLightnessEnd + Math.random() * (this.rectangleLightnessStart - this.rectangleLightnessEnd);
        } else {
          this.rectangleLightness[row][col] += this.rectangleLightnessStep;

          if (this.backgroundFade === "reverse") {
            if (
              this.rectangleLightness[row][col] >= this.rectangleLightnessEnd ||
              this.rectangleLightness[row][col] <= this.rectangleLightnessStart
            ) {
              this.rectangleLightnessStep *= -1; // Reverse direction
            }
          } else {
            if (this.rectangleLightness[row][col] > this.rectangleLightnessEnd) {
              this.rectangleLightness[row][col] = this.rectangleLightnessStart;
            }
          }
        }
      }
    }

    while (this.rectangleLightness[row].length > columns) {
      this.rectangleLightness[row].pop();
    }
  }

  this.ctx.clearRect(0, 0, canvasWidth, canvasHeight);

  let yPos = 0;
  for (let row = 0; row < rows; row++) {
    let xPos = 0;
    for (let col = 0; col < columns; col++) {
      const scaledWidth = tileBaseWidth * this.columnWidths[col];
      const scaledHeight = tileBaseHeight * this.rowHeights[row];
      const lightness = this.rectangleLightness[row][col];
      this.ctx.fillStyle = `hsl(${this.rectangleColor} ${this.rectangleSaturation}% ${lightness}%)`;
      this.ctx.fillRect(Math.ceil(xPos), Math.ceil(yPos), Math.ceil(scaledWidth), Math.ceil(scaledHeight));
      xPos += scaledWidth;
    }
    yPos += tileBaseHeight * this.rowHeights[row];
  }
},


    handleMouseMove(event) {
      const mouseX = event.offsetX;
      const mouseY = event.offsetY;
      const numColumns = this.columnWidths.length;
      const numRows = this.rowHeights.length;
      const columnWidth = this.canvas.width / numColumns;
      const rowHeight = this.canvas.height / numRows;

      let closestCol = -1;
      let closestRow = -1;
      let closestColDist = Number.POSITIVE_INFINITY;
      let closestRowDist = Number.POSITIVE_INFINITY;

      for (let col = 0; col < numColumns; col++) {
        const colCenter = col * columnWidth + columnWidth / 2;
        const dist = Math.abs(mouseX - colCenter);
        if (dist < closestColDist) {
          closestCol = col;
          closestColDist = dist;
        }
      }

      for (let row = 0; row < numRows; row++) {
        const rowCenter = row * rowHeight + rowHeight / 2;
        const dist = Math.abs(mouseY - rowCenter);
        if (dist < closestRowDist) {
          closestRow = row;
          closestRowDist = dist;
        }
      }

      const positionInCol = ((mouseX - (closestCol * columnWidth + columnWidth / 2)) / (columnWidth / 2)) / 2;
      const positionInRow = ((mouseY - (closestRow * rowHeight + rowHeight / 2)) / (rowHeight / 2)) / 2;

      this.applyWeighting(closestCol, closestRow, positionInCol, positionInRow);
      this.needsUpdate = true; // Setzt das Update-Flag
    },

    applyWeighting(colId, rowId, weightX, weightY) {
      let colWeights = new Array(this.columnWidths.length).fill(1);
      let rowWeights = new Array(this.rowHeights.length).fill(1);
      const effectRel = this.effectMax - 1;
      const absWeightX = Math.abs(weightX);
      const absWeightY = Math.abs(weightY);

      colWeights[colId] = 1 + effectRel * (1 - absWeightX);
      rowWeights[rowId] = 1 + effectRel * (1 - absWeightY);

      if (weightX < 0 && colId > 0) colWeights[colId - 1] = 1 + (absWeightX * effectRel);
      if (weightX > 0 && colId < colWeights.length - 1) colWeights[colId + 1] = 1 + (absWeightX * effectRel);

      if (weightY < 0 && rowId > 0) rowWeights[rowId - 1] = 1 + (absWeightY * effectRel);
      if (weightY > 0 && rowId < rowWeights.length - 1) rowWeights[rowId + 1] = 1 + (absWeightY * effectRel);

      this.columnWidths = colWeights;
      this.rowHeights = rowWeights;
    },

    animate() {
      requestAnimationFrame(this.animate);
      this.placeTiles();
    },
  },
};
</script>

<style scoped>
.animated-background {
  position: absolute;
  width: 100%;
  height: 100%;
  opacity: 1;
}

.animated-background canvas {
    image-rendering: pixelated;
    image-rendering: crisp-edges;
}

</style>
