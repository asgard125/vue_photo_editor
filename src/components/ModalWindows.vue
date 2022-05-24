<template>
  <!-- Modal -->
  <div class="modal fade" id="HelpModalWindow" tabindex="-1" role="dialog"
       ref="modal"
       :class="{ show: activeHelp, 'd-block': activeHelp }"
       aria-labelledby="exampleModalCenterTitle" aria-hidden="true">
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="exampleModalLongTitle">Справка</h5>
        </div>
        <div class="modal-body">
          Ctrl + Z - отменить действие<br>
          Ctrl + Y - отменить отмену<br>
          F1 - Справка<br>
          Delete - удалить объект<br>
          F8 (Текст - активный) - Изменить параметры текста<br>
          Shift + ArrowUp - поднять выделенный объект<br>
          Shift + ArrowDown - опустить выделенный объект
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" data-dismiss="modal" @click="showHelpWindow">Закрыть</button>
        </div>
      </div>
    </div>
  </div>
  <div class="modal fade" id="ChangeModalWindow" tabindex="-1" role="dialog"
       ref="modal"
       :class="{ show: activeChange, 'd-block': activeChange }"
       aria-labelledby="exampleModalCenterTitle" aria-hidden="true">
    <div class="modal-dialog" role="document">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title" id="example">Редактирование текста</h5>
        </div>
        <div class="modal-body">
          <div style="display: flex; flex-flow: column">
            <a>Цвет текста (задний)</a>
            <div><input type="checkbox" v-model="transparent">Прозрачный</div>
            <input type="color" v-model="backColor">
            <a>Цвет текста (текст)</a>
            <input type="color" v-model="textColor">
            <a>Стиль текста (обводка)</a>
            <select v-model="textStyle">
              <option disabled value="">no style</option>
              <option>normal</option>
              <option>italic</option>
              <option>bold</option>
            </select>
            <a><input type="checkbox" v-model="textShadow">Тень</a>
          </div>
        </div>
        <div class="modal-footer">
          <button type="button" class="btn btn-secondary" data-dismiss="modal" @click="sendChanges">Подтвердить</button>
          <button type="button" class="btn btn-secondary" data-dismiss="modal" @click="showChangeWindow">Закрыть</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
let emitter = require('tiny-emitter/instance');

export default {
  name: "ModalWindows",
  data() {
    return{
      activeHelp: false,
      activeChange: false,
      backColor: '#000000',
      textColor: '#000000',
      transparent: false,
      textStyle: 'normal',
      textShadow: false
  }
  },
  created() {
    emitter.on('help-window', this.showHelpWindow)
    emitter.on('change-window', this.showChangeWindow)
  },
  methods: {
    showHelpWindow(){
      const body = document.querySelector("#HelpModalWindow")
      this.activeHelp = !this.activeHelp
      this.activeHelp ? body.classList.add("modal-open") : body.classList.remove("modal-open")
    },
    showChangeWindow() {
      const body = document.querySelector("#ChangeModalWindow")
      this.activeChange = !this.activeChange
      this.activeChange ? body.classList.add("modal-open") : body.classList.remove("modal-open")
    },
    sendChanges(){
      let textShadow;
      if (this.textShadow){
        textShadow = 'rgba(0,0,0, 0.7)';
      }else{
        textShadow = 'rgba(0,0,0, 0.0)';
      }
      if (this.transparent) {
        emitter.emit('change-send', {
          'backColor': null,
          'textColor': this.textColor,
          'textStyle': this.textStyle,
          'shadow': textShadow
        })
      }else{
        emitter.emit('change-send', {
          'backColor': this.backColor,
          'textColor': this.textColor,
          'textStyle': this.textStyle,
          'shadow': textShadow
        })
      }
      this.showChangeWindow()
    }
  }
}
</script>

<style scoped>

</style>