<template>
  <div>
    <van-nav-bar
      title="感知器管理"
      left-arrow
      @click-left="onClickLeft"
      right-text="添加"
      :fixed="true"
      @click-right="onClickRight"
    />
    <van-row style="padding: 45px 15px 0px 15px">
      <template v-if="probeList.length > 0" v-for="(items,index) in probeList">
        <van-col
          :span="24"
          style="padding: 10px 0; border-bottom: 0.5px solid #eee;"
          :key="index"
        >
          <van-col :span="16">
            <h2 class="title">感知器名称 {{items.name}}</h2>
            <p class="time-wrap">
              <span class="time">感知器id {{items.number_id}}</span>
            </p>
          </van-col>
        </van-col>
      </template>
      <template v-else>
        <div style="margin: auto">暂无数据</div>
      </template>
    </van-row>
    <van-popup v-model="isShow" style="width: 100%;">
      <van-cell-group>
        <van-field
          v-model="name"
          required
          clearable
          label="名称"
          placeholder="请输入感知器名称"
        />

        <van-field
          v-model="probeId"
          required
          clearable
          label="感知器id"
          placeholder="请输入感知器id"
        />
      </van-cell-group>
      <div class="create-wrap" @click="commitProbe">
        <span class="custom-bg-color">提交</span>
      </div>
    </van-popup>
  </div>
</template>

<script>
  import { Field, NavBar, Cell, CellGroup, Button, Popup, Row, Col, Toast, Dialog } from 'vant';
  export default {
    name: "personalCenter",
    components: {
      [Field.name]: Field,
      [NavBar.name]: NavBar,
      [Cell.name]: Cell,
      [CellGroup.name]: CellGroup,
      [Button.name]: Button,
      [Popup.name]: Popup,
      [Row.name]: Row,
      [Col.name]: Col,
      [Toast.name]: Toast,
      [Dialog.name]: Dialog
    },
    data() {
      return {
        user: {
          telId: localStorage.getItem("telId")
        },
        name: '',
        probeId: '',
        isShow: false,
        probeList: []
      };
    },
    mounted() {
      this.$get(
        "probe/binding?user_id=" + this.user.telId,
        "",
        res => {
          console.log('返回的结果RES：', res);
          this.probeList = res.data.data;
          console.log('this.probeList:', this.probeList);
        }
      );
    },
    methods: {
      onClickLeft() {
        this.$router.push({path: 'setting'});
      },
      onClickRight() {
        this.isShow = true;
      },
      commitProbe() {
        this.isShow = false;
        this.$post("probe/binding?user_id=" + this.user.telId + "&name=" + this.name + "&number_id=" + this.probeId,
          "",
          res => {
          console.log(res.data);
            if (res.data.code === 412) {
              Dialog.alert({
                title: "提示",
                message: res.data.msg
              });
              return false;
            }
            this.probeList.push({
              created_at: res.data.data.created_at,
              deleted_at: '',
              id: res.data.data.id,
              lat: '',
              lng: '',
              name: res.data.data.name,
              number_id: res.data.data.number_id,
              pid: res.data.data.pid,
              status: res.data.data.status,
              updated_at: res.data.data.updated_at,
              user_id: res.data.data.user_id,
              user_id_link: {}
            });
          }
        );
        this.name = '';
        this.probeId = '';
      }
    }
  };
</script>

<style lang="scss"  scoped>
.imgBox{
  display: flex;
  justify-content: center;
  margin: 10px 0px;
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
.title {
  font-size: 14px;
  font-weight: normal;
  color: #666;
  /* margin-top: 10px; */
  padding: 3px 0;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
}
.time {
  font-size: 12px;
  line-height: 12px;
  vertical-align: middle;
  color: #999;
  /* margin-top: 3px; */
}
</style>
