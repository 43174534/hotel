<template>
    <div id="empAdd">
        <el-dialog
            :title="title"
            :visible.sync="visible"
            top="0.5rem"
            :lock-scroll="false"
            :show-close="false"
            :close-on-click-modal="false"
        >
            <el-form ref="empForm" :model="item"  label-width="100px">
                <el-form-item label="账号:" prop="number">
                    <el-input v-model="item.number" palceholder="账号长度必须在6-16之间，且不能包含非法字符*#@等" clearable />
                </el-form-item>

                <el-form-item label="姓名:" prop="readName">
                    <el-input v-model="item.readName" palceholder="请输入姓名" clearable />
                </el-form-item>

                <el-form-item label="密码:" prop="password">
                    <el-input v-model="item.password" palceholder="请输入密码" clearable />
                </el-form-item>

                <el-form-item label="性别:" prop="sex">
                    <el-radio-group v-model="item.sex">
                        <el-radio v-model="item.sex" label="1" border>男</el-radio>
                        <el-radio v-model="item.sex" label="0" border>女</el-radio>
                    </el-radio-group>
                </el-form-item>

                <el-form-item label="级别:" prop="grade">
                    <el-select v-model="item.grade" placeholder="请选择级别" clearable>
                        <el-option v-for="i in positionList" :key="i.value" :label="i.label" :value="i.value"> </el-option>
                    </el-select>
                </el-form-item>

                <el-form-item label="联系电话:" prop="telPhone">
                    <el-input v-model="item.telPhone" palceholder="请输入联系电话" clearable />
                </el-form-item>

                <el-form-item label="生日:" prop="birthday">
                    <el-date-picker v-model="item.birthday" type="date" placeholder="选择日期" value-format="yyyy-MM-dd"> </el-date-picker>
                </el-form-item>

                <el-form-item label="身份证:" prop="idcard">
                    <el-input v-model="item.idcard" palceholder="请输入常用地址" clearable />
                </el-form-item>

                <el-form-item label="地址:" prop="address">
                    <el-input v-model="item.address" palceholder="请输入常用地址" clearable />
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
import { uploadFile } from '@/api/uploadFile';
export default {
    props: {
        title: String,
        default: 'title'
    },
    data() {
        const validateNumber = (rule, value, callback) => {
            var reg = /^[0-9a-zA-Z]{6,16}$/; // 账号长度必须在6-16之间，且不能包含非法字符*#@
            if (value === '' || value === undefined) {
                callback(new Error('请输入账号'));
            } else if (value.length < 6 || value.length > 18) {
                callback(new Error('用户名长度必须在6-16之间'));
            } else if (value.indexOf('*') > 0 || value.indexOf('#') > 0 || value.indexOf('@') > 0 || value.indexOf('&') > 0) {
                callback(new Error('用户名不能含有非法字符*&#@'));
            } else {
                if (reg.test(value)) {
                    callback();
                }
            }
        };
        const validateTelPhone = (rule, value, callback) => {
            // 验证手机号码
            if (value === '') {
                callback(new Error('请输入手机号码'));
            } else {
                if (value !== '') {
                    var regTelPhone = /^1[3456789]\d{9}$/;
                    if (!regTelPhone.test(value)) {
                        callback(new Error('请输入有效的手机号码'));
                    } else {
                        callback();
                    }
                }
            }
        };
        return {
            visible: false,
            uptoken: {
                key: ''
            },
            positionList: [
                // 保存级别信息
                {
                    value: '0',
                    label: '普通会员'
                },
                {
                    value: '1',
                    label: '高级会员'
                }
            ],
            fileList: [],
            item: {
                number: '',
                readName: '',
                sex: '',
                position: '',
                telPhone: '',
                headImg: '',
                birthday: '',
                idcard: '',
                address: '',
                grade: ''
            }
        };
    },

    methods: {
        open(item) {
            this.visible = true;
            if (item === null || item === undefined) {
                // this.item = {}
            } else {
                this.item = item;
            }
        },
        handleAvatarSuccess(res, file) {
            this.headImg = URL.createObjectURL(file.raw);
        },
        beforeAvatarUpload(file) {
            const isJPG = file.type === 'image/jpeg';
            const isPNG = file.type === 'image/png';
            const isLt2M = file.size / 1024 / 1024 < 2;

            if (!isJPG && !isPNG) {
                // 如果不是jpg，也不是png的时候就弹出这条信息
                this.$message.error('上传头像图片只能是 JPG或PNG 格式!');
            }
            if (!isLt2M) {
                this.$message.error('上传头像图片大小不能超过 2MB!');
            }

            // console.log('file信息是',file)
            let files = new FormData();
            files.append('multipartFile', file);
            let headers = { 'Content-Type': 'multipart/form-data' };
            uploadFile(files, headers).then((res) => {
                // console.log('文件上传返回数据',res.data.data)
                if (res.data.code === 0) {
                    this.item.headImg = res.data.data;
                }
            });
            return isJPG || (isPNG && isLt2M);
        },
        submitForm(empForm) {
            this.$refs.empForm.validate((valid) => {
                if (valid) {
                    this.$confirm('确认保存吗？', '是否保存', {
                        cancelButtonText: '取消',
                        confirmButtonText: '确认',
                        lockScroll: false,
                        type: 'warning'
                    }).then(() => {
                        this.$emit('confirmData', this.item);
                        console.log(this.item.data);

                        this.resetForm('empForm');
                    });
                }
            });
        },
        resetForm(empForm) {
            this.$nextTick(() => {
                this.$refs.empForm.clearValidate();
            });
            this.item = {};
            this.fileFlag = false;
            this.fileUploadPercent = 0;
            this.visible = false;
        }
    }
};
</script>

// <style lang="less">
// .avatar-uploader .el-upload {
//   border: 1px dashed #d9d9d9;
//   border-radius: 6px;
//   cursor: pointer;
//   // position: relative;
//   // overflow: hidden;
//   // text-align: left;
//   width: 28%;
//   display: flex;
//   justify-content: center;
//   align-items: center;
// }
// .avatar-uploader .el-upload:hover {
//   border-color: #409eff;
// }
// .avatar-uploader-icon {
//   font-size: 28px;
//   color: #8c939d;
//   width: 80px;
//   height: 80px;
//   line-height: 80px;
//   text-align: center;
// }
// .el-upload__tip {
//   margin-top: 0;
// }
// .avatar {
//   width: 80px;
//   height: 80px;
//   display: block;
// }
//
//</style>

<style lang="less">
.avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
}
.avatar-uploader .el-upload:hover {
    border-color: #409eff;
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
