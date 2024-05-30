<template>
  <div class="content">
    <p class="font">x: {{ x }}px, y: {{ y }}px</p>
    <template v-if="showData">
      <p class="font">Исходные параметры изображения:</p>
      <div class="params">
        <p class="font">Ширина: {{ width }}px</p> 
        <p class="font">Высота: {{ height }}px</p>
      </div>
      <select :disabled="state!=''" v-model="scale" @change="$emit('scale')">
        <option value="12">12%</option>
        <option value="25">25%</option>
        <option value="50">50%</option>
        <option value="100">100%</option>
        <option value="150">150%</option>
        <option value="200">200%</option>
        <option value="300">300%</option>
      </select>
      <div v-if="selectedPixel != null" class="color">
          <div class="color__img" :style="selectedColorStyle"></div>
          <div class="color__inf">
            <p class="color__font">{{ color }}</p>
            <p class="color__font">Hex: {{ HEX }}</p>
          </div>
        </div>
    </template>
  </div>
</template>
  
  <script>
export default {
  name: "SidebarPanel",
  props: {
    x: Number,
    y: Number,
    height: Number,
    width: Number,
    showData: Boolean,
    selectedPixel: Array,
    state: String
  },
  data() {
    return {
      picker: null,
      scale: 100,
    }
  },
  computed: {
    selectedColorStyle() {
      return (
        "background-color: rgb(" + this.selectedPixel[0] + "," + "," + this.selectedPixel[2] + ");"
      );
    },
    color() {
      return (
        "rgb(" + this.selectedPixel[0] + ", " + this.selectedPixel[1] + ", " + this.selectedPixel[2] + ")"
      );
    },
    HEX() {
      return (
        "#" + [this.selectedPixel[0], this.selectedPixel[1], this.selectedPixel[2]].map((x) => {
            const hex = x.toString(16);
            return hex.length === 1 ? "0" + hex : hex;
          }).join("")
      );
    },
  }
};
</script>
  
  <style lang="scss" scoped>
.content {
  border-right: 1px solid white;
  background-color: #305aa7;
  display: flex;
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  height: 40px;
  gap: 20px;
  padding: 10px;
}
.color {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  column-gap: 10px;

  &__img{
  height: 25px;
  width: 25px;
  border: 1px solid white;
  }

  &__font{
    color: white;
  }

  &__inf{
    display: flex;
  }
}
.font{
  color: white;
  &_center{
    align-self: center;
  }
}
.params{
  padding-left: 15px;
  display: flex;
  gap: 20px;
}
</style>
  