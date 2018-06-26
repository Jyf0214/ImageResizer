<template>
  <el-container id="app">
    <el-header> <h1>ImageResizer</h1> </el-header>
    <el-container>
      <el-aside>
        <h2>第一步：输出尺寸设定</h2>
        <div class="box">
          <el-input size="small" v-model="settings.width">
            <template slot="prepend">宽度：</template>
            <template slot="append">px</template>
          </el-input>
          <el-input size="small" v-model="settings.height">
            <template slot="prepend">高度：</template>
            <template slot="append">px</template>
          </el-input>
        </div>

        <h2>第二步：上传本地图片</h2>
        <div class="box">
          <p>
            点击右侧 "+" 按钮，添加本地图片。支持的格式 PNG/JPEG 。
          </p>
        </div>

        <h2>第三步：调整图片位置</h2>
        <div class="box set_btns">
          <el-button type="primary" size="small" icon="el-icon-plus" circle @click="resize('big')"></el-button>
          <el-button type="primary" size="small" icon="el-icon-minus" circle @click="resize('small')"></el-button>
          <el-button type="warning" size="small" icon="el-icon-arrow-left" circle
                     @click="resize('left')"></el-button>
          <el-button type="warning" size="small" icon="el-icon-arrow-right" circle
                     @click="resize('right')"></el-button>
          <el-button type="warning" size="small" icon="el-icon-arrow-down" circle
                     @click="resize('down')"></el-button>
          <el-button type="warning" size="small" icon="el-icon-arrow-up" circle @click="resize('up')"></el-button>
        </div>


        <h2>第四步：输出质量调整</h2>
        <div class="box">
          <el-slider v-model="slider" @change="sliderChange"></el-slider>
          <p class="filesizeTip"> 文件大小约为
            <span>{{tempImgSize}}</span>
          </p>
        </div>

        <h2>第五步：生成图片</h2>
        <div class="box">
          <button @click="saveCanvasImg()">生成图片</button>
          <p>
            提示：点击上方按钮，相应尺寸和大小的文件会在浏览器中生成，请【右击】->【另存为...】
          </p>
        </div>


      </el-aside>
      <el-main>

        <div class="img_box">
          <div class="btn_addimg" v-if="!isOk">
            <i class="el-icon-plus"></i>
            <input type="file" accept="image/png,image/jpeg" @change="handleFile">
          </div>
          <canvas v-show="isOk" ref="canvas"></canvas>
          <span v-if="isOk" class="btn_reset" @click="refreshWindow">
            <i class="el-icon-close"></i>
          </span>
        </div>


      </el-main>
    </el-container>
    <el-footer> ©Mirror </el-footer>
  </el-container>
</template>

<script>
  export default {
    name: 'App',
    data: function () {
      return {
        settings: {
          width: 358,
          height: 441,
          maxSize: 20,
          minSize: 9
        },
        rawImgFile: undefined,
        base64Img: undefined,
        canvasCtx: undefined,
        slider: 50,
        imgObj: null,
        imgObj_h: undefined,
        imgObj_w: undefined,
        isOk: false,
        s: {
          x: 0,
          y: 0,
          w: 0,
          h: 0
        },
        d: {
          x: 0,
          y: 0,
          w: 0,
          h: 0,
        },
        canvas_img_scale: undefined,
        tempCanvasImg: undefined,
        tempImgSize: '--',
        imgHref: undefined
      }
    },
    computed: {},
    methods: {
      handleFile: function (e) {
        //console.log(e )
        var _this = this;

        if (e.target.files[0]) {
          _this.isOk = true;
          _this.rawImgFile = e.target.files[0];
          _this.readFile_draw(e.target.files[0]);
        }

      },
      readFile_draw: function (file) {
        if (!file) {
          console.log('Fn: readFile  error: lost params')
          return
        }
        let _this = this;

        let reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = function (e) {

          _this.base64Img = e.currentTarget.result;

          let img = new Image();
          img.src = e.currentTarget.result;
          img.onload = function () {

            _this.imgObj = img;
            _this.imgObj_w = img.width;
            _this.imgObj_h = img.height;


            _this.canvas_img_scale = _this.settings.width / _this.settings.height > _this.imgObj.width / _this.imgObj.height ? _this.settings.width / _this.imgObj.width : _this.settings.height / _this.imgObj.height;

            _this.s.x = 0;
            _this.s.y = 0;

            _this.d.x = 0;
            _this.d.y = 0;
            _this.d.w = _this.$refs.canvas.width;
            _this.d.h = _this.$refs.canvas.height;

            _this.drawImg(_this.imgObj)

          }
        }
      },
      set_sw_sy: function () {
        let _this = this;
        _this.s.w = _this.settings.width / _this.canvas_img_scale;
        _this.s.h = _this.settings.height / _this.canvas_img_scale;
      },
      drawImg: function (image) {
        this.clearCtx();
        this.set_sw_sy();
        this.canvasCtx.drawImage(image, this.s.x, this.s.y, this.s.w, this.s.h, this.d.x, this.d.y, this.d.w, this.d.h);
        this.sliderChange(this.slider)
      },
      clearCtx: function () {
        let _this = this;
        _this.canvasCtx.clearRect(0, 0, _this.settings.width, _this.settings.height)
      },
      resize: function (p) {
        let _this = this;
        if (!_this.isOk || !_this.imgObj) {
          _this.$message.error('请在右侧先上传一张本地图片（.png/.jpg）');
          return
        }
        switch (p) {
          case "big":
            console.log('big 图像');
            _this.clearCtx();
            _this.canvas_img_scale = _this.canvas_img_scale * 1.05;
            _this.drawImg(_this.imgObj);
            break;

          case"small":
            console.log('small 图像');
            _this.clearCtx();
            _this.canvas_img_scale = _this.canvas_img_scale / 1.05;
            _this.drawImg(_this.imgObj);
            break;
          case "left":
            console.log("left 图像");
            _this.clearCtx();
            _this.s.x = _this.s.x + Math.ceil( _this.settings.width/100 );
            _this.drawImg(_this.imgObj);
            break;

          case "right":
            console.log("right 图像");
            _this.clearCtx();
            _this.s.x = _this.s.x - Math.ceil( _this.settings.width/100 );
            _this.drawImg(_this.imgObj);
            break;

          case "up":
            console.log("up 图像");
            _this.clearCtx();
            _this.s.y = _this.s.y + 2;
            _this.drawImg(_this.imgObj);
            break;
          case "down":
            console.log("up 图像");
            _this.clearCtx();
            _this.s.y = _this.s.y - 2;
            _this.drawImg(_this.imgObj);
            break;

          default:
            console.log("resize err： 未知的图像处理参数");
            break;
        }

      },
      sliderChange: function (value) {
        //console.log("e", e)
        if (this.isOk) {
          let base64Img = this.$refs.canvas.toDataURL("image/jpeg", value / 100);
          this.tempCanvasImg = base64Img;

          this.tempImgSize = ( (base64Img.length / 1024) / 1.335 ).toFixed(2) + "kb";
        } else {
          this.$message.error('请在右侧先上传一张本地图片（.png/.jpg）');
          return
        }

      },
      saveCanvasImg: function () {
        let _this = this;
        if (!_this.tempCanvasImg) {
          this.$message.error('请在右侧窗口上传一张本地图片（.png/.jpg）');
          return
        }


        document.write(`<img src=\"${_this.tempCanvasImg}\"/>`);
      },
      refreshWindow:function () {
        window.location.reload()
      }
    },
    mounted: function () {
      console.log("mounted:>>>>>>>>>>>>>>>>>>>>>>>>")
      console.log(`canvas尺寸: w = ${this.settings.width} ; h = ${this.settings.height} `)
      this.$refs.canvas.width = this.settings.width;
      this.$refs.canvas.height = this.settings.height;
      this.canvasCtx = this.$refs.canvas.getContext("2d");

      console.log("mounted:<<<<<<<<<<<<<<<<<<<<<<<")
    },
    beforeUpdate: function () {

    },
    updated: function () {
      console.log("updated:>>>>>>>>>>>>>>>>>>>>>>>>")
      console.log(`canvas尺寸: w = ${this.settings.width} ; h = ${this.settings.height} `)
      this.$refs.canvas.width = this.settings.width;
      this.$refs.canvas.height = this.settings.height;
      this.d.w = this.settings.width;
      this.d.h = this.settings.height;

      let _this = this;
      if (_this.isOk && _this.imgObj) {
        _this.drawImg(_this.imgObj);
      } else {
        console.log('updated: can not handle file and draw')
      }

      console.log("updated:<<<<<<<<<<<<<<<<<<<<<<< \t\t")


    }

  }
</script>

<style lang="less">
  #app {
    width: 100%;
    height: 100%;
    overflow: hidden;
    background: wheat;
    > .el-header {
      @h:50px;
      height: @h !important;
      line-height: @h;
      background: #2d2d2d;
      text-align: center;
      >h1{
        height: @h;
        line-height: @h;
        font-size: 22px;
        padding: 0;
        margin: 0;
        color: rgba(255,255,255,0.5);
        text-shadow: -1px -1px #eeeeee;
      }

    }
    > .el-container {
      > .el-aside {
        width: 350px !important;
        background: #333;
        padding: 20px;
        h2 {
          font-size: 16px;
          color: #ddd;
          border-bottom: 1px solid rgba(255, 255, 255, 0.15);
          background: rgba(255, 255, 255, 0.1);
          margin-bottom: 0;
          padding: 10px;
          margin-top: 20px;
        }
        h2:nth-child(1) {
          margin-top: 0;
        }
        .box {
          padding: 15px 10px;
          background: rgba(255, 255, 255, 0.2);
          .filesizeTip {
            font-size: 12px;
            color: #cdcdcd;
            font-weight: lighter;
            > span {
              font-weight: normal;
              color: salmon !important;
            }
          }
          >p{
            text-indent: 2em;
            font-size: 12px;
            line-height: 1.6;
            color: deepskyblue;
            font-weight: lighter;
          }
        }
        .set_btns {
          text-align: center;
        }
      }
      > .el-main {
        position: relative;
        background: rgba(11, 26, 48, 0.58);
        .img_box {
          position: absolute;
          display: block;
          z-index: 900;
          top: 50%;
          left: 50%;
          transform: translate(-50%, -50%);
          background: #555;
          > .btn_addimg {
            display: block;
            width: 200px;
            height: 200px;
            position: relative;
            > input {
              position: absolute;
              z-index: 1000;
              display: block;
              width: 100%;
              height: 100%;
              opacity: 0;
            }
            > i {
              position: absolute;
              z-index: 999;
              top: 50%;
              left: 50%;
              transform: translate(-50%, -50%);
              display: block;
              width: 40px;
              height: 40px;
              font-size: 40px;
              color: #fff;
            }

            &:hover {
              border: 1px dashed #333;
              > i {
                text-shadow: 1px 2px 2px #ccc;
              }
            }
          }
          > canvas {
            display: block;
          }
          > .btn_reset{
            @h1:50px;
            @br:@h1/2;
            display: block;
            position: absolute;
            width: @h1;
            height: @h1;
            background: rgba(255,255,255,0.1);
            top:   8-@h1 ;
            right: 8-@h1;
            z-index: 700;
            border-radius: @br;
            >i{
              @h2:15px;
              position: absolute;
              left: (@h1 - @h2)/2;
              top: (@h1 - @h2)/2;
              color: #aaa;
              font-size: @h2;
              font-weight: lighter;
            }
            &:hover{
              background: rgba(255,255,255,0.7);
              >i{
                color: salmon;
                font-weight: bolder;
              }
            }
          }

          /*>.btns {*/
          /*z-index: 1100;*/
          /*position: absolute;*/
          /*display: block;*/
          /*width: 100%;*/
          /*text-align: center;*/
          /*bottom: -50px;*/
          /*}*/
        }
      }

    }
    > .el-footer {
      @h: 25px;
      height: @h !important;
      line-height: @h;
      background: #2d2d2d;
      color: rgba(96, 96, 96, 9);
      text-align: center;
      font-size: 14px;
    }
  }
</style>
