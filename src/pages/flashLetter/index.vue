<template>
  <div class="messagedetail">
    <van-nav-bar
      :title="navTitle"
      left-arrow
      right-text="添加模板"
      :fixed="true"
      @click-left="onClickLeft"
      @click-right="onClickRight"
    />
    <van-popup v-model="showPopup" :overlay="false" position="right">
      <new-tpl v-on:closeMe="showPopup = false" v-on:createSuccess="createSuccess"/>
    </van-popup>

    <scroller style="top:51px" :on-refresh="refresh" :on-infinite="infinite">
      <div class="tpl-panel" v-if="list.length > 0" v-for="(item,index) in list" :key="index">
        <van-row>
          <van-col :span="20">
            <span class="title">{{item.title}}</span>
          </van-col>
          <van-col :span="4" style="text-align: right;">
            <van-tag :color="item.status == 1 ? '#59b0a2' : 'rgb(120,120,120)'">{{item.status ? '通过' : '待审核'}}</van-tag>
          </van-col>
        </van-row>
        <div class="content">【{{item.sign}}】{{item.content}}</div>
        <van-row>
          <van-col :span="12">
            <span class="time">创建时间: {{item.u_times}}</span>
          </van-col>
          <van-col :span="12" style="text-align:right; font-size: 0">
            <span class="delete-btn custom-btn" @click="deleteModel(item.id)">
              <van-icon name="delete"/>
              <span class="text">删除</span>
            </span>
            <span
              class="send-btn custom-btn"
              v-if="item.status == 1"
              @click="showSendMsg(item.id, item.type)"
            >
              <img src="./imgs/message.png" alt v-if="item.types === 1">
              <img src="./imgs/flash.png" alt v-if="item.types === 2">
              <span class="text">{{item.types === 1?'短信群发' :'闪信群发'}}</span>
            </span>
          </van-col>
        </van-row>
      </div>
      <div class="tpl-panel" v-else>暂无数据</div>
    </scroller>
    <van-popup v-model="isShowMsgBox" :overlay="false" position="right">
      <van-nav-bar
        :title="Number(this.$route.params.id) === 1 ? '群发短信' : '群发闪信'"
        @click-left="isShowMsgBox = false"
        left-arrow
      />
      <van-row>
        <van-col span="24">
          <van-tabs v-model="masgactive">
            <van-tab title="人群包">
              <moneyPeople @close="close" :msgpm="msgpm"></moneyPeople>
            </van-tab>
            <van-tab title="客户包">
              <man @close="close" :msgpm="msgpm"></man>
            </van-tab>
          </van-tabs>
        </van-col>
      </van-row>
    </van-popup>
  </div>
</template>

<script>
import {
  NavBar,
  Row,
  Col,
  Icon,
  Dialog,
  Popup,
  Button,
  Toast,
  Tab,
  Field,
  Tabs,
  RadioGroup,
  Radio,
  Tag,
  Checkbox,
  CheckboxGroup
} from "vant";
import moneyPeople from "./moneyPeople";
import man from "./man";
export default {
  name: "money-detail",
  components: {
    newTpl: function(resolve) {
      require(["./new.vue"], resolve);
    },
    [Row.name]: Row,
    [Col.name]: Col,
    [Icon.name]: Icon,
    [Field.name]: Field,
    [NavBar.name]: NavBar,
    [Button.name]: Button,
    [Toast.name]: Toast,
    [Tab.name]: Tab,
    [Tabs.name]: Tabs,
    [Tag.name]: Tag,
    [Popup.name]: Popup,
    [RadioGroup.name]: RadioGroup,
    [Radio.name]: Radio,
    [Checkbox.name]: Checkbox,
    [CheckboxGroup.name]: CheckboxGroup,
    [Dialog.name]: Dialog,
    moneyPeople,
    man
  },
  data() {
    return {
      masgactive: 0,
      packageAnli: "",
      signName: "", // 签名
      navTitle: "", // 名字
      messageOrflashId: "", // 短信还是闪信  2短信 1闪信
      loadinglayer: "",
      packageContent: "",
      packageName: "",
      list: [],
      page: 1,
      showPopup: false,
      hasData: 0,
      msgCont: null, // 群发 信息内容
      isShowMsgBox: false,
      disabled: false,
      activeModel: "0", // 模板类型（短信还是闪信）
      msgpm: {}, // 模板信息
      restWord: null
    };
  },
  watch: {
    signName: function(newValue) {
      if (!newValue) {
        this.packageAnli = this.packageContent;
      } else {
        this.packageAnli = "【" + newValue + "】" + this.packageContent;
      }
    },
    packageContent: function(newValue) {
      if (!this.signName) {
        this.packageAnli = this.packageContent;
      } else {
        if (Number(this.$route.params.id) === 1) {
          this.packageAnli =
            "【" + this.signName + "】 " + this.packageContent + "祝您生活愉快";
        } else {
          this.packageAnli =
            "【" + this.signName + "】 " + this.packageContent + "退订回复T";
        }
      }
      var num =
        (Number(this.$route.params.id) === 1 ? 50 : 52) -
        this.packageContent.length;
      this.restWord = num < 0 ? 0 : num;
    },
    showPopup: function(newValue) {
      this.packageName = "";
      this.packageContent = "";
    },
    activeModel: function(newValue) {
      this.packageName = "";
      this.packageContent = "";
    },
    list: function(newValue) {
      if (newValue.length <= 5) {
        if (this.loadinglayer.length) {
          this.loadinglayer[0].style.opacity = 0;
        }
      } else {
        if (this.loadinglayer.length) {
          // console.info("总的list", this.loadinglayer);
          // this.loadinglayer.getElementsByClassName('no-data-text')[0].style.opacity = 1;
          this.loadinglayer[0].style.opacity = 1;
        }
      }
    }
  },
  mounted() {
    // console.log('this.list:', this.list);
    // console.info('短信还是闪信', this.$route.params.id);
    this.restWord = Number(this.$route.params.id) === 1 ? 50 : 52; // 初始化字数
    if (Number(this.$route.params.id) === 1) {
      this.messageOrflashId = "type=1";
      this.navTitle = "短信模板列表";
    } else if (Number(this.$route.params.id) === 2) {
      this.messageOrflashId = "type=2";
      this.navTitle = "闪信模板列表";
    }
    this.loadinglayer = document.getElementsByClassName("loading-layer");
    // console.info(this.loadinglayer);
    this.getModelList();
  },
  methods: {
    close(data) {
      // 子组件关闭 父组件
      if (!data) {
        this.isShowMsgBox = false;
      }
    },
    createSuccess() {
      this.showPopup = false;

      this.page = 1;
      this.hasData = 0;
      this.list = [];
      this.getModelList();
    },
    showSendMsg(id, type) {
      // 模板 id 和 短信还是闪信
      this.isShowMsgBox = true;
      this.msgpm.id = id;
      this.msgpm.type = type;
    },
    loadMore() {
      this.disabled = true;
      setTimeout(() => {
        for (let i = 0; i < 5; i++) {
          this.list.push(this.list.length);
        }
        this.disabled = false;
      }, 200);
    },
    deleteModel(valueId) {
      let params = {
        id: valueId
      };
      Dialog.confirm({
        title: "确定要删除吗？"
      })
        .then(() => {
          this.$post("client/deltemp", params, res => {
            console.info("删除", res);
            if (res.data.code === 200) {
              Toast({
                message: res.data.msg,
                position: "middle",
                duration: 1000
              });
              this.page = 1;
              this.getModelList();
            } else {
              Toast({
                message: res.data.msg,
                position: "middle",
                duration: 1000
              });
            }
          });
        })
        .catch(() => {
          // on cancel
        });
    }, // 删除模板
    createCancel() {
      this.showPopup = false;
    }, //  取消创建
    createComfirm() {
      let that = this;
      if (!this.signName) {
        Toast({
          message: "签名不能为空",
          position: "top"
        });
      }
      if (!this.packageName || !this.packageContent) {
        Toast({
          message: "标题内容不能为空",
          position: "top"
        });
        return;
      }
      this.disabled = true;
      var lletter = null;
      if (Number(this.$route.params.id) === 1) {
        lletter = "祝您生活愉快";
      } else {
        lletter = "退订回复T";
      }
      let params = {
        title: this.packageName,
        content: "【" + this.signName + "】" + this.packageContent + lletter,
        // type: this.activeModel + 1
        type: this.$route.params.id
      };
      console.info(params);
      this.$post("group/letter", params, res => {
        console.info(res);
        if (res.data.status === 1) {
          Dialog.alert({
            title: "提示",
            message: "创建成功"
          }).then(() => {
            that.showPopup = false;
            that.page = 1;
            that.hasData = 0;
            that.list = [];
            this.getModelList();
          });
        } else {
          Dialog.alert({
            title: "提示",
            message: res.data.msg
          }).then(() => {});
        }
        that.disabled = false;
      });
    }, //  确定创建
    onClickLeft() {
      console.log('this.$route.params.back:', this.$route.params.back);
      if (this.$route.params.back === 'peoplepackage') {
        this.$router.push("/peoplepackage");
      } else {
        this.$router.push("/setting");
      }
    },
    onClickRight() {
      this.showPopup = !this.showPopup;
    }, // 添加模板
    getModelList() {
      this.$get(
        `client/smslist?types=${this.$route.params.id}&page=${this.page}&limit=8` ,
        "",
        res => {
          if (res.data.code === 200) {
            if (this.page === 1) {
              this.list = [];
              // console.info("第一个if:打印this.list", this.list);
            }
            if (res.data.data.list.length === 0) {
              this.hasData = 1;
              // console.info("第二个if:打印this.hasData", this.hasData);
            } else {
              this.list = [];
              for (let key in res.data.data.list) {
                this.list.push(res.data.data.list[key]);
              }
              // console.info("第二个if的else:打印this.list", this.list);
            }
            if (this.list.length === 0) {
              if (this.loadinglayer.length) {
                this.loadinglayer[0].style.opacity = 0;
              }
            }
          } else {
            this.hasData = 1;
            Dialog.alert({
              title: "提示",
              message: res.data.msg
            });
          }
          console.info('res短信闪信', res);
        }
      );
    },
    refresh(done) {
      setTimeout(() => {
        this.page = 1;
        this.list = [];
        this.hasData = 0;
        this.getModelList();
        done();
      }, 1500);
    },
    infinite(done) {
      // 加载更多插件
      // console.info(this.hasData, this.page);
      if (this.hasData) {
        // 代表没有 更多数据了
        done(true);
      } else {
        setTimeout(() => {
          this.getModelList();
          setTimeout(() => {
            done();
            this.page++;
          }, 400);
        }, 1000);
      }
    }
  }
};
</script>
<style>
.messagedetail .van-popup {
  width: 100% !important;
  height: 100% !important;
  background: #fff;
}
</style>
<style scoped>
.tpl-panel {
  padding: 15px;
  border-bottom: 0.5px solid #eee;
}

.tpl-panel .time {
  font-size: 12px;
  line-height: 12px;
  vertical-align: middle;
  color: #999;
}

.tpl-panel .title {
  font-size: 14px;
  color: #333;
}

.tpl-panel .custom-btn {
  display: inline-block;
  padding: 5px;
  font-size: 0;
  border: 0.5px solid #eee;
  border-radius: 2px;
  margin-left: 10px;
}

.tpl-panel .custom-btn .van-icon {
  vertical-align: -6px;
  font-size: 12px;
}

.tpl-panel .custom-btn .text {
  font-size: 12px;
  line-height: 12px;
  vertical-align: middle;
  padding-left: 3px;
}

.tpl-panel .custom-btn img {
  vertical-align: -7px;
  height: 14px;
}

.tpl-panel .custom-btn.delete-btn {
  color: rgb(221, 70, 70);
  border-color: rgb(221, 70, 70);
  padding: 6px 5px;
}

.tpl-panel .custom-btn.send-btn {
  color: rgb(133, 182, 173);
  border-color: rgb(133, 182, 173);
}

.tpl-panel .content {
  padding: 5px;
  background-color: #f5f5f5;
  border-radius: 5px;
  font-size: 14px;
  line-height: 22px;
  margin: 10px 0;
}

.tabMsg {
  padding: 15px;
  border-bottom: 1px #ddd solid;
}
.sendMsg {
  margin-top: -2px;
  float: left;
  margin-left: 20px;
  padding: 0 !important;
  color: #6ab9af;
  position: absolute;
}
.title {
  font-size: 15px;
}
.pt5 {
  padding-top: 8px;
}
.createBtn {
  height: 40px;
  line-height: 40px;
  text-align: center;
  border: 1px solid #eee;
}
.labelTitle {
  font-size: 13px;
  text-align: center;
  color: rgba(0, 0, 0, 0.8);
}
.bagInput {
  width: 90%;
  border: 1px solid #eee;
  border-radius: 3px;
  padding-left: 10px;
  font-size: 12px;
}
.telephone {
  color: rgba(0, 0, 0, 0.8);
  font-size: 14px;
}
.ages {
  font-size: 11px;
  color: rgba(0, 0, 0, 0.6);
}
.label {
  color: rgba(255, 255, 255, 0.9);
  font-size: 11px;
  padding: 3px 5px;
  border-radius: 2px;
  position: relative;
  top: -2px;
  margin-left: 10px;
}

.labelbgcolorhave {
  background-color: #59b0a2;
}

.labelbgcolorhave.pending {
  background: rgb(120, 120, 120);
}

.labelbgcolorno {
  background-color: #e8988e;
}
.labelbgcolornonet {
  background-color: #93af47;
}
.labelbgcolornull {
  background-color: #d9a338;
}
.lastdate {
  font-size: 12px;
  color: rgba(0, 0, 0, 0.6);
}
</style>
