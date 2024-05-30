<template>
  <div class="modal">
    <div class="modal__container header">
      <p class="header__text">Определение цвета пикселя</p>
      <button class="header__button" @click="$emit('show')">X</button>
    </div>
    <div class="modal__line">
      <div class="modal__element">
        <input class="color-button" type="radio" :style="color1" />
      </div>
      <div class="modal__element">
        <input class="color-button" type="radio" :style="color2" />
      </div>
    </div>
    <div class="modal__line">
      <p>X</p>
    </div>
    <div class="modal__line">
      <div class="modal__element">
        <p>{{ X1 }}</p>
      </div>
      <div class="modal__element">
        <p>{{ X2 }}</p>
      </div>
    </div>
    <div class="modal__line">
      <p>Y</p>
    </div>
    <div class="modal__line">
      <div class="modal__element">
        <p>{{ Y1 }}</p>
      </div>
      <div class="modal__element">
        <p>{{ Y2 }}</p>
      </div>
    </div>
    <div class="modal__line">
      <p>XYZ</p>
    </div>
    <div class="modal__line">
      <div class="modal__element">
        <p>{{ XYZ1 }}</p>
      </div>
      <div class="modal__element">
        <p>{{ XYZ2 }}</p>
      </div>
    </div>
    <div class="modal__line">
      <p>RGB</p>
    </div>
    <div class="modal__line">
      <div class="modal__element">
        <p>{{ RGB1 }}</p>
      </div>
      <div class="modal__element">
        <p>{{ RGB2 }}</p>
      </div>
    </div>
    <div class="modal__line">
      <p>Lab</p>
    </div>
    <div class="modal__line">
      <div class="modal__element">
        <p>{{ Lab1 }}</p>
      </div>
      <div class="modal__element">
        <p>{{ Lab2 }}</p>
      </div>
    </div>
    <div class="modal__line" v-if="contrastRatio">
      <p :style="{ color: contrastRatio < 4.5 ? 'red' : 'black' }">Contrast {{ contrastRatio }}:1</p>
    </div>
  </div>
</template>
<script>
export default {
  name: "PippetModal",
  props: {
    selectedPixel: Array,
    startX: Number,
    startY: Number,
    curWidth: Number,
    curHeight: Number,
  },
  data() {
    return {
      rgb: [],
      XYZ: [],
      color1: "",
      color2: "",
      isAlt: false,
      X1: 0,
      Y1: 0,
      X2: 0,
      Y2: 0,
      RGB1: 0,
      rgb1: [],
      RGB2: 0,
      rgb2: [],
      XYZ1: 0,
      XYZ2: 0,
      Lab1: 0,
      Lab2: 0,
    };
  },
  watch: {
    RGB: function (value) {
      if (this.isAlt) {
        this.color2 = `background: rgb(${value.string});`;
        this.RGB2 = value.string;
        this.rgb2 = value.array;
        this.XYZ2 = this.rgbToXyz(value.array);
        this.Lab2 = this.xyzToLab(this.XYZ);
      } else {
        this.color1 = `background: rgb(${value.string});`;
        this.RGB1 = value.string;
        this.rgb1 = value.array;
        this.XYZ1 = this.rgbToXyz(value.array);
        this.Lab1 = this.xyzToLab(this.XYZ);
      }
    },
    coords: function (value) {
      if (this.isAlt) {
        this.X2 = value.X;
        this.Y2 = value.Y;
      } else {
        this.X1 = value.X;
        this.Y1 = value.Y;
      }
    },
  },
  methods: {
    pressAlt(event) {
      if (event.key === "Alt") {
        this.isAlt = true;
      }
    },
    unpressAlt(event) {
      if (event.key === "Alt") {
        this.isAlt = false;
      }
    },
    rgbToXyz(rgb) {
      let r = rgb[0];
      let g = rgb[1];
      let b = rgb[2];
      let r1 = r / 255;
      let g1 = g / 255;
      let b1 = b / 255;

      // Применяем коррекцию гамма-компрессии sRGB
      r1 = r1 > 0.04045 ? Math.pow((r1 + 0.055) / 1.055, 2.4) : r1 / 12.92;
      g1 = g1 > 0.04045 ? Math.pow((g1 + 0.055) / 1.055, 2.4) : g1 / 12.92;
      b1 = b1 > 0.04045 ? Math.pow((b1 + 0.055) / 1.055, 2.4) : b1 / 12.92;

      r1 = r1 * 100;
      g1 = g1 * 100;
      b1 = b1 * 100;

      // Коэффициенты преобразования RGB в XYZ
      let x = Math.round((r1 * 0.4124564 + g1 * 0.3575761 + b1 * 0.1804375) * 10) / 10;
      let y = Math.round((r1 * 0.2126729 + g1 * 0.7151522 + b1 * 0.072175) * 10) / 10;
      let z = Math.round((r1 * 0.0193339 + g1 * 0.119192 + b1 * 0.9503041) * 10) / 10;
      this.XYZ = [x, y, z];
      return `${x}\u00A0\u00A0\u00A0 ${y}\u00A0\u00A0\u00A0 ${z}`;
    },
    xyzToLab(xyz) {
      const [x, y, z] = xyz;

      // Коэффициенты для преобразования
      const kx = 95.047;
      const ky = 100.0;
      const kz = 108.883;

      const fx = x / kx;
      const fy = y / ky;
      const fz = z / kz;

      const epsilon = 0.008856;
      const kappa = 903.3;

      const f = (t) =>
        t > epsilon ? Math.pow(t, 1 / 3) : (kappa * t + 16) / 116;

      const L = Math.round((116 * f(fy) - 16) * 10) / 10;
      const a = Math.round(500 * (f(fx) - f(fy)) * 10) / 10;
      const b = Math.round(200 * (f(fy) - f(fz)) * 10) / 10;

      return `${L}, ${a}, ${b}`;
    },
  },
  computed: {
    RGB() {
      if (this.selectedPixel != null) {
        return {
          array: [this.selectedPixel[0], this.selectedPixel[1], this.selectedPixel[2]],
          string: `${this.selectedPixel[0]}, ${this.selectedPixel[1]}, ${this.selectedPixel[2]}`,
        };
      } else {
        return 0;
      }
    },
    contrastRatio() {
      if (this.rgb1 != [] && this.rgb2 != []) {
        const sRGBToLinear = (c) => {
          c = c / 255;
          return c <= 0.03928 ? c / 12.92 : Math.pow((c + 0.055) / 1.055, 2.4);
        };

        const getLuminance = (color) => {
          const rgb = color;
          const r = sRGBToLinear(rgb[0]);
          const g = sRGBToLinear(rgb[1]);
          const b = sRGBToLinear(rgb[2]);
          return 0.2126 * r + 0.7152 * g + 0.0722 * b;
        };

        const luminance1 = getLuminance(this.rgb1);
        const luminance2 = getLuminance(this.rgb2);

        const maxLuminance = Math.max(luminance1, luminance2);
        const minLuminance = Math.min(luminance1, luminance2);

        const contrastRatio = (maxLuminance + 0.05) / (minLuminance + 0.05);

        return Math.round(contrastRatio * 10) / 10
      } else {return 0}
    },
    coords() {
      if (
        this.startX != null &&
        this.startX > 0 &&
        this.startX < this.curWidth &&
        this.startY != null &&
        this.startY > 0 &&
        this.startY < this.curHeight
      ) {
        return { X: this.startX, Y: this.startY };
      } else {
        return 0;
      }
    },
  },
  mounted() {
    window.addEventListener("keydown", this.pressAlt);
    window.addEventListener("keyup", this.unpressAlt);
  },
};
</script>
<style lang="scss" scoped>
.modal {
  position: absolute;
  top: 12px;
  right: 12px;
  background-color: #e4e4e4;
  border: 1px solid black;
  width: 25vw;
  padding: 15px;
  display: flex;
  flex-direction: column;
  row-gap: 7px;

  &__container {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
  }

  &__line {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
  }

  &__line:nth-child(even) {
    padding-bottom: 5px;
    border-bottom: 1px solid rgb(83, 83, 83);
  }

  &__element {
    width: 100%;
    text-align: center;
  }
}

.color-button {
  appearance: none;
  cursor: pointer;
  border: 1px solid black;
  border-radius: 0;
  width: 23px;
  height: 23px;
  &:checked {
    border: 2px solid black;
  }
}

.header {
  &__text {
    font-size: 20px;
  }
  &__button {
    background: none;
    border: none;
    color: black;
    font-size: 24px;
    cursor: pointer;
  }
}
</style>