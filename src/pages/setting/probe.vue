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
    <scroller
      style="padding-top:0px"
      :on-refresh="refresh"
      :on-infinite="infinite"
      ref="myscroller"
      noDataText="我是有底线的"
    >
      <van-row style="padding: 45px 15px 0px 15px">
      <template v-if="list.length > 0" v-for="(items,index) in list">
        <van-col
          :span="24"
          style="padding: 10px 0; border-bottom: 0.5px solid #eee;"
          :key="index"
        >
          <van-col :span="16">
            <h2 class="title">感知器名称 {{items.name}}</h2>
            <p class="time-wrap">
              <span class="time">感知器id {{items.code}}</span>
            </p>
            <p class="time-wrap">
              <span class="time">感知器sim {{items.sim}}</span>
            </p>
          </van-col>
        </van-col>
      </template>
    </van-row>
    </scroller>
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
        <van-field
          v-model="probeSim"
          required
          clearable
          label="感知器sim"
          placeholder="请输入感知器sim"
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
        probeSim: '',
        isShow: false,
        list: [],
        hasData: 0,
        page: 1,
        limit: 10
      };
    },
    watch: {

    },
    mounted() {
      // this.getDataList();
    },
    methods: {
      getDataList() {
        this.$get(`client/boxlist?page=${this.page}&limit=${this.limit}`,
          "",
          result => {
          // console.log('result.data.data.list:', result.data.data.list);
          // if (result.data.data.list.length) {
          //   this.list = result.data.data.list;
          //   // if(result.data.data.list.length > 10){
          //   //   this.hasData = 1
          //   // }
          // } else {
          //   this.hasData = 1; // 返回没有数据
          // }
          // if (this.list.length === 0) {
          //   if (this.loadinglayer.length) {
          //     this.loadinglayer[0].style.opacity = 0;
          //   }
          // }

          if (result.data.code === 200) {
            if (this.page === 1) {
              this.list = [];
            }
            if (result.data.data.list.length === 0) {
              this.hasData = 1;
            } else {
              if (this.page === 1) {
                this.list = result.data.data.list;
              } else {
                if (this.list[0] && this.list[0].id === result.data.data.list[0].id) {
                  this.list = result.data.data.list;
                } else {
                  for (let key in result.data.data.list) {
                    this.list.push(result.data.data.list[key]);
                  }
                }
              }
            }
            if (this.list.length === 0) {
              if (this.loadinglayer.length) {
                this.loadinglayer[0].style.opacity = 0;
              }
            }
          } else {
            // this.hasData = 1;
            Dialog.alert({
              title: "提示",
              message: res.data.msg
            });
          }
        });
      },
      refresh(done) {
        setTimeout(() => {
          this.page = 1;
          this.list = [];
          this.hasData = 0;
          this.getDataList();
          done(true);
        }, 1500);
      },
      infinite(done) {
        // 加载更多插件
        if (this.hasData) {
          // 代表没有 更多数据了
          done(true);
        } else {
          setTimeout(() => {
            this.getDataList();
            setTimeout(() => {
              done();
              this.page++;
            }, 400);
          }, 1000);
        }
      },
      onClickLeft() {
        this.$router.push({path: 'setting'});
      },
      onClickRight() {
        this.isShow = true;
      },
      commitProbe() {
        this.isShow = false;
        let params = {
          name: this.name,
          code: this.probeId,
          sim: this.probeSim
        };
        this.$post("client/dobox",
          params,
          res => {
            if (res.data.code === 200) {
              this.getDataList()
            } else {
              this.$status(res.data.msg);
            }
          }
        );
        this.name = '';
        this.probeId = '';
        this.probeSim = '';
      },
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
