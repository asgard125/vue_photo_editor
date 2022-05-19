<template>
  <div id="app" class="main-div">
    <div>
    <div style="display: flex; flex-direction: row; align-items: center">
      <button class="btn btn-primary" id="uploadbtn" @click="upload">Загрузить</button>
      <button class="btn btn-primary" id="savebtn" @click="download">Скачать</button>
      <button class="btn btn-primary" id="uploadbtnJSON" @click="uploadJSON" style="min-width: 150px">Загрузить JSON</button>
      <button class="btn btn-primary" id="savebtnJSON" @click="downloadJSON" style="min-width: 150px">Скачать JSON</button>
    <ImgEditFuncs @resize-canvas="resize" @apply-filters="applyFilters" @apply-brightness-contrast="brightness_contrast"
                  @apply-effects="applyEffects" @flip-horizontal="flip_horizontal" @flip-vertical="flip_vertical" @rotate="rotate_90"
    />
    </div>
    <div class="painter-div">
      <div style="margin-right: 1%; max-width: 230px;">
        <Chrome  v-model="cpcolor"
        />
        <div style="display: flex; flex-direction: column;">
      <div style="margin-top: 3%; display: flex; flex-direction: row;">
        <input class="form-control" style="width: 225px;" type="number" :value="lineWidth" @input="setLineWidth">
      </div>
          Тип линии
          <div id="line-types" style="margin-top: 3%; display: flex; flex-direction: row; flex-flow: row wrap;">
            <button class="btn btn-light" value="select" @click="setTool" style="margin-right: 1%; background: #AAAAAA; min-width: 100px;">рука</button>
            <button class="btn btn-light" value="pencil" @click="setTool" style="margin-right: 1%; background: #E0E0E0; min-width: 100px;">обычная</button>
            <button class="btn btn-light" value="round" @click="setTool" style="margin-right: 1%; background: #E0E0E0; min-width: 100px;">круглая</button>
            <button class="btn btn-light" value="spray" @click="setTool" style="margin-right: 1%; background: #E0E0E0; min-width: 100px;">спрей</button>
          </div>
          Инструменты
          <div id="tools" style="margin-top: 3%; display: flex; flex-direction: row; flex-flow: row wrap;">
            <button class="btn btn-light" value="colorsucker" @click="setTool" style="margin-right: 1%; background: #E0E0E0; min-width: 100px;">пипетка</button>
            <button class="btn btn-light" @click="addText" style="margin-right: 1%; background: #E0E0E0; min-width: 100px;">текст</button>
          </div>
      </div>
      </div>
      <canvas id="paint_field" class="canvas" width=800 height=600 @mousedown="console.log('a')" v-on:keyup="return_back"/>
    </div>
    </div>
    <div class="footer">
      <input type="range" @change="setZoom" min="25" max="200" step="25" :value="zoom">{{ zoom }}%
    </div>
  </div>
</template>

<script>
import fs from 'fs';
const {dialog} = require('@electron/remote');

import {Chrome} from '@ckpack/vue-color';
import ImgEditFuncs from "@/components/ImgEditFuncs";
const fabric = require("fabric").fabric;
import 'fabric';
import 'fabric-history';




export default {
  name: "App",
  data(){
    return{
      canvasHeight: 0,
      canvasWidth: 0,
      history: [],
      x: 0,
      y: 0,
      lineWidth: 5,
      zoom: 100,
      canvasAngle: 0,
      tool: 'select',
      canvas2dBackend: null,
      activeToolColor: '#AAAAAA',
      inactiveToolColor: '#E0E0E0',
      colors: 'rgba(0, 0, 0, 1)',
      cpcolor: 'rgba(0, 0, 0, 1)',
      filters: { 'grayscale': [0, new fabric.Image.filters.Grayscale()], 'invert': [1, new fabric.Image.filters.Invert()],
        'sepia': [3, new fabric.Image.filters.Sepia()], 'polaroid': [2, new fabric.Image.filters.Polaroid()], 'hue': [4, new fabric.Image.filters.HueRotation(), 'rotation'],
        'vibrance': [8, new fabric.Image.filters.Vibrance(), 'vibrance'], 'blur': [9, new fabric.Image.filters.Blur(), 'blur']}
      }
    },
  mounted() {
    this.canvas = new fabric.Canvas("paint_field");
    document.addEventListener('keyup', this.keyupHandler);
    this.canvas.freeDrawingBrush = new fabric.PencilBrush(this.canvas);
    this.canvas.freeDrawingBrush.selectable = false;
    this.canvas.freeDrawingBrush.evented = false;
    this.canvas.uniScaleTransform = true;
    this.canvas2dBackend = new fabric.Canvas2dFilterBackend();
    fabric.filterBackend = fabric.initFilterBackend();
  },
  components: {ImgEditFuncs, Chrome},
  watch: {
    cpcolor(){
      this.changeColor();
    }
  },
  methods: {
    keyupHandler (event) {
      if (event.ctrlKey && event.code === 'KeyZ') {
        this.canvas.undo();
      }
      if (event.ctrlKey && event.code === 'KeyY') {
        this.canvas.redo();
      }
      if (event.code === 'Delete') {
        let objects = this.canvas.getActiveObjects();
        for (let i in objects) {
          this.canvas.remove(objects[i]);
        }
        this.canvas.renderAll();
      }
      if (event.shiftKey && event.code === 'ArrowDown') {
        console.log('down')
        let objects = this.canvas.getActiveObjects();
        let ind;
        for (let i in objects) {
          ind = this.canvas.getObjects().indexOf(objects[i]);
          this.canvas.moveTo(objects[i], Math.max(0, ind - 1));
          objects[i].moveTo(Math.max(0, ind - 1));
        }
        this.canvas.discardActiveObject().renderAll();
      }
      if (event.shiftKey && event.code === 'ArrowUp') {
        console.log('up')
        let objects = this.canvas.getActiveObjects();
        let ind;
        for (let i in objects) {
          ind = this.canvas.getObjects().indexOf(objects[i]);
          this.canvas.moveTo(objects[i], Math.min(this.canvas.getObjects().length, ind + 1));
          objects[i].moveTo(Math.min(this.canvas.getObjects().length, ind + 1));
        }
        this.canvas.discardActiveObject().renderAll();
      }
    },
    changeColor(){
      const {r, g, b, a} = this.cpcolor.rgba;
      this.colors = `rgba(${r}, ${g}, ${b}, ${a})`;
      this.canvas.freeDrawingBrush.color = `rgba(${r}, ${g}, ${b}, ${a})`;
    },
    flip_horizontal(){
      this.canvas.getObjects().forEach((obj) => {
        obj.flipX = !obj.flipX;
      });
      this.canvas.renderAll();
    },
    flip_vertical(){
      this.canvas.getObjects().forEach((obj) => {
        obj.flipY = !obj.flipY;
      });
      this.canvas.renderAll();

    },
    rotate_90(){
      let PositionAdjustment = ((this.canvas.getWidth() - this.canvas.getHeight()) / 2);

      this.canvasAngle = 90;
      let canvasCenter = new fabric.Point(this.canvas.getWidth() / 2, this.canvas.getHeight() / 2)
      let radians = fabric.util.degreesToRadians(this.canvasAngle)

      this.canvas.getObjects().forEach((obj) => {
        let objectOrigin = new fabric.Point((obj.left + PositionAdjustment), obj.top + PositionAdjustment)
        let new_loc = fabric.util.rotatePoint(objectOrigin, canvasCenter, radians)
        obj.top = new_loc.y
        obj.left = new_loc.x
        obj.angle += this.canvasAngle;
        obj.setCoords();
      });
      const w = this.canvas.getWidth();
      const h = this.canvas.getHeight();
      this.canvas.setWidth(h);
      this.canvas.setHeight(w);
      this.canvas.renderAll();
    },
    resize(resizeType, w, h){
      this.resetZoom();
      if (this.canvas.width !== w || this.canvas.height !== h) {
        let scaleMultiplierX = w / this.canvas.width;
        let scaleMultiplierY = h / this.canvas.height;
        if(resizeType === 'scale') {
          let objects = this.canvas.getObjects();
          for (let i in objects) {
            objects[i].scaleX = objects[i].scaleX * scaleMultiplierX;
            objects[i].scaleY = objects[i].scaleY * scaleMultiplierY;
            objects[i].left = objects[i].left * scaleMultiplierX;
            objects[i].top = objects[i].top * scaleMultiplierY;
            objects[i].setCoords();
          }
          let obj = this.canvas.backgroundImage;
          if (obj) {
            obj.scaleX = obj.scaleX * scaleMultiplierX;
            obj.scaleY = obj.scaleY * scaleMultiplierY;
          }
        }
        this.canvas.discardActiveObject();
        this.canvas.setWidth(this.canvas.getWidth() * scaleMultiplierX);
        this.canvas.setHeight(this.canvas.getHeight() * scaleMultiplierY);
        this.canvas.renderAll();
        this.canvas.calcOffset();
      }
    },
    resetZoom(){
      this.canvas.setZoom(this.canvas.getZoom()*(1/(this.zoom / 100)));
      this.canvas.setWidth(this.canvas.getWidth() * (1/(this.zoom / 100)));
      this.canvas.setHeight(this.canvas.getHeight() * (1/(this.zoom / 100)));
      this.zoom = 100;
    },
    setZoom(e) {
      this.resetZoom();
      this.zoom = e.target.value;
      let zoom = Number(e.target.value) / 100;
      this.canvas.setZoom(this.canvas.getZoom() * zoom);
      this.canvas.setWidth(this.canvas.getWidth() * zoom);
      this.canvas.setHeight(this.canvas.getHeight() * zoom);
      this.canvas.renderAll();
    },
    applyFilters(allObjects, f) {
      let objects;
      if (allObjects) {
        objects = this.canvas.getObjects();
      }else {
        objects = this.canvas.getActiveObjects();
      }
        for (let i in objects) {
          for (let j in Object.keys(this.filters)) {
            let f_key = Object.keys(this.filters)[j];
            if (objects[i].type !== "path") {
              if (f[f_key]) {
                objects[i].filters[this.filters[f_key][0]] = this.filters[f_key][1]
              } else {
                objects[i].filters[this.filters[f_key][0]] = null;
              }
              objects[i].applyFilters();
            }
          }
        }
      this.canvas.renderAll();
    },
    applyEffects(allObjects, f) {
      let objects;
      if (allObjects) {
        objects = this.canvas.getObjects();
      }else {
        objects = this.canvas.getActiveObjects();
      }
      let f_keys = Object.keys(f);
      for (let i in objects) {
        for (let j in Object.keys(this.filters)) {
          let f_key = Object.keys(this.filters)[j];
          if (objects[i].type !== "path" && f_keys.indexOf(f_key) !== -1) {
            objects[i].filters[this.filters[f_key][0]] = this.filters[f_key][1];
            objects[i].filters[this.filters[f_key][0]][this.filters[f_key][2]] = f[f_key] / 100;
            objects[i].applyFilters();
          }
        }
      }
      this.canvas.renderAll();
    },
    brightness_contrast(allObjects, brightness, contrast, saturate){
      let objects;
      if (allObjects) {
        objects = this.canvas.getObjects();
      }else {
        objects = this.canvas.getActiveObjects();
      }
      for (let i in objects) {
        if (objects[i].type !== "path") {
          objects[i].filters[5] = new fabric.Image.filters.Brightness({brightness: brightness / 100});
          objects[i].filters[6] = new fabric.Image.filters.Contrast({contrast: contrast / 100});
          objects[i].filters[7] = new fabric.Image.filters.Saturation({saturation: saturate / 100});
          objects[i].applyFilters();
        }
      }
      this.canvas.renderAll();
    },
    resetTools(){
      let tools = document.getElementById("tools");
      for (let i = 0; i < tools.children.length; i++) {
        let elem = tools.children[i];
        elem.style.background = this.inactiveToolColor;
      }
      let lines = document.getElementById("line-types");
      for (let i = 0; i < lines.children.length; i++) {
        let elem = lines.children[i];
        elem.style.background = this.inactiveToolColor;
      }
    },
    colorsuckerHandler(e){
      let pointer = this.canvas.getPointer(e);
      let posX = pointer.x;
      let posY = pointer.y;

      let activeObject = this.canvas.getActiveObject();
      if(activeObject) {
        console.log(posX, posY);
        let new_color = this.pixelAt(activeObject.canvas.getContext("2d"), posX, posY)
        this.cpcolor = {rgba: {r:new_color[0], g:new_color[1], b:new_color[2], a:new_color[3] }}
      }
    },
    setTool(e){
      this.resetTools();
      this.tool = e.target.value;
      e.target.style.background = this.activeToolColor;
      this.canvas.off('mouse:down', this.colorsuckerHandler);
      this.canvas.isDrawingMode = false;
      switch (this.tool) {
        case "select":
          this.canvas.isDrawingMode = false;
          break;
        case "round":
          this.canvas.freeDrawingBrush = new fabric.CircleBrush(this.canvas);
          this.canvas.freeDrawingBrush.width = this.lineWidth;
          this.canvas.freeDrawingBrush.color = this.colors;
          this.canvas.isDrawingMode = true;
          break;
        case "pencil":
          this.canvas.freeDrawingBrush = new fabric.PencilBrush(this.canvas);
          this.canvas.freeDrawingBrush.width = this.lineWidth;
          this.canvas.freeDrawingBrush.color = this.colors;
          this.canvas.isDrawingMode = true;
          break;
        case "spray":
          this.canvas.freeDrawingBrush = new fabric.SprayBrush(this.canvas);
          this.canvas.freeDrawingBrush.width = this.lineWidth;
          this.canvas.freeDrawingBrush.color = this.colors;
          this.canvas.isDrawingMode = true;
          break;
        case "colorsucker":
          this.canvas.isDrawingMode = false;
          this.canvas.on('mouse:down', this.colorsuckerHandler);
          break;
        default:
          break;
      }
    },
    addText(){
      let text = new fabric.IText('Пример текста', {
        fontFamily: 'arial black',
        left: 100,
        top: 100 ,
        fill: this.colors
      });
      this.canvas.add(text);
      this.canvas.setActiveObject(text);
      this.canvas.renderAll();
    },
    setLineWidth(e){
      this.lineWidth = Number(e.target.value);
      this.canvas.freeDrawingBrush.width = this.lineWidth;
    },
    getRGB(str){
      let match = str.match(/rgba?\((\d{1,4}), ?(\d{1,4}), ?(\d{1,4}), ?(\d{1,4})\)?(?:, ?(\d(?:\.\d*)?)\))?/);
      return match ? {
        r: match[1],
        g: match[2],
        b: match[3],
        a: match[4]
      } : {};
    },
    pixelAt(object, x, y) {
      return object.getImageData(x, y, 1, 1).data;
    },
    download() {
      dialog.showSaveDialog({filters: [
          { name: 'PNG', extensions: ['png'] },
          { name: 'JPEG', extensions: ['jpg'] }
        ], defaultPath: 'result.png'}).then((result) => {
        let base64Data = null;
        this.resetZoom();
        if (result.filePath.includes('png')) {
          let url = this.canvas.toDataURL({
            format: 'png',
            enableRetinaScaling: true
          });
          base64Data = url.replace(/^data:image\/png;base64,/, "");
        }else{
          this.canvas.setBackgroundColor('rgba(255, 255, 255, 1)', this.canvas.renderAll.bind(this.canvas));
          let url = this.canvas.toDataURL({
            format: 'jpeg',
            enableRetinaScaling: true
          });
          base64Data = url.replace(/^data:image\/jpeg;base64,/, "");
          this.canvas.setBackgroundColor('rgba(255, 255, 255, 0)', this.canvas.renderAll.bind(this.canvas));
        }
        this.canvas.renderAll();
        fs.writeFile(result.filePath, base64Data, 'base64', function (err) {
          console.log(err);
        });
      }).catch((err) => {
        err
      });
    },
    upload() {
      dialog.showOpenDialog({filters: [
          { name: 'Images', extensions: ['png', 'jpeg', 'jpg'] }
        ], properties: ['openFile']}).then((result) => {
          this.resetZoom();
          let img = new Image();
          img.src = result.filePaths;
          let parent = this;
          img.onload = function () {
            let image = new fabric.Image(this);
            image.set({
              angle: 0,
              height:this.naturalHeight,
              width:this.naturalWidth,
            });
            parent.canvas.centerObject(image);
            parent.canvas.add(image);
            parent.canvas.setActiveObject(image);
            parent.canvas.renderAll();
          }
      }).catch((err) => {
        err
      });
    },
    downloadJSON() {
      dialog.showSaveDialog({filters: [
          { name: 'json', extensions: ['json'] }
        ], defaultPath: 'resultjson.json'}).then((result) => {
        this.resetZoom();
        fs.writeFile(result.filePath,  JSON.stringify(this.canvas), function (err) {
          console.log(err);
        });
      }).catch((err) => {
        err
      });
    },
    uploadJSON() {
      dialog.showOpenDialog({filters: [
          { name: 'JSONS', extensions: ['json'] }
        ], properties: ['openFile']}).then((result) => {
        this.resetZoom();
        this.canvas.clear();
        this.canvas.loadFromJSON(JSON.parse(fs.readFileSync(result.filePaths[0], 'utf8')));
        this.canvas.renderAll();
      }).catch((err) => {
        console.log(err)
      });
    }
  }
}
</script>

<style>


.main-div{
  height: 100vh;
}

.painter-div{
  display: flex;
  margin-top: 1%;
  margin-left: 2%;
  flex-flow: row nowrap;
  align-items: flex-start;
  align-content: center;
  justify-content: flex-start;
  flex-direction: row;
}

.canvas{
  border: 2px dashed black;
}

.footer {
  min-height: 30px; /* Высота слоя */
  bottom: 0;
  width: 100%;
  z-index: 999999;
  background: #76B8D0;
  position: fixed;
  border-bottom: 2px solid #000;
}

</style>