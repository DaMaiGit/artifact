<template>
  <v-app class="main-section">
    <div class="content-section">
      <img src="./assets/logo.jpg" class="logo">
      <h1 align="center" class="title-text">快来试试你的<br>口红色号吧</h1>
      <div class="image-setcion">
        <div style="position: relative;width: 310px;height: 310px">
          <v-img
                  ref="peopleImg"
                  :src="fileUrl"
                  height="310"
                  width="310"
                  style="position: absolute;left: 0px;top: 0px;background-color: #ed455c"
          >
          </v-img>

          <v-img
                  :src="heartURL"
                  height="310"
                  width="310"
                  style="position: absolute;left: 0px;top: 0px;"
          >
          </v-img>
        </div>
      </div>
      <input type="file" accept="image/*" class="img-input" @change="getImageUrl" ref="fileInput">
      <div class="recognition-btn">
        <button @click="clickBtn" class="btn">{{btnText}}</button>
      </div>
      <div class="res-section" ref="resSection">
        <div class="res-color" ref="resColor"></div>
        <span class="res-text" ref="resText"></span>
      </div>
    </div>
    <v-dialog
            v-model="dialog"
            max-width="290"
    >
      <v-card>
        <v-card-title class="headline">检测失败</v-card-title>

        <v-card-text>
          请确认已经上传女神的正面单人照片
        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>

          <v-btn
                  color="green darken-1"
                  text
                  @click="dialog = false"
          >
            取消
          </v-btn>

        </v-card-actions>
      </v-card>
    </v-dialog>


  </v-app>
</template>

<script>
import * as faceApi from "face-api.js"
import * as exif from "exif-js"
export default {
  name: 'App',
  components: {
  },
  created(){
    this.initModel()
  },
  data: () => ({
    fileUrl:"",
    mouthColors:[],
    dialog: false,
    btnText:"点击上传图片",
    heartURL:require("./assets/heart.png")
  }),
  methods:{
    //获取图片文件
    getImageUrl:async function(){
      let file = this.$refs.fileInput.files[0]
      if (file!=undefined && file!=null){
        this.fileUrl = window.URL.createObjectURL(file)
        if (!this.isPC()){
          let orientation;
          let ctxThis = this
          exif.getData(file,function(){
            orientation = exif.getTag(this,"Orientation")
            let reader = new FileReader()
            reader.readAsDataURL(file)
            reader.onload = async (e)=>{
              console.log(e.target)
              if (orientation==1 || orientation==undefined){
                this.fileUrl = window.URL.createObjectURL(file)
                ctxThis.startRecognition()
                return
              }
              let uploadBase64 = new Image();
              uploadBase64.src = e.target.result
              uploadBase64.onload =async function () {
                let canvas = document.createElement("canvas")
                canvas.width = this.width
                canvas.height = this.height
                let ctx = canvas.getContext("2d")
                let width = this.width
                let height = this.height
                ctx.drawImage(uploadBase64,0,0,width,height)
                if (orientation!="" &&orientation!=1) {
                  switch (orientation) {
                    case 6:
                      console.log("顺时针旋转270度");
                      ctxThis.rotateImg(uploadBase64, "left", canvas)
                      break;
                    case 8:
                      console.log("顺时针旋转90度");
                      ctxThis.rotateImg(uploadBase64, "right", canvas);
                      break;
                    case 3:
                      console.log("顺时针旋转180度");
                      ctxThis.rotateImg(uploadBase64, "horizen", canvas);
                      break;
                  }
                  let base64 = canvas.toDataURL(file.type, 1);
                  let newBlob = ctxThis.convertBase64UrlToBlob(base64, file.type);
                  ctxThis.fileUrl =await window.URL.createObjectURL(newBlob)
                }
                ctxThis.startRecognition(canvas)
              }

            }
          })
        } else {
          this.fileUrl = window.URL.createObjectURL(file)
          this.startRecognition()
        }

      }
    },
    /**
     * 初始化模型
     **/
    initModel:async function () {
      try {
        await faceApi.nets.faceLandmark68Net.load('/weights')
        await faceApi.nets.ssdMobilenetv1.load('/weights')
        console.log("finish")
      }catch (e) {
        this.dialog = true
        this.btnText = "点击上传图片"
      }
    },
    clickBtn:function(){
      switch (this.btnText) {
        case "点击上传图片":
          this.$refs.fileInput.click()
          this.$refs.resSection.style.visibility="hidden"
          break
      }
    },
    /**
     * 开始识别嘴唇
     **/
    startRecognition:async function (canvas) {
      try {
        this.btnText = "识别中..."
        if (canvas==undefined || canvas==null){
          canvas =await faceApi.createCanvasFromMedia(this.$refs.peopleImg.image)
        }
        const landmarks = await faceApi.detectSingleFace(canvas).withFaceLandmarks()
        // faceApi.draw.drawFaceLandmarks(canvas,landmarks.landmarks)
        // canvas.toBlob(function(blob){
        //   console.log(window.URL.createObjectURL(blob))
        // })
        let mouthPoint = landmarks.landmarks.getMouth()
        this.getMouthColor(canvas,mouthPoint)
        let lipsticks = require('./lipstick')

        let compareVal = null;
        let lipstick = null ;
        let compareMouthColor = null;
        for (let i = 0;i<this.mouthColors.length;i++){
          let mouthColor = this.mouthColors[i]
          for (let branchItem of lipsticks.brands) {
            for (let seriesItem of branchItem.series) {
              for (let lipstickItem of seriesItem.lipsticks) {
                let color = this.hex2rgb(lipstickItem.color.toLowerCase())
                let compareTem = this.compareColor(mouthColor[0],mouthColor[1],mouthColor[2],color.r,color.g,color.b)
                if (compareVal == null){
                  compareVal = compareTem
                  lipstick = `${branchItem.name}${seriesItem.name}${lipstickItem.name} #${lipstickItem.id}`
                  compareMouthColor = mouthColor
                } else if(compareTem<compareVal){
                  compareVal = compareTem
                  lipstick = `${branchItem.name}${seriesItem.name}${lipstickItem.name} #${lipstickItem.id}`
                  compareMouthColor = mouthColor
                }
              }
            }
          }
        }
        this.addResult(compareMouthColor[0],compareMouthColor[1],compareMouthColor[2],compareMouthColor[3],lipstick)
      }catch (e) {
        this.dialog = true
        this.btnText = "点击上传图片"
        console.log(e)
      }

    },
    /**
     * 添加结果dom元素
     **/
    addResult:function(r,g,b,a,text){
      this.$refs.resSection.style.visibility="visible"
      this.$refs.resColor.style.background = `rgba(${r},${g},${b},${a})`
      this.$refs.resText.innerText = text
      this.btnText = "点击上传图片"
    },
    rgba2hex:function(r,g,b){
      if (r > 255 || g > 255 || b > 255)
        throw "Invalid color component";
      return "#" + ((1 << 24) + (r << 16) + (g << 8) + b).toString(16).slice(1).toUpperCase();
    },
    //hex to rgb
    hex2rgb:function(hex){
      let m = hex.match(/^#?([\da-f]{2})([\da-f]{2})([\da-f]{2})$/i);
      return {
        r: parseInt(m[1], 16),
        g: parseInt(m[2], 16),
        b: parseInt(m[3], 16)
      };
    },
    /**
     * 颜色差距计算
     * @param r1
     * @param g1
     * @param b1
     * @param r2
     * @param g2
     * @param b2
     * @returns {number}
     */
    compareColor:function(r1,g1,b1,r2,g2,b2){
      let r = Math.pow((r1-r2),2)
      let g = Math.pow((g1-g2),2)
      let b = Math.pow((b1-b2),2)
      return Math.sqrt((r+g+b))
    },
    rotateImg:function(img, direction,canvas){
      console.log("开始旋转图片");
      //图片旋转4次后回到原方向
      if (img == null) return;
      var height = img.height;
      var width = img.width;
      var step = 2;

      if (direction == "right") {
        step++;
      } else if (direction == "left") {
        step--;
      } else if (direction == "horizen") {
        step = 2; //不处理
      }
      //旋转角度以弧度值为参数
      var degree = step * 90 * Math.PI / 180;
      var ctx = canvas.getContext("2d");
      switch (step) {
        case 0:
          canvas.width = width;
          canvas.height = height;
          ctx.drawImage(img, 0, 0);
          break;
        case 1:
          canvas.width = height;
          canvas.height = width;
          ctx.rotate(degree);
          ctx.drawImage(img, 0, -height);
          break;
        case 2:
          canvas.width = width;
          canvas.height = height;
          ctx.rotate(degree);
          ctx.drawImage(img, -width, -height);
          break;
        case 3:
          canvas.width = height;
          canvas.height = width;
          ctx.rotate(degree);
          ctx.drawImage(img, -width, 0);
          break;
      }
      console.log("结束旋转图片");
    },
    isPC:function(){
      let userAgentInfo = navigator.userAgent
      let Agents = ["Android", "iPhone","SymbianOS",
        "Windows Phone","iPad", "iPod"]
      let flag = true
      for (var v = 0; v < Agents.length; v++) {
        if (userAgentInfo.indexOf(Agents[v]) > 0) {
          flag = false
          break
        }
      }
      return flag
    },
    convertBase64UrlToBlob:function (urlData) {
      let bytes=window.atob(urlData.split(',')[1]);        //去掉url的头，并转换为byte

      //处理异常,将ascii码小于0的转换为大于0
      let ab = new ArrayBuffer(bytes.length);
      let ia = new Uint8Array(ab);
      for (var i = 0; i < bytes.length; i++) {
        ia[i] = bytes.charCodeAt(i);
      }

      return new Blob( [ab] , {type : 'image/png'});
    },
    /**
     * 上口红
     * @param image
     * @param mouthPoint
     * @param r
     * @param g
     * @param b
     * @returns {null}
     */
    putLipstick:function(image,mouthPoint,r,g,b){
      if (image instanceof Image){
        let canvas = faceApi.createCanvasFromMedia(image)
        let ctx = canvas.getContext("2d")
        //画上嘴唇
        for (let i =0 ;i<7;i++){
          if (i==0){
            ctx.moveTo(mouthPoint[i]["_x"],mouthPoint[i]["_y"]);
          } else {
            ctx.lineTo(mouthPoint[i]["_x"],mouthPoint[i]["_y"]);
          }
        }
        for (let i =16 ;i>=14;i--){
          ctx.lineTo(mouthPoint[i]["_x"],mouthPoint[i]["_y"]);
        }
        ctx.closePath();
        ctx.fillStyle = `rgb(${r},${g},${b})`;
        ctx.fill();

        //画下嘴唇
        for (let i =6 ;i<=12;i++){
          if (i==6){
            ctx.moveTo(mouthPoint[i]["_x"],mouthPoint[i]["_y"]);
          } else {
            ctx.lineTo(mouthPoint[i]["_x"],mouthPoint[i]["_y"]);
          }
        }
        for (let i =19 ;i>=17;i--){
          ctx.lineTo(mouthPoint[i]["_x"],mouthPoint[i]["_y"]);
        }
        ctx.closePath();
        ctx.fillStyle = `rgb(${r},${g},${b})`;
        ctx.fill();

        canvas.toBlob(function(blob){
          // 涂口红的效果
          console.log(URL.createObjectURL(blob))
          return URL.createObjectURL(blob)
        })
      }
      return null
    },
    /**
     * 获取嘴部20个特征点的颜色
     * @param canvans
     * @param mouthPoint
     */
    getMouthColor:function (canvans,mouthPoint) {
      let context = canvans.getContext("2d")
      for (let i =0;i<mouthPoint.length;i++){
        let data = context.getImageData(mouthPoint[i]["_x"],mouthPoint[i]["_y"],1,1)
        this.mouthColors[i] = data.data
      }
    }
  }

};
</script>

<style lang="stylus" scoped="main">
.main-section
  background-image :url("~@/assets/bg.png") !important
  background-size: 100% !important
  background-repeat:no-repeat !important
  background-color #ffdfe4 !important
  .content-section
    display flex
    flex-direction column
    justify-content center
    .logo
      width 6rem
      position absolute
      right 0.5rem
      top 0.5rem
      @media (min-device-pixel-ratio:3),(-webkit-min-device-pixel-ratio:3)
        width 3rem
        position absolute
        right 0.5rem
        top 0.5rem
    .title-text
      color: #ed455c
      margin 2.5rem auto
      font-size 2rem
      font-weight bold
    .image-setcion
      display flex
      justify-content center
      justify-items center
      margin-top 4rem
    .img-input
      visibility hidden
    .recognition-btn
      display flex
      flex-direction row
      justify-content center
      margin 1rem
      .btn
        background-image :url("~@/assets/bg_btn.png") !important
        background-size: 100% 100% !important
        background-repeat:no-repeat !important
        padding 0.6rem 1rem
        color white
        box-shadow 3px 5px 5px #ed455c;
        border-radius 17px
    .res-section
      display flex
      flex-direction row
      align-items center
      justify-content center
      visibility hidden
      .res-color
        width 1.5rem
        height 1.5rem
        box-shadow 1px 1px 1px #888888
        border-radius 17px
        margin-right 0.5rem

</style>
