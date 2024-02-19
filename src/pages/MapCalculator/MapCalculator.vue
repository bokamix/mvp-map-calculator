<script>
import VueDrawingCanvas from "vue-drawing-canvas";

export default {
  name: "ServeDev",
  components: {
    VueDrawingCanvas,
  },
  data() {
    return {
      lineTypes: [
        {
          name: "old line",
        },
        {
          name: "new line",
        },
        {
          name: "scale",
        },
      ],
      initialImage: [
        {
          type: "dash",
          from: {
            x: 262,
            y: 154,
          },
          coordinates: [],
          color: "#000000",
          width: 5,
          fill: false,
        },
      ],
      x: 0,
      y: 0,
      image: "",
      eraser: false,
      disabled: false,
      fillShape: false,
      line: 5,
      height: 600,
      width: 800,
      maxWidth: 1200,
      color: "#000000",
      strokeType: "line",
      lineCap: "square",
      lineJoin: "miter",
      backgroundColor: "#FFFFFF",
      backgroundImage: null,
      watermark: null,
      additionalImages: [],
      distanceLines: [],
    };
  },
  mounted() {
    if ("vue-drawing-canvas" in window.localStorage) {
      this.initialImage = JSON.parse(
        window.localStorage.getItem("vue-drawing-canvas")
      );
    }
  },
  methods: {
    getAllObjects() {
      function calculateDistance(point1, point2) {
        var xDiff = point2.x - point1.x;
        var yDiff = point2.y - point1.y;
        return Math.sqrt(xDiff * xDiff + yDiff * yDiff);
      }

      const e = this.$refs.VueCanvasDrawing.getAllStrokes().map((e) => {
        console.log("e", e);
        return {
          start: {
            x: e?.from?.x,
            y: e?.from?.y,
            color: e?.color,
          },
          end: {
            x: e?.coordinates[0]?.x,
            y: e?.coordinates[0]?.y,
            color: e?.color,
          },
        };
      });
      const filtered = e.filter((e) => {
        console.log("asdasd", e);
        return e.start.x && e.start.y && e.end.x && e.end.y;
      });
      this.distanceLines = filtered.map((e) => {
        return {
          ...e,
          color: e.start.color,
          distance: calculateDistance(e.start, e.end),
        };
      });
      console.log(this.distanceLines);
    },
    async setImage(event) {
      console.log(event);
      let URL = window.URL;
      this.backgroundImage = URL.createObjectURL(event.target.files[0]);
      let img = new Image();
      img.src = this.backgroundImage;
      img.onload = () => {
        const scaleImage = img.width / img.height;
        this.width = this.maxWidth;
        this.height = this.width / scaleImage;
      };
      await this.$refs.VueCanvasDrawing.redraw();
    },
    async setWatermarkImage(event) {
      let URL = window.URL;
      this.watermark = {
        type: "Image",
        source: URL.createObjectURL(event.target.files[0]),
        x: 0,
        y: 0,
        imageStyle: {
          width: 600,
          height: 400,
        },
      };
      await this.$refs.VueCanvasDrawing.redraw();
    },
    getCoordinate(event) {
      let coordinates = this.$refs.VueCanvasDrawing.getCoordinates(event);
      this.x = coordinates.x;
      this.y = coordinates.y;
    },
    getStrokes() {
      window.localStorage.setItem(
        "vue-drawing-canvas",
        JSON.stringify(this.$refs.VueCanvasDrawing.getAllStrokes())
      );
      alert(
        "Strokes saved, reload your browser to see the canvas with previously saved image"
      );
    },
    removeSavedStrokes() {
      window.localStorage.removeItem("vue-drawing-canvas");
      alert("Strokes cleared from local storage");
    },
  },
};
</script>

<template>
  <div id="app">
    <div class="flex-row">
      <div class="source">
        <h2>Wybierz obraz</h2>
        <div>
          <p style="margin-bottom: 0">Upload Background Image:</p>
          <input type="file" @change="setImage($event)" />
        </div>
        <vue-drawing-canvas
          ref="VueCanvasDrawing"
          :height="height"
          :width="width"
          v-model:image="image"
          :stroke-type="strokeType"
          :line-cap="lineCap"
          :line-join="lineJoin"
          :fill-shape="fillShape"
          :eraser="eraser"
          :lineWidth="line"
          :color="color"
          :background-image="backgroundImage"
          :watermark="watermark"
          :initial-image="initialImage"
          saveAs="png"
          :styles="{
            border: 'solid 1px #000',
          }"
          :lock="disabled"
          :additional-images="additionalImages"
        />
        <div class="button-container">
          <button @click="getAllObjects">Przelicz odległości</button>
          <button type="button" @click.prevent="disabled = !disabled">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="16"
              height="16"
              fill="currentColor"
              class="bi bi-lock"
              viewBox="0 0 16 16"
            >
              <path
                v-if="!disabled"
                d="M11 1a2 2 0 0 0-2 2v4a2 2 0 0 1 2 2v5a2 2 0 0 1-2 2H3a2 2 0 0 1-2-2V9a2 2 0 0 1 2-2h5V3a3 3 0 0 1 6 0v4a.5.5 0 0 1-1 0V3a2 2 0 0 0-2-2zM3 8a1 1 0 0 0-1 1v5a1 1 0 0 0 1 1h6a1 1 0 0 0 1-1V9a1 1 0 0 0-1-1H3z"
              />
              <path
                v-else
                d="M8 1a2 2 0 0 1 2 2v4H6V3a2 2 0 0 1 2-2zm3 6V3a3 3 0 0 0-6 0v4a2 2 0 0 0-2 2v5a2 2 0 0 0 2 2h6a2 2 0 0 0 2-2V9a2 2 0 0 0-2-2zM5 8h6a1 1 0 0 1 1 1v5a1 1 0 0 1-1 1H5a1 1 0 0 1-1-1V9a1 1 0 0 1 1-1z"
              />
            </svg>
            <span v-if="!disabled">Unlock</span>
            <span v-else>Lock</span>
          </button>
          <button type="button" @click.prevent="$refs.VueCanvasDrawing.undo()">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="16"
              height="16"
              fill="currentColor"
              class="bi bi-arrow-counterclockwise"
              viewBox="0 0 16 16"
            >
              <path
                fill-rule="evenodd"
                d="M8 3a5 5 0 1 1-4.546 2.914.5.5 0 0 0-.908-.417A6 6 0 1 0 8 2v1z"
              />
              <path
                d="M8 4.466V.534a.25.25 0 0 0-.41-.192L5.23 2.308a.25.25 0 0 0 0 .384l2.36 1.966A.25.25 0 0 0 8 4.466z"
              />
            </svg>
            Undo
          </button>
          <button type="button" @click.prevent="$refs.VueCanvasDrawing.redo()">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="16"
              height="16"
              fill="currentColor"
              class="bi bi-arrow-clockwise"
              viewBox="0 0 16 16"
            >
              <path
                fill-rule="evenodd"
                d="M8 3a5 5 0 1 0 4.546 2.914.5.5 0 0 1 .908-.417A6 6 0 1 1 8 2v1z"
              />
              <path
                d="M8 4.466V.534a.25.25 0 0 1 .41-.192l2.36 1.966c.12.1.12.284 0 .384L8.41 4.658A.25.25 0 0 1 8 4.466z"
              />
            </svg>
            Redo
          </button>
          <button type="button" @click.prevent="$refs.VueCanvasDrawing.reset()">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="16"
              height="16"
              fill="currentColor"
              class="bi bi-file-earmark"
              viewBox="0 0 16 16"
            >
              <path
                d="M14 4.5V14a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V2a2 2 0 0 1 2-2h5.5L14 4.5zm-3 0A1.5 1.5 0 0 1 9.5 3V1H4a1 1 0 0 0-1 1v12a1 1 0 0 0 1 1h8a1 1 0 0 0 1-1V4.5h-2z"
              />
            </svg>
            Reset
          </button>
        </div>
        <div class="button-container">
          <input type="color" v-model="color" />
        </div>
        <div class="button-container">
          <button type="button" @click.prevent="getStrokes()">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="16"
              height="16"
              fill="currentColor"
              class="bi bi-image-alt"
              viewBox="0 0 16 16"
            >
              <path
                d="M7 2.5a2.5 2.5 0 1 1-5 0 2.5 2.5 0 0 1 5 0zm4.225 4.053a.5.5 0 0 0-.577.093l-3.71 4.71-2.66-2.772a.5.5 0 0 0-.63.062L.002 13v2a1 1 0 0 0 1 1h14a1 1 0 0 0 1-1v-4.5l-4.777-3.947z"
              />
            </svg>
            Save All Strokes
          </button>
          <button type="button" @click.prevent="removeSavedStrokes()">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="16"
              height="16"
              fill="currentColor"
              class="bi bi-eraser"
              viewBox="0 0 16 16"
            >
              <path
                d="M8.086 2.207a2 2 0 0 1 2.828 0l3.879 3.879a2 2 0 0 1 0 2.828l-5.5 5.5A2 2 0 0 1 7.879 15H5.12a2 2 0 0 1-1.414-.586l-2.5-2.5a2 2 0 0 1 0-2.828l6.879-6.879zm2.121.707a1 1 0 0 0-1.414 0L4.16 7.547l5.293 5.293 4.633-4.633a1 1 0 0 0 0-1.414l-3.879-3.879zM8.746 13.547 3.453 8.254 1.914 9.793a1 1 0 0 0 0 1.414l2.5 2.5a1 1 0 0 0 .707.293H7.88a1 1 0 0 0 .707-.293l.16-.16z"
              />
            </svg>
            Remove Saved Strokes
          </button>
        </div>
        <div v-for="(line, index) in distanceLines" :key="index">
          Distance {{ line.distance }} - {{ line.color }}
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Roboto:ital,wght@1,700&display=swap");
body {
  font-family: "Roboto", sans-serif;
}
.flex-row {
  display: flex;
  flex-direction: row;
}
.button-container {
  display: flex;
  flex-direction: row;
}
.button-container > * {
  margin-top: 15px;
  margin-right: 10px;
}
</style>
