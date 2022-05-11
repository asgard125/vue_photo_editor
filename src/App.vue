<template>
  <div id="app" class="main-div" @mouseleave="stopDrawing" @mouseup.left="stopDrawing">
    <div>
    <div style="display: flex; flex-direction: row; align-items: center">
      <button class="btn btn-primary" id="uploadbtn" @click="upload">Загрузить</button>
      <button class="btn btn-primary" id="savebtn" @click="download">Скачать</button>
    <ImgEditFuncs @resize-canvas="resize" @apply-filters="filters" @apply-brightness-contrast="brightness_contrast"
                  @flip-horizontal="flip_horizontal" @flip-vertical="flip_vertical" @rotate="rotate_90"
    />
    </div>
    <div class="painter-div">
      <div style="margin-right: 1%">
      <Chrome v-model="colors"/>
        <div style="display: flex; flex-direction: column;">
      <div style="margin-top: 3%; display: flex; flex-direction: row;">
        <button class="btn btn-light" @click="MinusWidth" style="margin-right: 1%; background: #E0E0E0;">-</button>
        <input class="form-control" style="width: 70px;" type="number" v-model="lineWidth">
        <button class="btn btn-light" @click="PlusWidth" style="margin-left: 1%; background: #E0E0E0;">+</button>
      </div>
          Тип линии
          <div id="line-types" style="margin-top: 3%; display: flex; flex-direction: row;">
          <button class="btn btn-light" value="round" @click="setTool" style="margin-right: 1%; background: #AAAAAA;">●</button>
            <button class="btn btn-light" value="butt" @click="setTool" style="margin-right: 1%; background: #E0E0E0">/</button>
            <button class="btn btn-light" value="square" @click="setTool" style="margin-right: 1%; background: #E0E0E0">■</button>
          </div>
          Инструменты
          <div id="tools" style="margin-top: 3%; display: flex; flex-direction: row;">
            <button class="btn btn-light" value="filler" @click="setTool" style="margin-right: 1%; background: #E0E0E0">Заливка</button>
            <button class="btn btn-light" value="colorSucker" @click="setTool" style="margin-right: 1%; background: #E0E0E0">Пипетка</button>
          </div>
      </div>
      </div>
      <canvas id="paint_field" class="canvas" width=400 height=300 @mousemove="draw" @mousedown.left="beginDrawing" @mouseleave="ResetCords" v-on:keyup="return_back"/>
    </div>
    </div>
    <div class="footer">
    </div>
  </div>
</template>

<script>
import fs from 'fs'
const {dialog} = require('@electron/remote');
import {Chrome} from '@ckpack/vue-color';
import resizeCanvas from 'resize-canvas'
import ImgEditFuncs from "@/components/ImgEditFuncs";




export default {
  name: "App",
  data(){
    return{
      canvas: null,
      canvasHeight: 0,
      canvasWidth: 0,
      history: [],
      x: 0,
      y: 0,
      lineWidth: 5,
      scale: 1,
      tool: 'round',
      activeToolColor: '#AAAAAA',
      inactiveToolColor: '#E0E0E0',
      isDrawing: false,
      colors: {rgba: {r: 0, g: 0, b: 0, a: 1}}
    }
  },
  mounted() {
    let c = document.getElementById("paint_field");
    this.canvas = c.getContext('2d');
    this.canvas.imageSmoothingQuality = 'high';
    this.canvas.imageSmoothingEnabled = true;
    document.addEventListener('keyup', this.keyupHandler)
  },
  components: {ImgEditFuncs, Chrome},
  methods: {
    keyupHandler (event) {
      if (event.ctrlKey && event.code === 'KeyZ') {
        this.return_back()
      }
    },
    getTempCanvas(){
      let tempCanvas = document.createElement('canvas');
      tempCanvas.width = this.canvas.canvas.width;
      tempCanvas.height = this.canvas.canvas.height;
      tempCanvas.imageSmoothingQuality = 'high';
      tempCanvas.imageSmoothingEnabled = true;
      tempCanvas = tempCanvas.getContext("2d");
      tempCanvas.drawImage(this.canvas.canvas, 0, 0);
      return tempCanvas;
    },
    make_history(){
      if(this.history.length === 5){
        this.history.shift();
      }
      let tempCanvas = this.getTempCanvas();
      this.history.push(tempCanvas.canvas);
      console.log('added');
    },
    return_back(){
      if (this.history.length > 0) {
        let oldCanvas = this.history.pop();
        this.canvas.canvas.width = oldCanvas.width;
        this.canvas.canvas.height = oldCanvas.height;
        this.canvas.drawImage(oldCanvas, 0, 0);
      }
    },
    flip_horizontal(){
      this.make_history();
      let tempCanvas = this.getTempCanvas();
      this.canvas.translate(this.canvas.canvas.width, 0);
      this.canvas.scale(-1, 1);
      this.canvas.clearRect(0, 0, this.canvas.canvas.width, this.canvas.canvas.height);
      this.canvas.drawImage(tempCanvas.canvas, 0, 0);
      this.canvas.scale(-1, 1);
      this.canvas.translate(-this.canvas.canvas.width, 0);
    },
    flip_vertical(){
      this.make_history();
      let tempCanvas = this.getTempCanvas();
      this.canvas.translate(0, this.canvas.canvas.height);
      this.canvas.scale(1, -1);
      this.canvas.clearRect(0, 0, this.canvas.canvas.width, this.canvas.canvas.height);
      this.canvas.drawImage(tempCanvas.canvas, 0, 0);
      this.canvas.scale(1, -1);
      this.canvas.translate(0, -this.canvas.canvas.height);
    },
    rotate_90(){
      this.make_history();
      let tempCanvas = this.getTempCanvas();
      this.canvas.canvas.width = tempCanvas.canvas.height;
      this.canvas.canvas.height = tempCanvas.canvas.width;
      this.canvas.rotate(90*Math.PI/180);
      this.canvas.drawImage(tempCanvas.canvas, 0, -this.canvas.canvas.width);
      this.canvas.rotate(270*Math.PI/180);
    },
    resize(resizeType, w, h){
      this.make_history();
      if (resizeType === 'scale') {
        let tempCanvas = document.createElement('canvas');

        tempCanvas.width = w;
        tempCanvas.height = h;
        tempCanvas = tempCanvas.getContext("2d");
        tempCanvas.imageSmoothingQuality = 'high';
        tempCanvas.imageSmoothingEnabled = true;
        tempCanvas.drawImage(this.canvas.canvas, 0, 0, this.canvas.canvas.width, this.canvas.canvas.height, 0, 0, w, h);

        this.canvas.canvas.height = h;
        this.canvas.canvas.width = w;

        this.canvas.drawImage(tempCanvas.canvas, 0, 0, w, h)
      }else {
        resizeCanvas({
          canvas: this.canvas.canvas,
          diff: [w - this.canvas.canvas.width, h - this.canvas.canvas.height],
          from: [0, 0]
        })
      }
    },
    filters(grayscale, sepia, invert){
      this.make_history();
      this.canvas.filter = `invert(${invert / 100}) sepia(${sepia / 100}) grayscale(${grayscale / 100})`;
      this.canvas.drawImage(this.canvas.canvas, 0, 0);
      this.canvas.filter = 'none';
    },
    brightness_contrast(brightness, contrast, saturate){
      this.make_history();
      this.canvas.filter = `brightness(${brightness / 100}) contrast(${contrast / 100}) saturate(${saturate / 100})`;
      this.canvas.drawImage(this.canvas.canvas, 0, 0);
      this.canvas.filter = 'none';
    },
    drawLine(x1, y1, x2, y2) {
      this.canvas.beginPath();
      const {rgba: {r, g, b, a}} = this.colors;
      this.canvas.strokeStyle = `rgba(${r}, ${g}, ${b}, ${a})`;
      this.canvas.lineWidth = this.lineWidth;
      this.canvas.lineCap = this.tool;
      this.canvas.moveTo(x1, y1);
      this.canvas.lineTo(x2, y2);
      this.canvas.stroke();
      this.canvas.closePath();
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
    setTool(e){
      this.resetTools();
      this.tool = e.target.value;
      e.target.style.background = this.activeToolColor;
    },
    draw(e) {
      if (this.isDrawing) {
        if (this.x === -1 && this.y === -1) {
          this.drawLine(e.offsetX, e.offsetY, e.offsetX, e.offsetY);
        }
        else{
          this.drawLine(this.x, this.y, e.offsetX, e.offsetY);
        }
        this.x = e.offsetX;
        this.y = e.offsetY;
      }
    },
    fill_canvas(startX, startY){
      let img = this.canvas.getImageData(0, 0, this.canvas.canvas.width, this.canvas.canvas.height);
      let imgData = img.data;
      const {rgba: {r, g, b}} = this.colors;
      let canvasWidth = this.canvas.canvas.width;
      let canvasHeight = this.canvas.canvas.height;
      let r_target = imgData[startY * canvasWidth * 4 + startX * 4];
      let g_target = imgData[startY * canvasWidth * 4 + startX * 4 + 1];
      let b_target = imgData[startY * canvasWidth * 4 + startX * 4 + 2];
      if (startX > 0 && startX < this.canvas.canvas.width && startY > 0 && startY < this.canvas.canvas.height) {
        let pixelStack = [[startX, startY]];
        let newPos, x, y, pixelPos;
        let colored = Array(canvasWidth * canvasHeight * 4).fill(0);
        while (pixelStack.length > 0) {
          newPos = pixelStack.pop();
          x = newPos[0];
          y = newPos[1];
          pixelPos = (y * canvasWidth + x) * 4;
          while (y-- >= 0 && this.matchStartColor(pixelPos, imgData, r_target, g_target, b_target) && colored[pixelPos] !== 1) {
            pixelPos -= canvasWidth * 4;
          }
          pixelPos += canvasWidth * 4;
          ++y;
          while (y++ < canvasHeight - 1 && this.matchStartColor(pixelPos, imgData, r_target, g_target, b_target) && colored[pixelPos] !== 1) {
            this.colorPixel(pixelPos, imgData, r, g ,b);
            colored[pixelPos] = 1;
            if (x > 0) {
              if (this.matchStartColor(pixelPos - 4, imgData, r_target, g_target, b_target) && colored[pixelPos - 4] !== 1) {
                  pixelStack.push([x - 1, y]);
              }
            }

            if (x < canvasWidth - 1) {
              if (this.matchStartColor(pixelPos + 4, imgData, r_target, g_target, b_target) && colored[pixelPos + 4] !== 1) {
                  pixelStack.push([x + 1, y]);
              }
            }

            pixelPos += canvasWidth * 4;
          }
        }
        this.canvas.putImageData(img, 0, 0);
      }
    },
    matchStartColor(pixelPos, imgData, r_target, g_target, b_target) {
      let r_current = imgData[pixelPos];
      let g_current = imgData[pixelPos + 1];
      let b_current = imgData[pixelPos + 2];

      return (r_current === r_target && g_current === g_target && b_current === b_target);
    },
    colorPixel(pixelPos, imgData, r, g ,b) {
      imgData[pixelPos] = r;
      imgData[pixelPos + 1] = g;
      imgData[pixelPos + 2] = b;
      imgData[pixelPos + 3] = 255;
    },
    pixelAt(x, y) {
      return this.canvas.getImageData(x, y, 1, 1).data;
    },
    beginDrawing(e) {
      if(this.tool === 'colorSucker'){
        let color = this.pixelAt(e.offsetX, e.offsetY);
        this.colors = {rgba: {r: color[0], g: color[1], b: color[2], a: color[3]}}
      }
      else if (this.tool === 'filler') {
        this.make_history();
        this.fill_canvas(e.offsetX, e.offsetY);
      }else {
        this.make_history();
        this.x = e.offsetX;
        this.y = e.offsetY;
        this.drawLine(this.x, this.y, this.x, this.y)
        this.isDrawing = true;
      }
    },
    stopDrawing() {
      this.isDrawing = false;
    },
    ResetCords() {
      this.x = -1;
      this.y = -1;
    },
    PlusWidth() {
      this.lineWidth = this.lineWidth + 1;
    },
    MinusWidth() {
      this.lineWidth = this.lineWidth - 1;
      if (this.lineWidth < 1){
        this.lineWidth = 1;
      }
    },
    download() {
      dialog.showSaveDialog({filters: [
          { name: 'PNG', extensions: ['png'] },
          { name: 'JPEG', extensions: ['jpg'] }
        ], defaultPath: 'result.png'}).then((result) => {
        let url = this.canvas.canvas;
        let base64Data = null;
        console.log(result.filePath);
        if (result.filePath.includes('png')) {
          url = url.toDataURL('image/png');
          base64Data = url.replace(/^data:image\/png;base64,/, "");
        }else{
          let tempCanvas = document.createElement('canvas').getContext("2d");
          tempCanvas.canvas.width = this.canvas.canvas.width;
          tempCanvas.canvas.height = this.canvas.canvas.height;
          let imgData=this.canvas.getImageData(0,0,this.canvas.canvas.width, this.canvas.canvas.height);
          let data=imgData.data;
          for(let i=0;i<data.length;i+=4) {
            if (data[i + 3] === 0) {
              data[i] = 255;
              data[i + 1] = 255;
              data[i + 2] = 255;
              data[i + 3] = 255;
            }
          }
          tempCanvas.putImageData(imgData, 0, 0);
          url = tempCanvas.canvas.toDataURL('image/jpeg', 1.0);
          base64Data = url.replace(/^data:image\/jpeg;base64,/, "");
        }
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
          this.make_history();
          let img = new Image();
          img.src = result.filePaths;
          let parent = this;
          img.onload = function () {
            parent.canvas.canvas.width = this.naturalWidth;
            parent.canvas.canvas.height = this.naturalHeight;
            parent.canvas.drawImage(this, 0, 0);
          }
      }).catch((err) => {
        err
      });
    }
  }
}
</script>

<style>

input[type="number"] {
  -moz-appearance: textfield;
}
input[type="number"]::-webkit-inner-spin-button {
  display: none;
}


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
  background: #2A6478;
  position: fixed;
  border-bottom: 2px solid #000;
}

</style>