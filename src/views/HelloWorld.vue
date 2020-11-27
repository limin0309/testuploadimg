<template>
  <div>
    <el-dialog
      class="update-dialog"
      width="700px"
      :visible.sync="outerVisible"
      :close-on-click-modal="false"
    >
      <p slot="title" class="titl">
        车源照片 <span>按住拖动可调整顺序，点击后可涂抹、旋转图片</span>
      </p>
      <p class="desc">首图要求正面或45度角车源全貌图。最少上传4张，最多15张。图片格式包括jpg、png和jpeg，每张最大10M。传满15张可大幅提高网站排名。 <span>拍摄示例</span></p>
      <p class="slid"><span>{{ value.length }}</span>/15</p>
      <draggable
        v-loading="loadingShow"
        :list="value"
        tag="ul"
        class="img-list"
        v-bind="dragOptions"
        draggable=".item"
      >
        <li v-for="(item,index) in value" :key="item" class="item">
          <img :src="item" alt="" @click="editImg(index)">
          <i class="el-icon-circle-close" @click="deletImgList(index)" />
        </li>
        <li v-show="value.length < 15" class="img-li">
          <el-upload
            class="upload-class"
            :action="uploadUrl"
            :headers="headers"
            :before-upload="uploadBefore"
            :on-success="onSucessUpload"
            multiple
            :on-exceed="onExceedupload"
            :limit="limitNumb"
            :show-file-list="false"
          >
            <i class="el-icon-plus" /> 从电脑上传图片
          </el-upload>
        </li>
      </draggable>
      <el-dialog
        width="700px"
        :visible.sync="innerVisible"
        append-to-body
        :before-close="beforeClose"
        center
      >
        <p slot="title" class="titl">编辑照片</p>
        <div v-loading="loadingCrop" class="edit-img">
          <vueCropper
            ref="cropper"
            :img="editImgUrl"
            :info="false"
            :output-type="'png'"
            :info-true="true"
            :can-move="true"
            :can-move-box="true"
            :fixed="true"
            :fixed-number="[3, 2]"
            :auto-crop-width="600"
            :auto-crop-height="400"
            :mode="'cover'"
            :center-box="true"
            :enlarge="5"
            :full="true"
            :max-img-size="200000"
            @imgLoad="imgLoadCrop"
          />
          <div v-if="toolIndex === 4" class="tumo-img">
            <canvas class="canvas" />
          </div>
          <div v-if="recommendDialog" class="recommend-title">
            您刚才的编辑还未保存，返回将放弃所有编辑
          </div>
        </div>
        <template v-if="!recommendDialog">
          <div class="too-btns">
            <div class="left" @click="toolClick(1)">左旋90</div>
            <div class="right" @click="toolClick(2)">右旋90</div>
            <div class="cut" :class="{'active':toolIndex === 3}" @click="toolClick(3)">{{ toolIndex === 3 ? '取消裁剪':'裁剪' }}</div>
            <div class="tumo" :class="{'active':toolIndex === 4}" @click="toolClick(4)">{{ toolIndex === 4 ? '取消涂抹':'涂抹' }}</div>
          </div>
          <div v-show="toolIndex === 4" class="block-slider">
            <p>涂抹直径</p>
            <el-slider v-model="widthTm" :min="2" :max="50" :step="2" @input="changeWidthTm" />
          </div>
        </template>
        <div slot="footer" class="dialog-footer">
          <el-button v-if="recommendDialog" @click="saveEdit">不保存</el-button>
          <el-button :loading="saveLoading" type="primary" @click="saveEdit(1)">保存修改</el-button>
        </div>
      </el-dialog>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="saveList">保存车源图片</el-button>
        <el-button @click="outerVisible = false">取 消</el-button>
      </div>
    </el-dialog>
    <div @click="outerVisible = true">
      <slot>
        <!-- slot 数据 -->
      </slot>
    </div>
  </div>
</template>

<script>
import { VueCropper } from 'vue-cropper'
import draggable from 'vuedraggable'
import axios from 'axios'
export default {
  name: 'UploadImg',
  components: {
    VueCropper,
    draggable
  },
  props: {
    value: {
      type: Array,
      default: () => { [] }
    }
  },
  data() {
    return {
      outerVisible: false, // 最外层dialog
      innerVisible: false, // 裁剪弹窗
      limitNumb: 15, // 图片上传数量限制
      loadingShow: false, // 上传图片列表加载
      loadingCrop: false, // 图片编辑时候加载
      // 需要修改的地方 改成自己的 uploadUrl headers
      uploadUrl: '/admin/basic/upload/ArticleCover',
      headers: {
        uid: '123', // this.$sessionObj.uid
        userToken: 'acadcdfefefeafeppoogr123fd', // this.$sessionObj.userToken
        platform: 1, // this.$sessionObj.platform
        deviceInfo: 'acadcdfefefeafeppoogr123fd' // this.$sessionObj.deviceInfo
      },

      editIndex: '', // 被选中的图片的角标
      editImgUrl: '', // 选中图片地址

      toolIndex: 0, // 操作按钮 1 做旋转 2 右旋转 3 裁剪 4 马赛克

      tumoIngUrl: '', // 涂抹时候的图片
      widthTm: 30, // 涂抹半径

      recommendDialog: false, // 没有保存退出的提示框
      isCut: false, // 是否裁剪
      isTumo: false, // 是否涂抹
      isRotate: 0, // 是否旋转
      saveLoading: false // 是否保存
    }
  },
  computed: {
    //   判断是否修改过
    isEdit() {
      return this.isCut || this.isTumo || parseInt(this.isRotate % 4) !== 0
    },
    dragOptions() {
      return {
        animation: 200, // 动画时间
        disabled: false, // false可拖拽，true不可拖拽
        group: 'description',
        ghostClass: 'ghost'
      }
    }
  },
  methods: {
    //  格式化第二个dialog
    refrehDialog() {
      this.toolIndex = 0
      this.tumoIngUrl = ''
      this.recommendDialog = false
      this.isCut = false
      this.isTumo = false
      this.isRotate = 0
      this.saveLoading = false
      this.editImgUrl = ''
    },
    beforeClose() {
      //   判断是否修改了
      if (this.isEdit) {
        this.recommendDialog = true
        return false
      } else {
        this.innerVisible = false
        this.refrehDialog()
      }
    },
    saveEdit(val) {
      if (val === 1) {
        this.saveLoading = true
        //   保存图片 上传图片
        if (this.toolIndex === 4) {
          this.editImgUrl = this.tumoIngUrl
          setTimeout(() => {
            this.uploadImg()
          }, 500)
        } else {
          this.uploadImg()
        }
      } else {
        //   直接关闭
        this.innerVisible = false
        this.refrehDialog()
      }
    },
    // 上传图片
    uploadImg() {
      // 需要修改的地方 改成自己的 res.data.response[0] 这个获取图片地址格式自己定
      const formData = new FormData()
      this.$refs.cropper.getCropBlob(data => {
        formData.append('file', data, new Date().getTime() + 'edit_.png')
        axios.post(this.uploadUrl, formData).then((res) => {
          const imgList = this.value.slice()
          imgList[this.editIndex] = res.data.response[0]
          this.$emit('input', imgList)
          this.saveLoading = false
          this.innerVisible = false
          this.refrehDialog()
        }).catch(() => { this.saveLoading = false })
      })
    },
    onSucessUpload(response, file, fileList) {
      console.log(response, file, fileList)
      this.loadingShow = false
      const imgList = this.value.slice()
      // 需要修改的地方 改成自己的 response.response[0] 这个获取图片地址格式
      imgList.push(response.response[0])
      this.$emit('input', imgList)
      this.limitNumb = 15 - this.value.length
    },
    onExceedupload(files) {
      console.log(files,'files')
      this.$message.warning(`做多上传15张图片`)
    },
    uploadBefore(file) {
      console.log(file,'uploadBefore')

      this.loadingShow = true
      // 图片格式是否正确
      let bool = null
      // 获取图片的后缀名
      const dotIndex = file.name.lastIndexOf('.')
      const suffixOfImage = file.name.slice(dotIndex + 1).toLowerCase()
      switch (suffixOfImage) {
        case 'jpeg':
        case 'png':
        case 'jpg':
          bool = true
          break
        default:
          bool = false
      }
      if (bool) {
        const isLt5Mb = file.size / 1024 / 1024 < 5
        if (!isLt5Mb) {
          this.loadingShow = false
          this.$message.warning('上传图片大小不能超过 5MB!')
          return false
        }
      } else {
        this.$message.warning(
          '您上传的图片格式不正确!（仅支持jpeg，png，jpg）'
        )
        this.loadingShow = false
        return false
      }
    },
    // 删除图片
    deletImgList(index) {
      const imgList = this.value.slice()
      imgList.splice(index, 1)
      this.$emit('input', imgList)
    },
    // 编辑图片
    editImg(index) {
      this.editIndex = index
      this.editImgUrl = this.value[index]
      this.innerVisible = true
      this.loadingCrop = true
    },
    // 开启裁剪
    imgLoadCrop() {
      this.loadingCrop = false
    },
    // 点击工具栏
    toolClick(val) {
      if (val === 1 || val === 2) {
        this.roatatImg(val)
      } else if (val === 3) {
        // 裁剪
        if (this.toolIndex === 3) {
          this.isCut = false
          this.$refs.cropper.clearCrop()
          setTimeout(() => {
            this.toolIndex = 0
          }, 0)
        } else if (this.toolIndex === 4) {
          this.isTumo = true
          this.editImgUrl = this.tumoIngUrl
          setTimeout(() => {
            this.$refs.cropper.goAutoCrop()
          }, 0)
        } else {
          this.$refs.cropper.goAutoCrop()
        }
      } else {
        if (this.toolIndex === 4) {
          setTimeout(() => {
            this.toolIndex = 0
          }, 0)
        } else {
          this.$refs.cropper.getCropData((data) => {
            this.tumoIngUrl = data
            this.$refs.cropper.clearCrop()
            setTimeout(() => {
              this.tuMoImg()
            }, 0)
          })
        }
      }
      this.toolIndex = val
    },
    // 左右旋转
    roatatImg(val) {
      if (val === 1) this.isRotate++
      else this.isRotate--
      if (this.toolIndex === 3) {
        this.isCut = true
        this.$refs.cropper.getCropData((data) => {
          this.editImgUrl = data
          this.$refs.cropper.clearCrop()
          setTimeout(() => {
            if (val === 1) this.$refs.cropper.rotateLeft()
            else this.$refs.cropper.rotateRight()
          }, 0)
        })
      } else if (this.toolIndex === 4) {
        this.isTumo = true
        this.editImgUrl = this.tumoIngUrl
        setTimeout(() => {
          if (val === 1) this.$refs.cropper.rotateLeft()
          else this.$refs.cropper.rotateRight()
        }, 0)
      } else {
        if (val === 1) this.$refs.cropper.rotateLeft()
        else this.$refs.cropper.rotateRight()
      }
    },
    changeWidthTm(val) {
      document.querySelector('.block-slider .el-slider__button').style.width = val + 'px'
      document.querySelector('.block-slider .el-slider__button').style.height = val + 'px'
    },
    // 涂抹逻辑
    tuMoImg() {
      const cs = document.querySelector('.canvas')
      const roate = 3
      cs.width = roate * 650
      cs.height = cs.width / 3 * 2

      const ctx = cs.getContext('2d')
      const upImg = new Image()
      const backImg = new Image()
      let isRip = false // 是否开撕
      upImg.src = this.tumoIngUrl
      //  需要修改的地方 换成马赛克图片
      backImg.src = require('../assets/images/bg2.png')
      upImg.onload = () => {
        // 判断绘制路径
        let dw = 0
        let dh = 0
        let dx = 0
        let dy = 0
        if ((upImg.width / upImg.height) >= (3 / 2)) {
          // 需要满足宽度
          const r = upImg.width / cs.width
          dw = cs.width
          dh = upImg.height / r
          dy = (cs.height - dh) / 2
        } else {
          // 需要满足高度
          const r = upImg.height / cs.height
          dh = cs.height
          console.log(upImg.height, cs.height, r)
          dw = upImg.width / r
          dx = (cs.width - dw) / 2
        }
        console.log(upImg.width, upImg.height, dx, dy, dw, dh)
        ctx.drawImage(upImg, 0, 0, upImg.width, upImg.height, dx, dy, dw, dh) // 画上面的图片
      }
      cs.onmousedown = function() {
        isRip = true
      }
      cs.onmouseup = function() {
        isRip = false
      }
      let timer = null
      cs.onmousemove = (e) => {
        if (isRip) {
          const x = e.offsetX
          const y = e.offsetY
          const w = this.widthTm
          ctx.drawImage(backImg, 0, 0, w, w, (x - w / 2) * roate, (y - w / 2) * roate, w, w)
        }
        if (timer) clearTimeout(timer)
        timer = setTimeout(() => {
          this.tumoIngUrl = cs.toDataURL()
        }, 200)
      }
    },
    // 保存
    saveList() {
      this.outerVisible = false
      this.$emit('saveList', this.value)
    }
  }
}
</script>
<style>
.update-dialog .el-dialog__body {
  padding-top: 10px;
}
.update-dialog .el-icon-circle-close {
  color: red;
  position: absolute;
  font-size: 30px;
  right: -10px;
  top: -10px;
}
.cropper-modal {
  background: rgba(255, 0, 0, .75) !important;
}
.block-slider .el-slider__bar {
    background: rgba(0, 0, 0, .5);
}
.block-slider .el-slider__button-wrapper {
    position: relative;
}
.block-slider .el-slider__button {
    border: none;
    background: red;
    height: 30px;
    width: 30px;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%,-50%);
}
</style>
<style scoped>
.titl {
  font-size: 20px;
  font-weight: 700;
}
.titl span {
  font-size: 16px;
  font-weight: 500;
  margin-left: 20px;
  color: red;
}
.desc {
    padding-bottom: 15px;
}
.desc span,.slid span {
  color: #409eff;
}
.img-list {
    list-style: none;
    display: flex;
    flex-wrap: wrap;
    max-height: 350px;
    overflow-y: scroll;
}
.img-list .item:nth-child(1)::after {
    content: '首图';
    position: absolute;
    width: 40px;
    height: 20px;
    text-align: center;
    line-height: 20px;
    top: 0;
    left: 0;
    color: #ffffff;
    font-size: 14px;
    background: coral;
}
.img-list li,.img-li{
    width: 200px;
    height: 133px;
    margin: 15px 10px;
    position: relative;
}

.img-li {
    float: left;
}
.img-list li img{
    width: 100%;
    height: 100%;
    object-fit: contain;
    background: #000;
}
.upload-class {
  color: #409eff;
  text-align: center;
  height: 100%;
  line-height: 133px;
  border: 1px dashed #409eff;
  box-sizing: border-box;
}
.edit-img {
    width: 650px;
    height: 433px;
    position: relative;
}
.tumo-img {
    position: absolute;
    width: 650px;
    height: 433px;
    top: 0;
    left: 0;
}
.edit-img img{
    width: 100%;
    height: 100%;
    object-fit: contain;
}
.too-btns {
    display: flex;
    justify-content: space-around;
    padding-top: 20px;
}
.too-btns  div{
    min-width: 110px;
    height: 40px;
    line-height: 40px;
    text-align: center;
    padding: 0 10px;
    color: #fff;
    background: rgba(0, 0, 0, .6);
}
.too-btns .active{
    background: #000;
    color: yellow;
}
.too-btns div:hover{
    background: #000;
    color: yellow;
    cursor:pointer
}
.block-slider {
    width: 45%;
    margin: 0 auto;
    margin-top: 30px;
    text-align: center;
}
.block-slider p{
    margin-bottom: 10px;
}
.canvas {
    width: 100%;
    height: 100%;
    background: url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQAQMAAAAlPW0iAAAAA3NCSVQICAjb4U/gAAAABlBMVEXMzMz////TjRV2AAAACXBIWXMAAArrAAAK6wGCiw1aAAAAHHRFWHRTb2Z0d2FyZQBBZG9iZSBGaXJld29ya3MgQ1M26LyyjAAAABFJREFUCJlj+M/AgBVhF/0PAH6/D/HkDxOGAAAAAElFTkSuQmCC');
}
.recommend-title {
    text-align: center;
    line-height: 433px;
    position: absolute;
    width: 650px;
    height: 453px;
    background: #fff;
    top: -10px;
    left: 0;
}
</style>
