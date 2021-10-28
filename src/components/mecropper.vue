<template>

<el-button  @click="dialogVisible = true" class="covercss"
    ><img v-if="isImageUrl(Cover)" :src="imageUrl" class="avatar">
    <i v-else class="el-icon-plus avatar-uploader-icon"></i>
    </el-button
  >

  <el-dialog
    v-model="dialogVisible"
    title="上传封面"
    width="55%"
    :before-close="handleClose"
  >

    <el-row>
    <el-col :span="12"><div class="grid-content bg-purple">
       <div class="model" v-show="model" @click="model = false">
       <div class="model-show">
        <img :src="modelSrc" alt="">
      </div>
    </div>

    <div class="cut">
      <vue-cropper ref="cropper" :img="option.img" :output-size="option.size" :output-type="option.outputType" :info="true" :full="option.full" :fixed="fixed" :fixed-number="fixedNumber"
        :can-move="option.canMove" :can-move-box="option.canMoveBox" :fixed-box="option.fixedBox" :original="option.original"
        :auto-crop="option.autoCrop" :auto-crop-width="option.autoCropWidth" :auto-crop-height="option.autoCropHeight" :center-box="option.centerBox"
 @real-time="realTime" :high="option.high"
          @img-load="imgLoad" mode="cover" :max-img-size="option.max" @crop-moving="cropMoving" ></vue-cropper>
    </div>

      </div></el-col>
    <el-col :span="12"><div class="grid-content bg-purple-light">

      <div style="display:block; width: 100%;">
        <label class="c-item">
          <span>上传图片是否显示原始宽高 (针对大图 可以铺满)</span>
          <input type="checkbox" v-model="option.original">
          <span>original: {{ option.original}}</span>
        </label>

        <label class="c-item">
          <span>截图框固定大小</span>
          <input type="checkbox" v-model="option.fixedBox">
          <span>fixedBox: {{ option.fixedBox}}</span>
        </label>
        <label class="c-item">
          <span>是否输出原图比例的截图</span>
          <input type="checkbox" v-model="option.full">
          <span>full: {{ option.full}}</span>
        </label>
        <label class="c-item">
          <span>是否自动生成截图框</span>
          <input type="checkbox" v-model="option.autoCrop">
          <span>autoCrop: {{ option.autoCrop}}</span>
        </label>
        <label class="c-item">
          <span>是否根据dpr生成适合屏幕的高清图片</span>
          <input type="checkbox" v-model="option.high">
          <span>high: {{ option.high}}</span>
        </label>

      </div>
      </div></el-col>
  </el-row>

  <el-row>
    <el-col :span="24"><div class="grid-content bg-purple-dark">

          <div class="test-button">
      <button @click="changeImg" class="btn">初始化</button>
      <label class="btnx" for="uploads">选择封面</label>
      <input type="file" id="uploads" style="position:absolute; clip:rect(0 0 0 0);" accept="image/png, image/jpeg, image/gif, image/jpg"
        @change="uploadImg($event, 1)">
      <button @click="startCrop" v-if="!crap" class="btn">开始</button>
      <button @click="stopCrop" v-else class="btn">锁定</button>
      <button @click="clearCrop" class="btn">清空</button>
      <button @click="refreshCrop" class="btn">重置</button>
      <button @click="changeScale(1)" class="btn">+</button>
      <button @click="changeScale(-1)" class="btn">-</button>
      <button @click="rotateLeft" class="btn">向左翻</button>
      <button @click="rotateRight" class="btn">向右翻</button>

      <a @click="down('base64')" class="btnx">上传封面(base64)</a>

    </div></div></el-col>
  </el-row>

     </el-dialog>

</template>

<script>
// @ is an alias to /src

import 'vue-cropper/dist/index.css'
import { VueCropper } from 'vue-cropper'
import axios from 'axios'

export default {
  name: 'imgcropper',
  components: {
    VueCropper
  },
  props: ['Cover'],
  data: function () {
    return {
      imageUrl: null,
      dialogVisible: false,
      model: false,
      modelSrc: '',
      crap: false,
      previews: {},
      lists: [
        {
          img: 'https://avatars2.githubusercontent.com/u/15681693?s=460&v=4'
        }
      ],
      option: {
        img: 'https://avatars2.githubusercontent.com/u/15681693?s=460&v=4',
        size: 1,
        full: false,
        outputType: 'jpeg', // 裁剪生成图片的格式（jpeg || png || webp）
        canMove: true,
        fixedBox: false,
        original: false,
        canMoveBox: true,
        autoCrop: true,
        // 只有自动截图开启 宽度高度才生效
        autoCropWidth: 380,
        autoCropHeight: 214,
        centerBox: false,
        high: true,
        max: 99999
      },
      show: true,
      fixed: true,
      fixedNumber: [1, 1]

    }
  },
  watch: {
    Cover () {
      this.imageUrl = this.Cover
    }
  },
  methods: {
    isImageUrl (Cover) {
      if (Cover != null) {
        if (this.imageUrl == null) { this.imageUrl = Cover }

        return true
      } else { return false }
    },
    changeImg () {
      this.option.img = this.lists[~~(Math.random() * this.lists.length)].img
    },
    startCrop () {
      // start
      this.crap = true
      this.$refs.cropper.startCrop()
    },
    stopCrop () {
      //  stop
      this.crap = false
      this.$refs.cropper.stopCrop()
    },
    clearCrop () {
      // clear
      this.$refs.cropper.clearCrop()
    },
    refreshCrop () {
      // clear
      this.$refs.cropper.refresh()
    },
    changeScale (num) {
      num = num || 1
      this.$refs.cropper.changeScale(num)
    },
    rotateLeft () {
      this.$refs.cropper.rotateLeft()
    },
    rotateRight () {
      this.$refs.cropper.rotateRight()
    },
    finish (type) {
      // 输出
      // var test = window.open('about:blank')
      // test.document.body.innerHTML = '图片生成中..'
      if (type === 'blob') {
        this.$refs.cropper.getCropBlob((data) => {
          console.log(data)
          var img = window.URL.createObjectURL(data)
          this.model = true
          this.modelSrc = img
        })
      } else {
        this.$refs.cropper.getCropData((data) => {
          this.model = true
          this.modelSrc = data
        })
      }
    },
    // 实时预览函数
    realTime (data) {
      this.previews = data
      console.log(data)
    },

    finish2 (type) {
      this.$refs.cropper2.getCropData((data) => {
        this.model = true
        this.modelSrc = data
        console.log(type)
      })
    },
    finish3 (type) {
      this.$refs.cropper3.getCropData((data) => {
        this.model = true
        this.modelSrc = data
        console.log(type)
      })
    },
    down (type) {
      const _this = this
      // event.preventDefault()
      var aLink = document.createElement('a')
      aLink.download = 'demo'
      // 输出
      if (type === 'blob') {
        this.$refs.cropper.getCropBlob(async (data) => {
          const formData = new FormData()
          const nameImg = new Date().getTime() + '.jpg'
          formData.append('file', data, nameImg)
          const { data: res } = await axios.post(_this.$store.state.baseUrl + '/api/Common/UpFile/UploadOneFile', formData)
          if (res.returnCode === 200) {
            _this.$message({
              message: res.returnMsg,
              type: 'success'
            })
            _this.$emit('uploadImgSuccess', res.data.file)
            this.imageUrl = res.data.file
            this.dialogVisible = false
          }
        })
      } else {
        this.$refs.cropper.getCropData((data) => {
          _this.$emit('uploadImgSuccess', data)
          this.dialogVisible = false
        })
      }
    },

    uploadImg (e, num) {
      // 上传图片
      // this.option.img
      var file = e.target.files[0]
      if (!/\.(gif|jpg|jpeg|png|bmp|GIF|JPG|PNG)$/.test(e.target.value)) {
        alert('图片类型必须是.gif,jpeg,jpg,png,bmp中的一种')
        return false
      }
      var reader = new FileReader()
      reader.onload = (e) => {
        let data
        if (typeof e.target.result === 'object') {
          // 把Array Buffer转化为blob 如果是base64不需要
          data = window.URL.createObjectURL(new Blob([e.target.result]))
        } else {
          data = e.target.result
        }
        if (num === 1) {
          this.option.img = data
        } else if (num === 2) {
          this.example2.img = data
        }
      }
      // 转化为base64
      // reader.readAsDataURL(file)
      // 转化为blob
      reader.readAsArrayBuffer(file)
    },
    imgLoad (msg) {
      console.log(msg)
    },
    cropMoving (data) {
      console.log(data, '截图框当前坐标')
    }
  }
}
</script>

<style scoped>
*{
        margin: 0;
        padding: 0;
      }

      .cut {
        width: 480px;
        height: 480px;
        margin: 30px auto;
      }

      .c-item {
        max-width: 800px;
        margin: 10px auto;
        margin-top: 20px;
      }

      .content {
        margin: auto;
        max-width: 1200px;
        margin-bottom: 100px;
      }

      .test-button {
        display: flex;
        flex-wrap: wrap;
        align-content: center;
        justify-content: center;
      }

      .btn {
        display: inline-block;
        line-height: 1;
        white-space: nowrap;
        cursor: pointer;
        background: #fff;
        border: 1px solid #c0ccda;
        color: #1f2d3d;
        text-align: center;
        box-sizing: border-box;
        outline: none;
        margin:20px 10px 0px 0px;
        padding: 9px 15px;
        font-size: 14px;
        border-radius: 4px;
        color: #fff;
        background-color: #50bfff;
        border-color: #50bfff;
        transition: all .2s ease;
        text-decoration: none;
        user-select: none;
      }
        .btnx {
        display: inline-block;
        line-height: 1;
        white-space: nowrap;
        cursor: pointer;
        background: #fff;
        border: 1px solid #c0ccda;
        color: #1f2d3d;
        text-align: center;
        box-sizing: border-box;
        outline: none;
        margin:20px 10px 0px 0px;
        padding: 9px 15px;
        font-size: 14px;
        border-radius: 4px;
        color: #fff;
        background-color: #67C23A;
        border-color: #67C23A;
        transition: all .2s ease;
        text-decoration: none;
        user-select: none;
      }

      .des {
        line-height: 30px;
      }

      code.language-html {
        padding: 10px 20px;
        margin: 10px 0px;
        display: block;
        background-color: #333;
        color: #fff;
        overflow-x: auto;
        font-family: Consolas, Monaco, Droid, Sans, Mono, Source, Code, Pro, Menlo, Lucida, Sans, Type, Writer, Ubuntu, Mono;
        border-radius: 5px;
        white-space: pre;
      }

      .show-info {
        margin-bottom: 50px;
      }

      .show-info h2 {
        line-height: 50px;
      }

      /*.title, .title:hover, .title-focus, .title:visited {
        color: black;
      }*/

      .title {
        display: block;
        text-decoration: none;
        text-align: center;
        line-height: 1.5;
        margin: 20px 0px;
        background-image: -webkit-linear-gradient(left,#3498db,#f47920 10%,#d71345 20%,#f7acbc 30%,#ffd400 40%,#3498db 50%,#f47920 60%,#d71345 70%,#f7acbc 80%,#ffd400 90%,#3498db);
        color: transparent;

        background-size: 200% 100%;
        animation: slide 5s infinite linear;
        font-size: 40px;
      }

      .test {
        height: 500px;
      }

      .model {
        position: fixed;
        z-index: 10;
        width: 100vw;
        height: 100vh;
        overflow: auto;
        top: 0;
        left: 0;
        background: rgba(0, 0, 0, 0.8);
      }

      .model-show {
        display: flex;
        justify-content: center;
        align-items: center;
        width: 100vw;
        height: 100vh;
      }

      .model img {
        display: block;
        margin: auto;
        max-width: 80%;
        user-select: none;
        background-position: 0px 0px, 10px 10px;
        background-size: 20px 20px;
        background-image: linear-gradient(45deg, #eee 25%, transparent 25%, transparent 75%, #eee 75%, #eee 100%),linear-gradient(45deg, #eee 25%, white 25%, white 75%, #eee 75%, #eee 100%);
      }

      .c-item {
        display: block;
        user-select: none;
      }

      @keyframes slide {
        0%  {
          background-position: 0 0;
        }
        100% {
          background-position: -100% 0;
        }
      }
 .avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  .avatar-uploader .el-upload:hover {
    border-color: #409EFF;
  }
  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 90px;
    height: 90px;
    line-height: 90px;
    text-align: center;
  }
  .avatar {
    width: 90px;
    height: 90px;
    display: block;
  }
  .covercss{
    margin: 10px;
  }
</style>
