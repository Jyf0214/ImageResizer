<template>
  <el-container id="app">
    <el-header> header</el-header>
    <el-container>
      <el-aside>
        <h2>输出尺寸：</h2>
        <el-input size="small" v-model="settings.width">
          <template slot="prepend">宽度：</template>
          <template slot="append">px</template>
        </el-input>

        <el-input size="small" v-model="settings.height">
          <template slot="prepend">高度：</template>
          <template slot="append">px</template>
        </el-input>

        <h2>文件大小：</h2>
        <el-input size="small">
          <template slot="prepend">Max：</template>
          <template slot="append">KB</template>
        </el-input>

        <el-input size="small">
          <template slot="prepend">Min：</template>
          <template slot="append">KB</template>
        </el-input>
        <h2>缩放和移动：</h2>

        <div v-show="isOk" class="btns">
          <el-button type="primary" size="medium" icon="el-icon-plus" circle @click="resize('big')"></el-button>
          <el-button type="primary" size="medium" icon="el-icon-minus" circle @click="resize('small')"></el-button>

          <el-button type="warning" size="medium" icon="el-icon-arrow-left" circle
                     @click="setImg('left')"></el-button>
          <el-button type="warning" size="medium" icon="el-icon-arrow-right" circle
                     @click="setImg('right')"></el-button>
          <el-button type="warning" size="medium" icon="el-icon-arrow-down" circle
                     @click="setImg('down')"></el-button>
          <el-button type="warning" size="medium" icon="el-icon-arrow-up" circle @click="setImg('up')"></el-button>

        </div>

      </el-aside>
      <el-main>

        <div class="img_box">
          <div class="btn_addimg" v-if="!isOk">
            <i class="el-icon-plus"></i>
            <input type="file" accept="image/png,image/jpeg" @change="handleFile">
          </div>
          <canvas v-show="isOk" ref="canvas"></canvas>
        </div>

      </el-main>
    </el-container>
    <el-footer> designed by Mirror</el-footer>
  </el-container>

  <!--<div class="img_box">-->
  <!--<i ref="btn_addimg" class="fa fa-plus-square-o" aria-hidden="true"></i>-->
  <!--</div>-->
</template>

<script>
  export default {
    name: 'App',
    data: function () {
      return {
        settings: {
          width: 355,
          height: 441,
          maxSize: 20,
          minSize: 9
        },
        rawImgFile:undefined,
        canvasCtx: undefined,
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
        zoomStep: 0.1
      }
    },
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
          let img = new Image();
          img.src = e.currentTarget.result;
          img.onload = function () {

            _this.imgObj = img;
            _this.imgObj_w = img.width;
            _this.imgObj_h = img.height;


            _this.canvas_img_scale = _this.settings.width/_this.settings.height > _this.imgObj.width/_this.imgObj.height ? _this.settings.width/_this.imgObj.width : _this.settings.height/_this.imgObj.height ;

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

        this.set_sw_sy();

        this.canvasCtx.drawImage(image, this.s.x, this.s.y, this.s.w, this.s.h, this.d.x, this.d.y, this.d.w, this.d.h);
      },
      clearCtx:function(){
        let _this = this ;
        _this.canvasCtx.clearRect(0, 0 ,_this.settings.width , _this.settings.height )
      },
      resize:function (p) {
        let _this = this;
        switch (p){
          case "big":
            console.log('big 图像');
            _this.clearCtx();
            _this.canvas_img_scale = _this.canvas_img_scale*1.1;
            _this.drawImg(_this.imgObj);
            break;

          case"small":
            console.log('small 图像');
            _this.clearCtx();
            _this.canvas_img_scale = _this.canvas_img_scale/1.1;
            _this.drawImg(_this.imgObj);
            break;
          case "left":
            console.log( "left 图像");
            //_this.clearCtx();


            break;

          case "right":
            console.log( "right 图像");
            //_this.clearCtx();

            break;

          default:
            console.log( "resize err： 未知的图像处理参数");
            break;
        }

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

      if(this.isOk){
        this.readFile_draw( this.rawImgFile );
      }else{
        console.log( 'updated: can not handle file and draw' )
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
      height: 10%;
      background: aquamarine;
    }
    > .el-container {
      > .el-aside {
        width: 250px !important;
        background: salmon;
        padding: 50px 20px;

        .el-input {
          margin-top: 20px;
        }
      }
      > .el-main {
        position: relative;
        background: #1b112a;
        .img_box {
          position: absolute;
          display: block;
          z-index: 900;
          top: 50%;
          left: 50%;
          transform: translate(-50%, -50%);
          background: greenyellow;
          > .btn_addimg {
            display: block;
            width: 200px;
            height: 200px;
            border: 1px dashed #ccc;
            position: relative;
            > input {
              position: absolute;
              z-index: 1000;
              display: block;
              width: 100%;
              height: 100%;
              /*visibility: hidden;*/
              /*opacity: 0;*/
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
              color: seagreen;
            }

            &:hover {
              border: 1px dotted slateblue;
              > i {
                text-shadow: 1px 2px 2px #ccc;
              }
            }
          }
          > canvas{
            display: block;
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
      background: sandybrown;
    }
  }
</style>
