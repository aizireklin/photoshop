<template>
  <div class="photoshop">
    <TopPanel class="header" @show="changeUploadModal" @clear="clear(), closeDisplay(), updateCanvas()" @scale="changeScaleModal"
      @pippet="pippetClick" @grab="grabClick" @curve="curvesClick" @filtering="filterClick" @save="saveImage"
      :state="this.state" :startImage="this.startImage" ref="header" />
    <div class="wrapper">
      <BottomPanel class="sidebar" @scale="scaleImage()" :x="this.x" :state="this.state" :y="this.y" :height="this.height"
        :width="this.width" :selectedPixel="this.selectedPixel" :showData="this.showData" ref="sidebar" />
      <div class="drawing">
        <canvas width="1250" height="640" ref="drawing" @mousedown="mouseClickStart" @mouseup="mouseClickEnd" 
        @mousemove="mouseMove" @mousewheel="mouseWheel" />
      </div>
      <PippetModal @show="pippetClick" :selectedPixel="this.selectedPixel" :startX="this.startX" :startY="this.startY"
        :curWidth="this.curWidth" :curHeight="this.curHeight" ref="pippet" v-if="showPippetModal" />
      <CurvesModal @show="curvesClick" @apply="curvesClick(), updateImage()" :ctxRef="this.ctx" :leftX="this.leftX"
        :leftY="this.leftY" :curWidth="this.curWidth" :curHeight="this.curHeight" :startImage="this.startImage" :showCurvesModal="this.showCurvesModal"
        ref="curves" v-show="showCurvesModal" />
      <FilteringModal @show="filterClick" @apply="filterClick(), updateImage()" :ctxRef="this.ctx" :leftX="this.leftX"
        :leftY="this.leftY" :curWidth="this.curWidth" :curHeight="this.curHeight" :startImage="this.startImage" ref="Filtering" v-if="showFilterModal" />
    </div>
    <div style="display: none"> <canvas ref="canvasRef" /> </div>
    <UploadModal v-show="showUploadModal" @show="changeUploadModal" @download="clear(), draw(), showDisplay()" ref="modal" />
    <ScaleModal :curHeight="this.curHeight" :curWidth="this.curWidth" v-show="showScaleModal" @show="changeScaleModal" ref="scaleModal" @resize="resizeImg" />
  </div>
</template>

<script>
import TopPanel from "../components/TopPanel.vue";
import UploadModal from "../components/UploadModal.vue";
import PippetModal from "../components/PippetModal.vue";
import ScaleModal from "../components/ScaleModal.vue";
import CurvesModal from "../components/CurvesModal.vue";
import FilteringModal from "../components/FilteringModal.vue";
import BottomPanel from "../components/BottomPanel.vue";

export default {
  name: "PhotoShop",
  components: {
    BottomPanel,
    TopPanel,
    UploadModal,
    ScaleModal,
    PippetModal,
    CurvesModal,
    FilteringModal,
  },
  data() {
    return {
      startImage: null,
      canvas: null,
      ctx: null,
      canvasRef: null,
      ctxCanvasRef: null,
      selectedPixel: null,
      x: 0,
      y: 0,
      showUploadModal: false,
      showScaleModal: false,
      showPippetModal: false,
      showCurvesModal: false,
      showFilterModal: false,
      showData: false,
      width: 0,
      height: 0,
      curWidth: null,
      curHeight: null,
      leftX: null,
      leftY: null,
      isDragging: false,
      startX: null,
      startY: null,
      state: "",
      shift: false,
    };
  },
  methods: {
    updateCanvas(){
      this.$refs.modal.result = null;
      this.startImage = null;
    },
    scaleImage() {
      let scalePers = this.$refs.sidebar.scale / 100;
      if (this.height > this.canvas.height - 100 || this.width > this.canvas.width - 100) {
        let scale_factor = Math.min(
          this.canvas.width / this.width,
          this.canvas.height / this.height
        );
        let newWidth = (this.width * scale_factor - 100) * scalePers;
        let newHeight = (this.height * scale_factor - 100) * scalePers;
        this.curHeight = Math.round(newHeight);
        this.curWidth = Math.round(newWidth);
        let x = this.canvas.width / 2 - newWidth / 2;
        let y = this.canvas.height / 2 - newHeight / 2;
        this.leftX = Math.round(x);
        this.leftY = Math.round(y);
        this.clear();
        this.ctx.drawImage(this.$refs.modal.result, x, y, newWidth, newHeight);
      } else {
        let newWidth = this.width * scalePers;
        let newHeight = this.height * scalePers;
        this.curHeight = Math.round(newHeight);
        this.curWidth = Math.round(newWidth);
        let x = this.canvas.width / 2 - newHeight / 2;
        let y = this.canvas.height / 2 - newWidth / 2;
        this.leftX = Math.round(x);
        this.leftY = Math.round(y);
        this.clear();
        this.ctx.drawImage(this.$refs.modal.result, x, y, newWidth, newHeight);
      }
    },
    mouseClickStart(e) {
      this.isDragging = true;
      this.startX = e.offsetX - this.leftX;
      this.startY = e.offsetY - this.leftY;
      if (this.state == "pippet" && this.$refs.modal.result != null) {
        this.selectedPixel = this.ctx.getImageData(this.x, this.y, 1, 1).data;
      }
    },
    mouseClickEnd() {
      this.isDragging = false;
    },
    mouseMove(e) {
      this.x = e.offsetX;
      this.y = e.offsetY;
      if (this.isDragging && this.state == "grab") {
        this.leftX = e.offsetX - this.startX;
        this.leftY = e.offsetY - this.startY;
        if (this.leftX + 20 > this.canvas.width) {
          this.leftX = this.canvas.width - 20;
        }
        if (this.leftY + 20 > this.canvas.height) {
          this.leftY = this.canvas.height - 20;
        }
        if (this.leftX + this.curWidth < 20) {
          this.leftX = -this.curWidth + 20;
        }
        if (this.leftY + this.curHeight < 20) {
          this.leftY = -this.curHeight + 20;
        }
        this.moveImage();
      }
    },
    mouseWheel(event) {
      if (this.state != "pippet" && this.startImage != null && this.state != "curves" && this.state != "filter") {
        event.preventDefault();
        const delta = Math.sign(event.deltaY);
        if (this.shift) {
          this.leftX -= delta * 6;
        } else {
          this.leftY += delta * 6;
        }
        this.moveImage();
      }
    },
    grabClick() {
      if (this.state == "grab") {
        this.state = "";
      } else {
        this.state = "grab";
      }
      this.showPippetModal = false;
      this.showCurvesModal = false;
      this.showFilterModal = false;
    },
    pippetClick() {
      if (this.state == "pippet") {
        this.state = "";
      } else {
        this.state = "pippet";
      }
      this.showPippetModal = !this.showPippetModal;
      this.showCurvesModal = false;
      this.showFilterModal = false;
    },
    curvesClick() {
      if (this.state == "curves") {
        this.state = "";
      } else {
        this.state = "curves";
      }
      this.showCurvesModal = !this.showCurvesModal;
      this.showPippetModal = false;
      this.showFilterModal = false;
    },
    filterClick() {
      if (this.state == "filter") {
        this.state = "";
      } else {
        this.state = "filter";
      }
      this.showPippetModal = false;
      this.showCurvesModal = false;
      this.showFilterModal = !this.showFilterModal;
    },
    pressShift(event) {
      if (event.key === "Shift") {
        this.shift = true;
      }
    },
    unpressShift(event) {
      if (event.key === "Shift") {
        this.shift = false;
      }
    },
    moveImage() {
      this.clear();
      // console.log("1111", this.leftX, this.leftY);
      this.ctx.drawImage(this.$refs.modal.result, this.leftX, this.leftY, this.curWidth, this.curHeight);
    },
    draw() {
      this.startImage = this.$refs.modal.result;
      this.height = this.$refs.modal.result.height;
      this.width = this.$refs.modal.result.width;
      if (this.height > this.canvas.height - 100 || this.width > this.canvas.width - 100) {
        let scaleFactor = Math.min(this.canvas.width / this.width, this.canvas.height / this.height);
        let newWidth = this.width * scaleFactor - 100;
        let newHeight = this.height * scaleFactor - 100;
        this.curHeight = Math.round(newHeight);
        this.curWidth = Math.round(newWidth);
        let x = this.canvas.width / 2 - newWidth / 2;
        let y = this.canvas.height / 2 - newHeight / 2;
        // console.Console;og("!!!", x, y, this.curWidth, this.curHeight);
        this.leftX = Math.round(x);
        this.leftY = Math.round(y);
        this.ctx.drawImage(this.$refs.modal.result, x, y, newWidth, newHeight);
      } else {
        let x = this.canvas.width / 2 - this.height / 2;
        let y = this.canvas.height / 2 - this.width / 2;
        this.leftX = Math.round(x);
        this.leftY = Math.round(y);
        this.curHeight = Math.round(this.height);
        this.curWidth = Math.round(this.width);
        // console.Console;og("!!!", x, y, this.curWidth, this.curHeight);
        this.ctx.drawImage(this.$refs.modal.result, x, y, this.width, this.height);
      }
    },
    changeUploadModal() {
      this.showUploadModal = !this.showUploadModal;
    },
    changeScaleModal() {
      this.showScaleModal = !this.showScaleModal;
    },
    updateImage() {
      let newImage = this.ctx.getImageData(this.leftX, this.leftY, this.curWidth, this.curHeight);
      this.canvasRef.width = this.curWidth;
      this.canvasRef.height = this.curHeight;
      this.ctxCanvasRef.putImageData(newImage, 0, 0);
      let image = new Image();
      image.src = this.canvasRef.toDataURL();
      this.$refs.modal.result = image;
      this.startImage = image;
    },
    showDisplay() {
      this.showData = true;
    },
    closeDisplay() {
      this.showData = false;
    },
    clear() {
      this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
      this.selectedPixel = null;
    },
    saveImage() {
      this.canvasRef.width = this.width;
      this.canvasRef.height = this.height;
      this.ctxCanvasRef.drawImage(this.$refs.modal.result, 0, 0, this.width, this.height );
      const imageDataURL = this.canvasRef.toDataURL("image/png");
      const link = document.createElement("a");
      link.href = imageDataURL;
      link.download = "my_image.png";
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },
    resizeImg() {
      let dx = this.canvas.width / 2 - this.curWidth / 2;
      let dy = this.canvas.height / 2 - this.curHeight / 2;
      const img = this.ctx.getImageData(dx, dy, this.curWidth, this.curHeight);
      const originalWidth = this.curWidth;
      const originalHeight = this.curHeight;
      const newHeight = this.$refs.scaleModal.outputH;
      const newWidth = this.$refs.scaleModal.outputW;
      const scaleX = originalWidth / newWidth;
      const scaleY = originalHeight / newHeight;
      const newData = new Uint8ClampedArray(newWidth * newHeight * 4);

      for (let y = 0; y < newHeight; y++) {
        for (let x = 0; x < newWidth; x++) {
          const px = Math.floor(x * scaleX);
          const py = Math.floor(y * scaleY);
          const index = (y * newWidth + x) * 4;
          const originalIndex = (py * originalWidth + px) * 4;

          newData[index] = img.data[originalIndex];
          newData[index + 1] = img.data[originalIndex + 1];
          newData[index + 2] = img.data[originalIndex + 2];
          newData[index + 3] = img.data[originalIndex + 3];
        }
      }
      let nx = this.canvas.width / 2 - newWidth / 2;
      let ny = this.canvas.height / 2 - newHeight / 2;
      let image = new ImageData(newData, newWidth, newHeight);
      this.clear();
      this.ctx.putImageData(image, nx, ny);
      this.height = newHeight;
      this.width = newWidth;
      this.curHeight = newHeight;
      this.curWidth = newWidth;
      this.leftX = Math.round(nx);
      this.leftY = Math.round(ny);
    },
  },
  mounted() {
    this.canvas = this.$refs["drawing"];
    this.ctx = this.canvas.getContext("2d", { willReadFrequently: true });
    this.canvasRef = this.$refs["canvasRef"];
    this.ctxCanvasRef = this.canvasRef.getContext("2d", {
      willReadFrequently: true,
    });
    window.addEventListener("keydown", this.pressShift);
    window.addEventListener("keyup", this.unpressShift);
  },
};
</script>

<style lang="scss" scoped>
.photoshop {
  height: 100%;
}
.wrapper {
  position: relative;
  display: flex;
  flex-direction: row;
  height: calc(100% - 80px);
}
.source {
  display: none;
}

.drawing {
  height: 642px;
  margin: 0 auto;
  // background-image: url("@/assets/1674303626_catherineasquithgallery-com-p-fon-serii-kvadratiki-foto-22.jpg");
  background-size: 20%;
  background-repeat: repeat;
  border: 1px solid #e0e1dd;
}
</style>
