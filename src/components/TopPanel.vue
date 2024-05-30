<template>
  <div class="content">
    <button class="button" @click="$emit('show')" title="Upload your image!">Загрузить фото</button>
    <button class="button" :disabled="!startImage" :class="{ active: isPippet}" @click="$emit('pippet')" title="Color picker"><img class="button__img" src="@/assets/pipette.svg" alt=""></button>
    <button class="button" :disabled="!startImage" :class="{ active: isGrab}" @click="$emit('grab')" title="Grab your image"><img class="button__img" src="@/assets/hand-grab.svg" alt=""></button>
    <button class="button" :disabled="!startImage" @click="$emit('scale')" title="Resize your image">Изменить размер</button>
    <button class="button" :disabled="!startImage" :class="{ active: isCurves}" @click="$emit('curve')" title="Save your image">Кривая</button>
    <button class="button" :disabled="!startImage" :class="{ active: isFiltering}" @click="$emit('filtering')" title="Save your image">Фильтры</button>
    <button class="button" :disabled="!startImage" @click="$emit('save')" title="Save your image">Сохранить</button>
    <button class="button" :disabled="!startImage" @click="$emit('clear')" title="Clear">Очистить</button>
  </div>
</template>

<script>
export default {
  name: "HeaderPanel",
  props: {
    state: String,
    startImage: Object,
  },
  data() {
    return {
      new: null,
    };
  },

  methods: {
  },
  computed:{
    isPippet(){
      if (this.state == 'pippet'){
        return true
      } else {
        return false
      }
    },
    isGrab(){
      if (this.state == 'grab'){
        return true
      } else {
        return false
      }
    },
    isCurves(){
      if (this.state == 'curves'){
        return true
      } else {
        return false
      }
    },
    isFiltering(){
      if (this.state == 'filter'){
        return true
      } else {
        return false
      }
    }
  }
};
</script>
  
<style lang="scss" scoped>
.content {
  height: 60px;
  background-color: #305aa7;
  display: flex;
  flex-direction: row;
  align-items: center;
  column-gap: 15px;
  padding: 10px;
}
.button{
  cursor: pointer;
  background-color: white;
  border: 1px solid #305aa7;
  border-radius: 5px;
  padding: 7px;
  height: fit-content;
  margin-left: 20px;

  &__img{
    height: 20px;
  }

  &:last-child{
    margin-left: auto;
    margin-right: 20px;
  }

  &:disabled{
    background-color: #a5a5a5;
    cursor: default;
  }

  &.active{
    background-color: #a5a5a5;
  }
}
</style>
  