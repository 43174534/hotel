<template>
  <div id="empAdd">
    <el-dialog :title="title" :visible.sync="visible" top="0.5rem" :lock-scroll="false" :show-close="false" :close-on-click-modal="false">
      <el-form ref="empForm" :model="item" :rules="rules" label-width="100px">
        <el-form-item label="姓名:" prop="readName">
          <el-input v-model="item.readName"  palceholder="请输入姓名" clearable/>
        </el-form-item>
        <el-form-item label="性别:" prop="sex">
          <el-radio-group v-model="item.sex">
           <el-radio v-model="item.sex" label="1" border>男</el-radio>
          <el-radio v-model="item.sex" label="2" border>女</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="职位:" prop="position">
          <!-- 显示的是职位，传递的是positionId -->
          <el-select v-model="item.position" placeholder="请选择职位" clearable>
              <el-option v-for="i in positionList" :key="i.positionId" :label="i.position" :value="i.positionId"> </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="联系电话:" prop="telPhone">
          <el-input v-model="item.telPhone"  palceholder="请输入联系电话" clearable/>        
        </el-form-item>
        <el-form-item label="身份证号:" prop="IDCard">
          <el-input v-model="item.IDCard"  palceholder="请输入身份证号" clearable/>
        </el-form-item>
        <el-form-item label="工资:" prop="salary">
          <el-input v-model="item.salary"  palceholder="请输入工资" clearable/>
        </el-form-item>
        <el-form-item label="用户头像:">
         <el-upload
            class="avatar-uploader"
            action=""
            :show-file-list="false"
            :on-success="handleAvatarSuccess"
            :before-upload="beforeAvatarUpload">
            <img v-if="item.headImg" :src="item.headImg" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
        </el-form-item>
      </el-form>
      <span slot="footer">
      <el-button type="primary" @click="resetForm('empForm')">取消</el-button>
      <el-button type="primary" @click="submitForm('empForm')">提交</el-button>
    </span>
    </el-dialog>
  </div>
</template>

<script>
import { uploadFile } from '@/api/uploadFile'
import axios from 'axios'
import { getPositionList } from '@/api/position';
export default {
  props: {
    title: String,
    default: 'title'
  },
  data () {
    const validateTelPhone = (rule, value, callback) => { // 验证手机号码
      if (value === '') {
        callback(new Error('请输入手机号码'))
      } else {
        if (value !== '') {
          var regTelPhone = /^1[3456789]\d{9}$/
          if (!regTelPhone.test(value)) {
            callback(new Error('请输入有效的手机号码'))
          } else {
            callback()
          }
        }
      }
    }
    return {
      visible: false,
      uptoken: {
        key: ''
      },
      positionList: [], // 保存职位信息
      fileList: [],
      item: {
        readName: '',
        sex: '',
        position: '',
        telPhone: '',
        headImg: '',
        IDCard:'',
        salary:''
      },
      rules: {
        readName: [{ required: true, message: '请输入姓名', trigger: 'change' }],
        sex: [{ required: true, message: '请选择性别', trigger: 'change' }],
        position: [{ required: true, message: '请选择职位', trigger: 'blur' }],
        telPhone: [{required: true, validator: validateTelPhone, trigger: 'blur'}]
      }
    }
  },
  mounted () {
    this.getPositionList();
  },
  methods: {
    open (item) {
      this.visible = true
      if (item === null || item === undefined) {
        // this.item = {}
      } else {
        this.item = item
      }
    },
    formateDate (date) {
      let theDate = new Date(date)
      let year = theDate.getFullYear()
      let month = theDate.getMonth() + 1
      let day = theDate.getDate()
      let hour = theDate.getHours()
      let minute = theDate.getMinutes()
      let second = theDate.getSeconds()
      return year + '-' + this.formatTen(month) + '-' + this.formatTen(day) + ' ' + this.formatTen(hour) + ':' + this.formatTen(minute) + ':' + this.formatTen(second)
    },
    formatTen (num) {
      return num > 9 ? (num + '') : ('0' + num)
    },
    handleAvatarSuccess(res, file) {
      this.headImg = URL.createObjectURL(file.raw);
    },
    beforeAvatarUpload(file) {
      const isJPG = file.type === 'image/jpeg'
      const isPNG = file.type === 'image/png'
      const isLt2M = file.size / 1024 / 1024 < 2

      if (!isJPG && !isPNG) { // 如果不是jpg，也不是png的时候就弹出这条信息
        this.$message.error('上传头像图片只能是 JPG或PNG 格式!')
      }
      if (!isLt2M) {
        this.$message.error('上传头像图片大小不能超过 2MB!')
      }

      // console.log('file信息是',file)
      let files = new FormData();
      files.append('multipartFile',file)
      let headers = {'Content-Type': 'multipart/form-data'}
      uploadFile(files,headers).then((res) => {
        // console.log('文件上传返回数据',res.data.data)
        if (res.data.code === 0){
          this.item.headImg = res.data.data
        }
      })
      return isJPG || isPNG && isLt2M
    },
    getPositionList() {
      getPositionList(null).then(res => {
        this.positionList = res.data.obj
      })
    },
    submitForm (empForm) {
      this.$refs.empForm.validate(valid => {
        if (valid) {
          this.$confirm('确认保存吗？', '是否保存', {
            cancelButtonText: '取消',
            confirmButtonText: '确认',
            lockScroll: false,
            type: 'warning'
          }).then(() => {
            this.$emit('confirmData', this.item)
            this.resetForm('empForm')
          })
        }
      })
    },
    resetForm (empForm) {
      this.$nextTick(() => {
        this.$refs.empForm.clearValidate()
      })
      this.item = {}
      this.fileFlag = false;
      this.fileUploadPercent = 0
      this.visible = false
    }
  }
}
</script>

<style lang="less">
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
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
  }
  .avatar {
    width: 178px;
    height: 178px;
    display: block;
  }
</style>
