<template>
  <div class="form">
    <div class="postcard">
      <div
        ref="viewport"
        :class="`postcard-viewport ${isPortrait ? '' : 'postcard-viewport-landscape'}`"
      >
        <canvas
          ref="canvas"
          class="canvas"
          :width="isPortrait ? 350 : 500"
          :height="isPortrait ? 500 : 350"
        ></canvas>
        <TextBlock
          ref="textBlock"
          v-if="isTextInputVisible"
          class="text-block"
          v-model="drawOptions.text"
          @blur="addTextToCanvas"
        />
      </div>
    </div>
    <div class="divider"></div>
    <div class="controls">
      <Button class="button" @click="uploadImage">Upload Image</Button>
      <Button
        class="button"
        @click="showTextInput"
        :disabled="!isImageLoaded"
      >New text block</Button>
      <Button
        class="button"
        @click="zoomIn"
        :disabled="!isImageLoaded"
      >Zoom in</Button>
      <Button
        class="button"
        @click="zoomOut"
        :disabled="!isImageLoaded"
      >Zoom out</Button>
      <Button
        class="button"
        @click="rotate"
        :disabled="!isImageLoaded"
      >Rotate</Button>
      <Button
        class="button"
        @click="undo"
        :disabled="!isImageLoaded || undoQueue.length === 0"
      >Undo</Button>
      <Button
        class="button"
        @click="redo"
        :disabled="!isImageLoaded || redoQueue.length === 0"
      >Redo</Button>
      <Button
        class="button green"
        @click="save"
        :disabled="!isImageLoaded"
      >Save</Button>
    </div>
  </div>
</template>

<script>
import Button from './Button.vue';
import TextBlock from './TextBlock.vue';

const PORTRAIT = 'portrait';
const LANDSCAPE = 'landscape';

const drawOptionsInitialState = {
  orientation: PORTRAIT,
  scale: 1,
  image: new Image(),
  text: '',
  textCoords: { x: 0, y: 0 },
  textSize: 24,
  textFont: 'Arial, Helvetica, sans-serif',
};

export default {
  components: {
    Button,
    TextBlock,
  },
  data() {
    return {
      isTextInputVisible: false,
      isImageLoaded: false,
      drawOptions: {
        ...drawOptionsInitialState,
      },
      undoQueue: [],
      redoQueue: [],
      isUndoLogPaused: false,
    };
  },
  computed: {
    isPortrait() {
      return this.drawOptions.orientation === PORTRAIT;
    },
  },
  watch: {
    drawOptions: {
      handler(options) {
        if (!this.isUndoLogPaused) {
          this.undoQueue.push({ ...options });
        }
      },
      deep: true,
    },
  },
  mounted() {
    this.ctx = this.$refs.canvas.getContext('2d');
    this.fileInput = document.createElement('input');
    this.fileInput.setAttribute('type', 'file');
    this.bindDOMEvents();
  },
  beforeDestroy() {
    this.unbindDOMEvents();
  },
  methods: {
    bindDOMEvents() {
      this.fileInput.addEventListener('change', this.onFileInputChange);
      this.drawOptions.image.addEventListener('load', this.onImageLoad);
    },
    unbindDOMEvents() {
      this.fileInput.removeEventListener('change', this.onFileInputChange);
      this.drawOptions.image.removeEventListener('load', this.onImageLoad);
    },
    uploadImage() {
      this.fileInput.click();
    },
    onFileInputChange() {
      if (this.fileInput.files.length > 0) {
        const reader = new FileReader();
        reader.addEventListener('load', this.onFileLoad);
        reader.readAsDataURL(this.fileInput.files[0]);
      }
    },
    onFileLoad(event) {
      this.drawOptions.image.src = event.target.result;
    },
    async onImageLoad() {
      this.isImageLoaded = true;

      this.undoQueue = [];
      this.redoQueue = [];

      this.isUndoLogPaused = true;
      this.drawOptions = { ...drawOptionsInitialState };
      await this.$nextTick();
      this.isUndoLogPaused = false;

      this.drawCanvas(this.drawOptions);
    },
    drawCanvas({
      orientation,
      scale,
      image,
      text,
      textCoords,
      textSize,
      textFont,
    } = {}) {
      const w = this.$refs.canvas.width;
      const h = this.$refs.canvas.height;
      const imageRatio = image.width / image.height;

      // reset before every paint
      this.ctx.clearRect(0, 0, w, h);

      switch (orientation) {
        case PORTRAIT: {
          let imageXOffset = 0;

          if (image.width > image.height) {
            imageXOffset = w / 2;
          }

          this.ctx.drawImage(image, -imageXOffset, 0, (h * imageRatio) * scale, h * scale);

          if (text) {
            const fontSize = textSize * scale;
            const padding = 8;
            this.ctx.font = `${fontSize}px ${textFont}`;
            this.ctx.fillStyle = 'white';
            this.ctx.fillText(
              text,
              (textCoords.x + padding),
              (textCoords.y + padding + fontSize),
            );

            // hide text input after text is added to canvas
            this.isTextInputVisible = false;
          }

          break;
        }
        case LANDSCAPE: {
          let imageXOffset = 0;

          if (image.width > image.height) {
            imageXOffset = h / 2;
          }

          this.ctx.translate(w / 2, h / 2);
          this.ctx.rotate(Math.PI / 2);
          this.ctx.drawImage(
            image,
            -(h / 2) - imageXOffset,
            -(w / 2),
            (w * imageRatio) * scale,
            w * scale,
          );
          this.ctx.rotate(-Math.PI / 2);
          this.ctx.translate(-w / 2, -h / 2);

          if (text) {
            const fontSize = textSize * scale;
            const padding = 8;
            this.ctx.font = `${fontSize}px ${textFont}`;
            this.ctx.fillStyle = 'white';
            this.ctx.fillText(
              text,
              // (textCoords.x + padding),
              // (textCoords.y + padding + fontSize),
              (w / 2) - (this.ctx.measureText(text).width / 2),
              textCoords.y + padding + fontSize,
            );

            // hide text input after text is added to canvas
            this.isTextInputVisible = false;
          }

          break;
        }
        // no default
      }
    },
    async showTextInput() {
      this.isTextInputVisible = true;
      await this.$nextTick();
      this.$refs.textBlock.focus();
    },
    addTextToCanvas() {
      if (this.drawOptions.text) {
        const viewportRect = this.$refs.viewport.getBoundingClientRect();
        const textBlockRect = this.$refs.textBlock.$el.getBoundingClientRect();

        this.drawOptions.textCoords = {
          x: textBlockRect.x - viewportRect.x,
          y: textBlockRect.y - viewportRect.y,
        };
      } else {
        this.drawOptions = {
          ...this.drawOptions,
          text: '',
          textCoords: { x: 0, y: 0 },
          textSize: 24,
          textFont: 'Arial, Helvetica, sans-serif',
        };
      }

      this.drawCanvas(this.drawOptions);
    },
    async undo() {
      this.isUndoLogPaused = true;

      const [lastUndo] = this.undoQueue.slice(-1);
      this.undoQueue = this.undoQueue.slice(0, this.undoQueue.length - 1);
      this.redoQueue.push({ ...lastUndo });

      const [optionsToDraw] = this.undoQueue.slice(-1);

      if (optionsToDraw) {
        this.drawOptions = { ...optionsToDraw };
      } else {
        this.drawOptions = { ...drawOptionsInitialState };
      }

      await this.$nextTick();
      this.drawCanvas(this.drawOptions);
      await this.$nextTick();

      this.isUndoLogPaused = false;
    },
    async redo() {
      const [lastRedo] = this.redoQueue.slice(-1);
      this.redoQueue = this.redoQueue.slice(0, this.redoQueue.length - 1);

      if (lastRedo) {
        this.drawOptions = { ...lastRedo };
        await this.$nextTick();
        this.drawCanvas(this.drawOptions);
        await this.$nextTick();
      }
    },
    zoomIn() {
      this.drawOptions.scale += 0.1;
      this.drawCanvas(this.drawOptions);
    },
    zoomOut() {
      this.drawOptions.scale -= 0.1;
      this.drawCanvas(this.drawOptions);
    },
    async rotate() {
      const { orientation } = this.drawOptions;
      this.drawOptions.orientation = orientation === PORTRAIT ? LANDSCAPE : PORTRAIT;
      await this.$nextTick();
      this.drawCanvas(this.drawOptions);
    },
    save() {
      const anchor = document.createElement('a');
      anchor.setAttribute('download', 'MyNewPostcard.png');
      anchor.setAttribute(
        'href',
        this.$refs.canvas.toDataURL('image/png').replace('image/png', 'image/octet-stream'),
      );
      anchor.click();
    },
  },
};
</script>

<style scoped>
  .form {
    display: flex;
    justify-content: space-between;
    width: 80%;
    height: 70%;
    background-color: white;
  }

  .postcard {
    width: 70%;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .postcard-viewport {
    position: relative;
    background-color: lightgray;
    width: 350px;
    height: 500px;
    overflow: hidden;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .postcard-viewport-landscape {
    height: 350px;
    width: 500px;
  }

  .postcard-image {
    height: 100%;
    width: auto;
    transition: height 150ms;
  }

  .canvas {
    height: 100%;
    width: 100%;
  }

  .text-block {
    position: absolute;
    top: 50px;
    width: auto;
  }

  .divider {
    width: 2px;
    height: 100%;
    margin: 0 24px;
    background-color: lightgray;
  }

  .controls {
    width: 30%;
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .button:not(:last-child) {
    margin-bottom: 24px
  }
</style>
