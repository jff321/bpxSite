<template>
    <div>
      <van-nav-bar
        title="个人中心"
        left-arrow
        @click-left="onClickLeft"
      />
      <van-cell-group>
        <van-field
          v-model="phone"
          required
          clearable
          label="登录账号"
          placeholder="手机号"
        />

        <van-field
          v-model="newPwd"
          type="password"
          label="新密码"
          placeholder="新密码"
          required
          clearable
        />

        <van-field
          v-model="company"
          clearable
          label="公司名称"
          placeholder="公司名称"
        />

        <van-field
          v-model="sign"
          clearable
          label="公司签名"
          placeholder="公司签名，只能设置一次"
          :readonly="isReadOnly"
          required
        />

        <van-field
          v-model="name"
          required
          clearable
          label="负责人姓名"
          placeholder="负责人姓名"
        />

        <!--<van-field-->
          <!--v-model="signature"-->
          <!--clearable-->
          <!--label="签名"-->
          <!--placeholder="签名"-->
          <!--readonly-->
        <!--/>-->

        <van-field
          v-model="phone2"
          type="textarea"
          clearable
          label="可拨打手机号"
          placeholder="例：13890563283,13956487936"
          autosize
          @clear="clearPhone2"
          error-message="可填写多个手机号，并以英文逗号隔开"
        />

        <!--<div class="note">可填写多个手机号，并以英文逗号隔开</div>-->

      </van-cell-group>
      <van-cell-group>
        <van-cell value="上传营业执照" />
        <div class="imgBox">
          <!--<div class="img">-->
            <el-upload
              class="avatar-uploader"
              action="http://192.168.0.120/client/upload/image"
              :show-file-list="false"
              accept="image/*"
              :on-preview="handlePictureCardPreview"
              :on-success="handleAvatarSuccess">
              <img v-if="imageUrl" :src="imageUrl" class="avatar">
              <i v-else class="el-icon-plus avatar-uploader-icon"></i>
            </el-upload>
          <!--</div>-->
        </div>
      </van-cell-group>
      <van-cell-group>
        <van-cell value="上传法人/使用者身份证部件" />
        <div class="imgBox">
          <!--<div class="img">-->
          <el-upload
            class="avatar-uploader"
            action="http://192.168.0.120/client/upload/image"
            :show-file-list="false"
            accept="image/*"
            :on-success="handleAvatarSuccess2">
            <img v-if="idCardUrl" :src="idCardUrl" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
          <!--</div>-->
        </div>
      </van-cell-group>
      <van-cell-group>
        <van-cell value="上传公司LOGO" />
        <div class="imgBox">
          <!--<div class="img">-->
          <el-upload
            :headers="myHeaders"
            class="avatar-uploader"
            action="http://192.168.0.120/client/upload/image"
            :show-file-list="false"
            accept="image/*"
            :on-success="handleAvatarSuccess3">
            <img v-if="cardUrl" :src="cardUrl" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload>
          <!--</div>-->
        </div>
      </van-cell-group>
      <div class="create-wrap" @click="commitInfo">
        <span class="custom-bg-color">提交</span>
      </div>
    </div>
</template>

<script>
  import { Field, NavBar, Cell, CellGroup, Button, Dialog } from 'vant';
  export default {
    name: "personalCenter",
    components: {
      [Field.name]: Field,
      [NavBar.name]: NavBar,
      [Cell.name]: Cell,
      [CellGroup.name]: CellGroup,
      [Button.name]: Button,
      [Dialog.name]: Dialog
    },
    data() {
      return {
        user: {
          telId: localStorage.getItem("telId"),
          user_type: localStorage.getItem("user_type")
        },
        phone: '',
        originPwd: '',
        newPwd: '',
        manyPhone: [],
        phone2: '',
        name: '',
        company: '',
        sign: '',
        imageUrl: '',
        imageName: '',
        cardUrl: '',
        cardName: '',
        idCardUrl: '',
        idCardName: '',
        isReadOnly: false,
        myHeaders:{
          Authorization: localStorage.getItem('token')
        },
      };
    },
    mounted() {
      this.$get("client/myinfo", '', res => {
        console.log('res:', res);
        // // 登录账号
        this.phone = res.data.mobile;
        // // 公司名称
        this.company = res.data.company;
        // console.log('公司名称：', this.company);
        // // 公司签名
        this.sign = res.data.sign;
        if (this.sign === null || '') {
          this.isReadOnly = false;
        } else {
          this.isReadOnly = true;
        }
        // // 负责人姓名
        this.name = res.data.uname;
        // // 可拨打手机号
        this.manyPhone = res.data.phone;
        // this.phone2 = this.manyPhone.join(",");
        this.phone2 = this.manyPhone;
        // // 公司营业执照名字
        // this.imageName = res.data.data[0].company_photo;
        // // 公司营业执照路径
        this.imageUrl = res.data.biz_url;
        // // 身份证头像
        // this.idCardName = res.data.data[0].id_card;
        // // 身份证路径
        this.idCardUrl = res.data.id_url;
        // // 头像名字
        // this.cardName = res.data.data[0].picture;
        // // 头像路径
        this.cardUrl = res.data.logo_url;
      });
    },
    methods: {
      onClickLeft() {
        this.$router.push({path: 'setting'});
      },
      handlePictureCardPreview(file) {
        // console.log('FILE: ', file);
      },
      // 上传营业执照
      handleAvatarSuccess(res, file) {
        this.imageName = res.data.name;
        this.imageUrl = res.data.big_url;
        console.log('this.imageUrl服务器:', this.imageUrl);
      },
      // 上传身份证
      handleAvatarSuccess2(res, file) {
        this.idCardName = res.data.name;
        this.idCardUrl = res.data.big_url;
      },
      // 上传公司LOGO
      handleAvatarSuccess3(res, file) {
        this.cardName = res.data.name;
        this.cardUrl = res.data.big_url;
      },
      // 清除多个手机号
      clearPhone2() {
        console.log('this.phone2:', this.phone2);
      },
      commitInfo() {
        console.log('query in');
        if (this.phone === '') {
          Dialog.alert({
            title: "提示",
            message: "请输入手机号"
          });
          return false;
        }
        if (this.phone.length !== 11) {
          Dialog.alert({
            title: "提示",
            message: "手机号格式不对"
          });
          return false;
        }
        // for (let i in this.phone2) {
        //   console.log('this.manyPhone:', this.phone2);
        //   let myreg = /^(13[0-9]|14[579]|15[0-3,5-9]|16[6]|17[0135678]|18[0-9]|19[89])\d{8}$/;
        //   if (!myreg.test(this.phone2[i])) {
        //     Dialog.alert({
        //       title: "提示",
        //       message: "手机号格式不对"
        //     });
        //     this.phone2.splice(i, 1);
        //     return false;
        //   }
        // }
        if (this.newPwd === '') {
          Dialog.alert({
            title: "提示",
            message: "请输入新密码"
          });
          return false;
        }
        // if (this.company === '') {
        //   Dialog.alert({
        //     title: "提示",
        //     message: "请输入公司名称"
        //   });
        //   return false;
        // }
        if (this.sign === '') {
          Dialog.alert({
            title: "提示",
            message: "请输入公司签名"
          });
          return false;
        }
        if (this.name === '') {
          Dialog.alert({
            title: "提示",
            message: "请输入负责人姓名"
          });
          return false;
        }
        if (this.imageUrl === '') {
          Dialog.alert({
            title: "提示",
            message: "请上传营业执照"
          });
          return false;
        }
        if (this.idCardUrl === '') {
          Dialog.alert({
            title: "提示",
            message: "请上传法人/使用者身份证部件"
          });
          return false;
        }
        if (this.cardUrl === '') {
          Dialog.alert({
            title: "提示",
            message: "请上传公司LOGO"
          });
          return false;
        }
        let params = {
          pwd: this.newPwd,
          company: this.company,
          sign: this.sign,
          uname: this.name,
          phone: this.phone2,
          biz_url: this.imageUrl,
          id_url: this.idCardUrl,
          logo_url: this.cardUrl
        };
        this.$post("client/doinfo", params, res => {
          console.log('res:', res);
          if (res.data.code === 200) {
            this.$message({
              message: '修改成功！',
              type: 'success'
            });
            this.onClickLeft();
          } else {
            Dialog.alert({
              title: "提示",
              message: res.data.msg
            });
            return false;
          }
        });
      }
    }
  };
</script>

<style lang="scss" scoped>
.imgBox{
  display: flex;
  justify-content: center;
  align-items: center;
  padding-top: 20px;
}
.img{
  width: 90%;
  height: 300px;
  border: 1px dashed #999;
  text-align: center;
  line-height: 300px;
  color: #999999;
}
.create-wrap {
  padding: 15px;
  .custom-bg-color {
    display: inline-block;
    background: linear-gradient(to right, #92c5e7, #4588d4);
    border-radius: 3px;
    color: white;
    text-align: center;
    height: 50px;
    line-height: 50px;
    width: 100%;
    vertical-align: middle;
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
.note {
  text-align: right;
  font-size: 12px;
  padding:5px 15px 0px 0px;
  color: #8c939d;
}
</style>
