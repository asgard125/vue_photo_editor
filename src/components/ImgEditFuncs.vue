<template>
  <div style="display: flex; flex-direction: row; align-items: center; background: #76B8D0; width: 100%">
  <div class="dropdown">
    <button class="btn btn-secondary dropdown-toggle" type="button" id="dropdownMenuButton1" data-bs-toggle="dropdown" aria-expanded="false">
      Изменить размер
    </button>
    <div class="dropdown-menu" style="margin: 2%; min-width: 200px" aria-labelledby="dropdownMenuButton1">
      <div style="margin-left: 5%; display: flex; flex-direction: row; align-items: center">
        W<input class="form-control" id="c_width" style="width: 70px; margin: 1%" type="number" v-model="width">
        <input class="form-control" id="c_height" style="width: 70px; margin: 1%" type="number" v-model="height">H
      </div>
      <div style="display: flex; flex-direction: row; align-items: center; justify-content: center">
        <input type="radio" id="scale" value="scale" v-model="resizeType">
        <label for="scale" style="margin-right: 2%">растянуть</label>
        <input type="radio" id="noscale" value="noscale" v-model="resizeType">
        <label for="noscale">расширить</label>
      </div>
      <center><button class="btn btn-primary" id="reszie" @click="$emit('resize-canvas', resizeType, width, height);">Применить</button></center>
    </div>
  </div>
    <div class="dropdown">
      <button class="btn btn-secondary dropdown-toggle" type="button" id="dropdownMenuButton2" data-bs-toggle="dropdown" aria-expanded="false">
        Яркость и контраст
      </button>
      <div class="dropdown-menu" style="margin: 2%; min-width: 200px" aria-labelledby="dropdownMenuButton1">
        <div style="display: flex; flex-direction: column; align-items: center; justify-content: space-around">
          <div><input type="checkbox" v-model="allObjectsBandC">Все объекты</div>
        <center>Яркость</center>
        <input type="range" v-model="brightness" min="50" max="150">{{ brightness }}%
        <center>Контраст</center>
        <input type="range" v-model="contrast" min="50" max="150">{{ contrast }}%
        <center>Насыщенность</center>
        <input type="range" v-model="saturate" min="50" max="150">{{ saturate }}%
        <button class="btn btn-primary" id="apply_brightness_contrast" @click=send_brightness_contrast_data>Применить</button>
        </div>
      </div>
    </div>
    <div class="dropdown">
      <button class="btn btn-secondary dropdown-toggle" type="button" id="dropdownMenuButton5" data-bs-toggle="dropdown" aria-expanded="false">
        Эффекты
      </button>
      <div class="dropdown-menu" style="margin: 2%; min-width: 200px" aria-labelledby="dropdownMenuButton5">
        <div style="display: flex; flex-direction: column; align-items: center; justify-content: space-around">
          <div><input type="checkbox" v-model="allObjectsEffects">Все объекты</div>
          <center>Цвета</center>
          <input type="range" v-model="hue" min="0" max="200">{{ hue }}%
          <center>Размытие</center>
          <input type="range" v-model="blur" min="0" max="100">{{ blur }}%
          <center>Выделение деталей</center>
          <input type="range" v-model="vibrance" min="0" max="200">{{ vibrance }}%
          <button class="btn btn-primary" id="apply_effects" @click=send_effects_data>Применить</button>
        </div>
      </div>
    </div>
    <div class="dropdown">
      <button class="btn btn-secondary dropdown-toggle" type="button" id="dropdownMenuButton3" data-bs-toggle="dropdown" aria-expanded="false">
        Фильтры
      </button>
      <div class="dropdown-menu" style="margin: 2%; min-width: 200px" aria-labelledby="dropdownMenuButton2">
        <div style="display: flex; flex-direction: column; align-items: center; justify-content: space-around">
          <div style="display: flex; flex-direction: column; align-items: flex-start; justify-content: space-around">
          <div><input type="checkbox" v-model="allObjectsFilter">Все объекты</div>
          <div><input type="checkbox" v-model="invert">Негатив</div>
          <div><input type="checkbox" v-model="sepia">Сепия</div>
          <div><input type="checkbox" v-model="grayscale">Оттенки серого</div>
          <div><input type="checkbox" v-model="polaroid">Полароид</div>
            </div>
          <button class="btn btn-primary" id="apply_filters" @click=send_filters_data>Применить</button>
        </div>
      </div>
    </div>
    <div class="dropdown">
      <button class="btn btn-secondary dropdown-toggle" type="button" id="dropdownMenuButton4" data-bs-toggle="dropdown" aria-expanded="false">
        Положение
      </button>
      <div class="dropdown-menu" style="margin: 2%; min-width: 200px" aria-labelledby="dropdownMenuButton2">
        <div style="display: flex; flex-direction: column; align-items: center; justify-content: space-around">
          <button class="btn btn-primary" style="margin: 1%; width: 130px" id="rotate" @click="$emit('rotate')">поворот на 90</button>
          <button class="btn btn-primary" style="margin: 1%; width: 130px" id="flip-hor" @click="$emit('flip-horizontal')">отразить горизонтально</button>
          <button class="btn btn-primary" style="margin: 1%; width: 130px" id="flip-vert" @click="$emit('flip-vertical')">отразить вертикально</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

export default {
  name: "ImgEditFuncs",
  emits: ['resize-canvas', 'apply-brightness-contrast', 'apply-filters', 'rotate', 'flip-horizontal', 'flip-vertical'],
  data() {
    return {
      resizeType: 'scale',
      width: 400,
      height: 300,
      brightness: 100,
      contrast: 100,
      saturate: 100,
      grayscale: false,
      sepia: false,
      invert: false,
      polaroid: false,
      hue: 100,
      blur: 0,
      vibrance: 100,
      allObjectsBandC: false,
      allObjectsFilter: false,
      allObjectsEffects: false,

    }
  },
  methods: {
    send_brightness_contrast_data(){
      this.$emit('apply-brightness-contrast', this.allObjectsBandC, this.brightness - 100, this.contrast - 100, this.saturate - 100);
      this.brightness = 100;
      this.contrast = 100;
      this.saturate = 100;
      this.allObjectsBandC = false;
    },
    send_filters_data(){
      this.$emit('apply-filters', this.allObjectsFilter, {'grayscale': this.grayscale, 'sepia': this.sepia, 'invert': this.invert, 'polaroid': this.polaroid});
      this.grayscale = false;
      this.sepia = false;
      this.invert = false;
      this.polaroid = false;
      this.allObjectsFilter = false;
    },
    send_effects_data(){
      this.$emit('apply-effects', this.allObjectsEffects, {'hue': this.hue - 100, 'blur': this.blur, 'vibrance': this.vibrance - 100});
      this.hue = 100;
      this.blur = 0;
      this.vibrance = 100;
      this.allObjectsEffects = false;
    }
  }
}
</script>

<style scoped>

.btn-secondary{
  background-color: #76B8D0;
  border-color: #76B8D0;
}


</style>