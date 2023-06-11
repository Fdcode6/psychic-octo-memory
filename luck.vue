<template>
  <view class="app">
    <view class="headad" @click="getluckinfo">
      <!-- banner广告 -->
      <ad unit-id="adunit-e4e76198d96491cc"></ad>
    </view>
    <view class="luck" v-if="luckshow">
      <LuckyGrid ref="myLucky" :blocks="blocks" :prizes="prizes" :buttons="buttons" :activeStyle="activeStyle" :defaultStyle="defaultStyle" @start="startCallBack" @end="endCallBack" />
      <view class="tips">
        <br>当前余额:{{jfnum}}元[0.1元可提现]
      </view>
      <view class="anniu">
        <button class="getjf" @click="getjfnum(1)">刷新余额</button>
        <button class="dhluck" @click="dhluck">余额提现</button>
      </view>
    </view>

    <view v-show="showToast || showMsg || shownoToast || showqrToast || showQrcode || showwxpay||showlingqu||showjfpay">
      <uni-popup type="center" ref="drawer" :is-mask-click="false">

        <view v-if="showToast">
          <view class="lottery-alert">
            <view class="alert-header">恭喜您获得{{prizes[index].fonts[0].text}}</view>
            <p><img :src=prizes[index].imgs[0].src style="height: 300px;" /></p>
            <h2>点击下方领取按钮复制口令领取</h2>
            <div class="btnsave" @click="showToast=false;luckshow=true;copytkl()">领取奖品</div>
          </view>
        </view>

        <view v-show="showwxpay" class="lottery-alert">
          <view class="alert-header">恭喜您获得{{prizes[index].fonts[0].text}}</view>
          <p><img src="https://kfoss6.oss-accelerate.aliyuncs.com/0c36owetB4.jpg" style="height: 300px;" /></p>
          <view class="tip">系统将在一分钟内发送到你微信零钱中</view>
          <view v-show="showsup" class="suptip">原始奖励:{{jpnum}}，翻倍加成:{{beilv}}，最终获得:{{zznum}}</view>
          <view class="btnsave" @click="showwxpay=false;luckshow=true">我知道了</view>
        </view>

        <view v-show="showjfpay" class="lottery-alert">
          <view class="alert-header">恭喜您获得{{prizes[index].fonts[0].text}}</view>
          <p><img :src=prizes[index].imgs[0].src style="height: 300px;" /></p>
          <view class="tip">{{jpnum}}积分将在一分钟内到账，首页刷新积分即可查询</view>
          <view v-show="showsup" class="suptip">原始奖励:{{jpnum}}，翻倍加成:{{beilv}}，最终获得:{{zznum}}</view>
          <view class="btnsave" @click="showjfpay=false;luckshow=true">我知道了</view>
        </view>

        <view v-if="showqrToast">
          <view class="lottery-alert">
            <view>恭喜您获得{{prizes[index].fonts[0].text}}</view>
            <p><img :src=prizes[index].imgs[0].src style="height: 300px;" /></p>
            <view class="tip">请点击下方按钮联系客服</view>
            <view class="btnsave" @click="showqrToast=false;linkQrcode()">点我添加客服领取</view>
          </view>
        </view>

        <view v-show="showQrcode">
          <view class="lottery-alert">
            <view>长按识别下方二维码添加好友</view>
            <p><img :src="qrCodeImg" :show-menu-by-longpress="true" style="height: 300px;" /></p>
            <view class="btnsave" @click="showQrcode=false;luckshow=true">关闭</view>
          </view>
        </view>

        <view v-show="showMsg">
          <view class="lottery-alert">
            <view class="alert-header">{{msg}}</view>
            <view class="btnsave" @click="showMsg=false;luckshow=true">关闭</view>
          </view>
        </view>

        <view v-if="shownoToast">
          <view class="lottery-alert">
            <view class="alert-header">很遗憾本轮未中奖</view>
            <p><img src="https://uservipcdn.oss-accelerate.aliyuncs.com/luck/img/j1.png" alt=""></p>
            <view class="alert-header">谢谢参与</view>
            <div class="btnsave" @click="shownoToast=false;luckshow=true">确定</div>
          </view>
        </view>

        <view v-if="showlingqu">
          <view class="lottery-alert">
            <view class="alert-header">恭喜您获得{{prizes[index].fonts[0].text}}</view>
            <p><img :src=prizes[index].imgs[0].src style="height: 300px;" /></p>
            <view class="alert-header">超级领取有机会奖励翻倍[需完整观看激励广告视频]</view>
            <div class="bts">
              <div class="btndef" @click="showlingqu=false;luckshow=true;ptlingqu()">普通领取<br />少量奖励</div>
              <div class="btnsup" @click="showlingqu=false;luckshow=true;showAdJiLi()">超级领取<br />看广告有机会奖励翻倍</div>
            </div>
          </view>
        </view>

      </uni-popup>

    </view>

  </view>

</template>

<script>
let interstitialAd = null;
import LuckyGrid from "@/components/@lucky-canvas/uni/lucky-grid.vue";
import UniPopup from "@/uni_modules/uni-popup/components/uni-popup/uni-popup";
export default {
  components: {
    LuckyGrid,
    UniPopup,
  },
  data() {
    return {
      //视图层变量
      luckshow: false,
      code: "",
      openid: "",
      unionid: "",
      index: -1, // 当前转动到哪个位置，起点位置
      title: "", //抽奖活动标题
      showToast: false, //显示中奖弹窗
      priceCode: 0, //接口返回代码
      msg: "用户不符或抽奖活动结束",
      showMsg: false,
      shownoToast: false, //显示未中奖弹窗
      showqrToast: false, //显示客服弹窗
      showQrcode: false, //显示客服弹窗
      showwxpay: false, //显示零钱弹窗
      showlingqu: false, //显示领取弹窗
      tkl: "",
      qrCodeImg: "",
      lucktype: 0,
      showjfpay: false, //显示积分弹窗
      showsup: false, //超级领取提示
      jptype: 0, //奖品类型
      beilv: "", //奖品倍率
      jpnum: "", //原始奖品金额
      zznum: "", //最终奖品金额
      jfnum: 0, //积分数量
      blocks: [
        {
          padding: "25px",
          imgs: [
            {
              src: "/static/grid-bg.png",
              width: "100%",
              height: "100%",
            },
          ],
        },
      ],
      prizes: [],
      buttons: [
        {
          x: 1,
          y: 1,
          background: "linear-gradient(270deg, #FFDCB8, #FDC689)",
          shadow: "0 5 1 #e89b4f",
          imgs: [
            {
              src: "/static/btn.png",
              width: "70%",
              top: "20%",
            },
          ],
        },
      ],
      activeStyle: {
        background: "linear-gradient(270deg, #FFDCB8, #FDC689)",
        shadow: "0 5 1 #e89b4f",
      },
      defaultConfig: {
        gutter: 5,
      },
      defaultStyle: {
        borderRadius: 15,
        fontColor: "#708abf",
        fontSize: "14px",
        textAlign: "center",
        background: "#fff",
        shadow: "0 5 1 #ebf1f4",
      },
    };
  },
  created() {},
  onLoad() {
    // #ifdef MP-WEIXIN
    if (wx.createRewardedVideoAd) {
      this.videoAd = wx.createRewardedVideoAd({
        adUnitId: "adunit-6dbb5150e1d3ee09",
      });
      this.videoAd.onLoad(() => {});
      this.videoAd.onError((err) => {});
      this.videoAd.onClose((res) => {
        if (res && res.isEnded) {
          // 正常播放结束，可以下发游戏奖励
          this.suplingqu();
        } else {
          // 播放中途退出，不下发游戏奖励
          uni.showToast({
            title: "看完视频才能可以领取奖励哦",
            icon: "none",
          });
        }
      });
    }
    if (wx.createInterstitialAd) {
      interstitialAd = wx.createInterstitialAd({
        adUnitId: "adunit-9a9596dca299cf9e",
      });
      interstitialAd.onLoad(() => {});
      interstitialAd.onError((err) => {});
      interstitialAd.onClose(() => {});
    }
    // #endif
  },
  onReady() {
    this.getjfnum();
  },
  onShow() {
    this.code = getLocationParams("code");
    this.openid = getLocationParams("openid");
    this.unionid = getLocationParams("unionid");
    //如果没有code，则赋值为空
    if (!this.code) {
      this.code = "";
    }
    //如果没有openid，则赋值为空
    if (!this.openid) {
      this.openid = "";
      if (this.openid.length < 10) {
        this.openid = uni.getStorageSync("openid");
      }
    }
    //如果没有unionid，则赋值为空
    if (!this.unionid) {
      this.unionid = "";
    }

    //如果openid长度大于10把openid存入缓存
    if (this.openid.length > 10) {
      uni.setStorageSync("openid", this.openid);
    }

    //如果code长度小于5，或者openid长度小于5，或者unionid长度小于5，则跳转到首页
    if (
      this.code.length < 5 ||
      this.openid.length < 5 ||
      this.unionid.length < 5
    ) {
      // console.log("参数异常");
      this.msg = "请从群内点击链接进入小程序即可参与抽奖！";
      this.showMsg = true;
      this.luckshow = false;
      this.$refs.drawer.open("center");
    } else {
      // console.log("参数正常");
      this.getluckinfo();
    }
  },
  methods: {
    showAdJiLi() {
      // #ifdef MP-WEIXIN
      if (this.videoAd) {
        this.videoAd.show().catch(() => {
          this.videoAd
            .load()
            .then(() => this.videoAd.show())
            .catch((err) => {
              // console.log("激励视频 广告显示失败");
            });
        });
      }
      // #endif
    },
    // 点击抽奖按钮触发回调
    startCallBack() {
      // console.log("点击抽奖");
      //参数验证
      //如果没有code，则赋值为空
      if (!this.code) {
        this.code = "";
      }
      //如果没有openid，则赋值为空
      if (!this.openid) {
        this.openid = "";
      }
      //如果没有unionid，则赋值为空
      if (!this.unionid) {
        this.unionid = "";
      }

      //如果code长度小于5，或者openid长度小于5，或者unionid长度小于5，则跳转到首页
      if (
        this.code.length < 5 ||
        this.openid.length < 5 ||
        this.unionid.length < 5
      ) {
        //调试跳转到抽奖页面
        console.log("执行跳转");
        uni.setStorageSync("openid", 'oAHpawabE8aDe5hwz9A6-pdVQT_M');
        uni.reLaunch({
          url: "../luck/luck?code=7qE88Y&openid=oAHpawabE8aDe5hwz9A6-pdVQT_M&unionid=oyjhv6RR_c9-cdnd6VFEO5qrq2Uw",
        });
        return;
        console.log("参数异常");
        this.msg = "请从群内点击链接进入小程序即可参与抽奖！";
        this.showMsg = true;
        this.luckshow = false;
        this.$refs.drawer.open("center");
        return;
      }

      //判断lucktype
      if (this.lucktype == 0) {
        this.shownoToast = true;
        this.luckshow = false;
        this.$refs.drawer.open("center");
        return;
      } else if (this.lucktype == 2) {
        this.showqrToast = true;
        this.luckshow = false;
        this.$refs.drawer.open("center");
        return;
      } else if (this.lucktype == 3) {
        //如果tkl文本大于40个字符this.showwxpay = true否则this.showToast = true
        if (this.tkl.length == 44) {
          this.showjfpay = true;
          this.luckshow = false;
          this.$refs.drawer.open("center");
          return;
        } else if (this.tkl.length == 64) {
          this.showwxpay = true;
          this.luckshow = false;
          this.$refs.drawer.open("center");
          return;
        } else {
          this.showToast = true;
          this.luckshow = false;
          this.$refs.drawer.open("center");
          return;
        }
      } else if (this.lucktype == 4) {
        this.showlingqu = true;
        this.luckshow = false;
        this.$refs.drawer.open("center");
        return;
      }

      if (this.priceCode == 1) {
        //用户未中奖
        this.shownoToast = true;
        this.luckshow = false;
        this.$refs.drawer.open("center");
        return;
      } else if (this.priceCode == 2) {
        //中奖了未加好友
        this.showqrToast = true;
        this.luckshow = false;
        this.$refs.drawer.open("center");
        return;
      } else if (this.priceCode == 3) {
        //中奖了已加好友
        //如果tkl文本大于40个字符this.showwxpay = true否则this.showToast = true
        if (this.tkl.length == 44) {
          this.showjfpay = true;
          this.luckshow = false;
          this.$refs.drawer.open("center");
          return;
        } else if (this.tkl.length == 64) {
          this.showwxpay = true;
          this.luckshow = false;
          this.$refs.drawer.open("center");
          return;
        } else {
          this.showToast = true;
          this.luckshow = false;
          this.$refs.drawer.open("center");
          return;
        }
      } else if (this.priceCode == -1) {
        //错误提示
        this.showMsg = true;
        this.luckshow = false;
        this.$refs.drawer.open("center");
        return;
      } else if (this.priceCode == 4) {
        this.showlingqu = true;
        this.luckshow = false;
        this.$refs.drawer.open("center");
        return;
      }

      uni
        .request({
          url:
            "https://e2ee.zdfan.com/api/fpluck_wex?code=" +
            this.code +
            "&openid=" +
            this.openid,
        })
        .then((data) => {
          //data为一个数组数组第一项为错误信息即为fail回调第二项为返回数据
          var [err, res] = data;
          // console.log(res.data);
          this.priceCode = res.data.code;
          this.index = res.data.index;
          this.msg = res.data.msg;
          this.tkl = res.data.tkl;
          this.qrCodeImg = res.data.qrimg;
          this.lucktype = res.data.lucktype;

          //如果tkl为undefined则赋值为空
          if (this.tkl == undefined) {
            this.tkl = "";
          }

          if (this.lucktype == undefined) {
            this.lucktype = -1;
          }

          if (this.index == undefined) {
            this.index = 2;
          }
        });
      // 先开始旋转
      this.$refs.myLucky.play();
      // 使用定时器来模拟请求接口
      setTimeout(() => {
        //设置中奖索引为随机0-7
        // const index = Math.floor(Math.random() * 8);
        // 调用stop停止旋转并传递中奖索引
        // console.log(this.index);
        this.$refs.myLucky.stop(this.index);
      }, 3000);
    },
    // 抽奖结束触发回调
    endCallBack(prize) {
      this.getjfnum();
      // 奖品详情
      if (this.priceCode == 1) {
        //用户未中奖
        this.msg = "用户未中奖";
        this.shownoToast = true;
        this.luckshow = false;
        this.$refs.drawer.open("center");
      } else if (this.priceCode == 2) {
        //中奖了未加好友
        this.msg = "请添加好友领取奖品";
        this.showqrToast = true;
        this.luckshow = false;
        this.$refs.drawer.open("center");
      } else if (this.priceCode == 3) {
        //中奖了已加好友
        this.msg = "已添加好友，复制口令成功";
        //如果tkl文本大于40个字符this.showwxpay = true否则this.showToast = true
        if (this.tkl.length == 44) {
          this.showjfpay = true;
          this.luckshow = false;
          this.$refs.drawer.open("center");
          // return;
        } else if (this.tkl.length == 64) {
          this.showwxpay = true;
          this.luckshow = false;
          this.$refs.drawer.open("center");
          // return;
        } else {
          this.showToast = true;
          this.luckshow = false;
          this.$refs.drawer.open("center");
          // return;
        }
      } else if (this.priceCode == 4) {
        this.showlingqu = true;
        this.luckshow = false;
        this.$refs.drawer.open("center");
      }
    },
    getluckinfo() {
      //参数验证
      //如果没有code，则赋值为空
      if (!this.code) {
        this.code = "";
      }
      //如果没有openid，则赋值为空
      if (!this.openid) {
        this.openid = "";
      }
      //如果没有unionid，则赋值为空
      if (!this.unionid) {
        this.unionid = "";
      }

      //如果code长度小于5，或者openid长度小于5，或者unionid长度小于5，则跳转到首页
      if (
        this.code.length < 5 ||
        this.openid.length < 5 ||
        this.unionid.length < 5
      ) {
        // console.log("参数异常");
        this.msg = "请从群内点击链接进入小程序即可参与抽奖！";
        this.showMsg = true;
        this.luckshow = false;
        this.$refs.drawer.open("center");
        return;
      }

      uni.request({
        url:
          "https://e2ee.zdfan.com/api/fwluck_wex?code=" +
          this.code +
          "&openid=" +
          this.openid,
        method: "GET",
        success: (res) => {
          // console.log(res);
          this.msg = res.data.msg;
          this.title = res.data.title;
          this.priceCode = res.data.code;

          this.lucktype = res.data.lucktype;
          this.index = res.data.index;
          this.qrCodeImg = res.data.qrimg;
          this.tkl = res.data.tkl;
          this.jfnum = res.data.jfnum;

          if (this.priceCode == 0) {
            this.luckshow = true;
            //判断lucktype
            if (this.lucktype == 0) {
              this.shownoToast = true;
              this.luckshow = false;
              this.$refs.drawer.open("center");
            } else if (this.lucktype == 2) {
              this.showqrToast = true;
              this.luckshow = false;
              this.$refs.drawer.open("center");
            } else if (this.lucktype == 3) {
              //如果tkl文本大于40个字符this.showwxpay = true否则this.showToast = true
              if (this.tkl.length == 44) {
                this.showjfpay = true;
                this.luckshow = false;
                this.$refs.drawer.open("center");
                // return;
              } else if (this.tkl.length == 64) {
                this.showwxpay = true;
                this.luckshow = false;
                this.$refs.drawer.open("center");
                // return;
              } else {
                this.showToast = true;
                this.luckshow = false;
                this.$refs.drawer.open("center");
                // return;
              }
            } else if (this.lucktype == 4) {
              this.showlingqu = true;
              this.luckshow = false;
              this.$refs.drawer.open("center");
            }

            //获取res.data.luck数组遍历输出
            for (let i = 0; i < res.data.luck.length; i++) {
              if (i == 0) {
                this.prizes.push({
                  x: 0,
                  y: 0,
                  imgs: [
                    {
                      src: res.data.luck[i].img,
                      width: "53%",
                      top: "8%",
                    },
                  ],
                  fonts: [
                    {
                      text: res.data.luck[i].name,
                      top: "70%",
                    },
                  ],
                });
              } else if (i == 1) {
                this.prizes.push({
                  x: 1,
                  y: 0,
                  imgs: [
                    {
                      src: res.data.luck[i].img,
                      width: "53%",
                      top: "8%",
                    },
                  ],
                  fonts: [
                    {
                      text: res.data.luck[i].name,
                      top: "70%",
                    },
                  ],
                });
              } else if (i == 2) {
                this.prizes.push({
                  x: 2,
                  y: 0,
                  imgs: [
                    {
                      src: res.data.luck[i].img,
                      width: "53%",
                      top: "8%",
                    },
                  ],
                  fonts: [
                    {
                      text: res.data.luck[i].name,
                      top: "70%",
                    },
                  ],
                });
              } else if (i == 3) {
                this.prizes.push({
                  x: 2,
                  y: 1,
                  imgs: [
                    {
                      src: res.data.luck[i].img,
                      width: "53%",
                      top: "8%",
                    },
                  ],
                  fonts: [
                    {
                      text: res.data.luck[i].name,
                      top: "70%",
                    },
                  ],
                });
              } else if (i == 4) {
                this.prizes.push({
                  x: 2,
                  y: 2,
                  imgs: [
                    {
                      src: res.data.luck[i].img,
                      width: "53%",
                      top: "8%",
                    },
                  ],
                  fonts: [
                    {
                      text: res.data.luck[i].name,
                      top: "70%",
                    },
                  ],
                });
              } else if (i == 5) {
                this.prizes.push({
                  x: 1,
                  y: 2,
                  imgs: [
                    {
                      src: res.data.luck[i].img,
                      width: "53%",
                      top: "8%",
                    },
                  ],
                  fonts: [
                    {
                      text: res.data.luck[i].name,
                      top: "70%",
                    },
                  ],
                });
              } else if (i == 6) {
                this.prizes.push({
                  x: 0,
                  y: 2,
                  imgs: [
                    {
                      src: res.data.luck[i].img,
                      width: "53%",
                      top: "8%",
                    },
                  ],
                  fonts: [
                    {
                      text: res.data.luck[i].name,
                      top: "70%",
                    },
                  ],
                });
              } else if (i == 7) {
                this.prizes.push({
                  x: 0,
                  y: 1,
                  imgs: [
                    {
                      src: res.data.luck[i].img,
                      width: "53%",
                      top: "8%",
                    },
                  ],
                  fonts: [
                    {
                      text: res.data.luck[i].name,
                      top: "70%",
                    },
                  ],
                });
              }
            }
            // console.log(this.prizes);
          } else if (this.priceCode == -2) {
            this.msg =
              "当前活动仅限群内成员参与，如您在群内重新点击抽奖链接进入";
            //弹出错误提示
            this.luckshow = false;
            this.showMsg = true;
            this.$refs.drawer.open("center");
          } else {
            this.luckshow = false;
            this.showMsg = true;
            this.$refs.drawer.open("center");
          }
        },
        fail: (err) => {
          console.log(err);
        },
      });
    },
    copytkl() {
      //弹出信息框
      uni.setClipboardData({
        data: this.tkl,
        success: (res) => {
          uni.showToast({
            title: "复制成功,打开淘宝APP即可领取",
            icon: "none",
            duration: 5000,
          });
        },
        fail: (err) => {
          uni.showToast({
            title: "复制失败,请重试",
            icon: "none",
            duration: 5000,
          });
        },
      });
    },
    linkQrcode() {
      this.showqrToast = false;
      this.showQrcode = true;
    },
    ptlingqu() {
      // console.log("ptlingqu");
      //参数验证
      //如果没有code，则赋值为空
      if (!this.code) {
        this.code = "";
      }
      //如果没有openid，则赋值为空
      if (!this.openid) {
        this.openid = "";
      }
      //如果没有unionid，则赋值为空
      if (!this.unionid) {
        this.unionid = "";
      }

      //如果code长度小于5，或者openid长度小于5，或者unionid长度小于5，则跳转到首页
      if (
        this.code.length < 5 ||
        this.openid.length < 5 ||
        this.unionid.length < 5
      ) {
        // console.log("参数异常");
        this.msg = "请从群内点击链接进入小程序即可参与抽奖！";
        this.showMsg = true;
        this.luckshow = false;
        this.$refs.drawer.open("center");
        return;
      }
      //参数验证完成，发送普通请求
      uni
        .request({
          url:
            "https://e2ee.zdfan.com/api/ptluck_wex?code=" +
            this.code +
            "&openid=" +
            this.openid,
        })
        .then((data) => {
          //data为一个数组数组第一项为错误信息即为fail回调第二项为返回数据
          //目前只有2种情况 -1解析msg 0解析index
          var [err, res] = data;
          // console.log(res.data);
          this.priceCode = res.data.code;
          this.msg = res.data.msg;
          this.index = res.data.index;
          this.tkl = res.data.tkl;

          this.beilv = res.data.beilv;
          this.jpnum = res.data.jpnum;
          this.zznum = res.data.zznum;
          //以上3个参数如果为空，则赋值为空
          if (!this.beilv) {
            this.beilv = "";
          }
          if (!this.jpnum) {
            this.jpnum = "";
          }
          if (!this.zznum) {
            this.zznum = "";
          }

          if (!this.tkl) {
            this.tkl = "";
          }

          if (this.priceCode == -1) {
            this.showMsg = true;
            this.luckshow = false;
            this.$refs.drawer.open("center");
          } else {
            //如果tkl文本大于40个字符this.showwxpay = true否则this.showToast = true
            if (this.tkl.length == 44) {
              this.showjfpay = true;
              this.luckshow = false;
              this.$refs.drawer.open("center");
              // return;
            } else if (this.tkl.length == 64) {
              this.showwxpay = true;
              this.luckshow = false;
              this.$refs.drawer.open("center");
              // return;
            } else {
              this.showToast = true;
              this.luckshow = false;
              this.$refs.drawer.open("center");
              // return;
            }
            //设置lucktype为3
            this.lucktype = 3;
          }
        });
      this.showad();
    },
    suplingqu() {
      // console.log("suplingqu");
      //参数验证
      //如果没有code，则赋值为空
      if (!this.code) {
        this.code = "";
      }
      //如果没有openid，则赋值为空
      if (!this.openid) {
        this.openid = "";
      }
      //如果没有unionid，则赋值为空
      if (!this.unionid) {
        this.unionid = "";
      }

      //如果code长度小于5，或者openid长度小于5，或者unionid长度小于5，则跳转到首页
      if (
        this.code.length < 5 ||
        this.openid.length < 5 ||
        this.unionid.length < 5
      ) {
        // console.log("参数异常");
        this.msg = "请从群内点击链接进入小程序即可参与抽奖！";
        this.showMsg = true;
        this.luckshow = false;
        this.$refs.drawer.open("center");

        return;
      }
      //参数验证完成，发送普通请求
      uni
        .request({
          url:
            "https://e2ee.zdfan.com/api/supluck_wex?code=" +
            this.code +
            "&openid=" +
            this.openid,
        })
        .then((data) => {
          //data为一个数组数组第一项为错误信息即为fail回调第二项为返回数据
          //目前只有2种情况 -1解析msg 0解析index
          var [err, res] = data;
          // console.log(res.data);
          this.priceCode = res.data.code;
          this.msg = res.data.msg;
          this.index = res.data.index;
          this.tkl = res.data.tkl;

          this.jptype = res.data.jptype;
          this.beilv = res.data.beilv;
          this.jpnum = res.data.jpnum;
          this.zznum = res.data.zznum;
          //以上4个参数如果为空，则赋值为空
          if (!this.jptype) {
            this.jptype = 0;
          }
          if (!this.beilv) {
            this.beilv = "";
          }
          if (!this.jpnum) {
            this.jpnum = "";
          }
          if (!this.zznum) {
            this.zznum = "";
          }

          if (!this.tkl) {
            this.tkl = "";
          }

          if (this.priceCode == -1) {
            this.showMsg = true;
            this.luckshow = false;
            this.$refs.drawer.open("center");
          } else {
            this.showsup = true;
            //如果tkl文本大于40个字符this.showwxpay = true否则this.showToast = true
            if (this.tkl.length == 44) {
              this.showjfpay = true;
              this.luckshow = false;
              this.$refs.drawer.open("center");
              // return;
            } else if (this.tkl.length == 64) {
              this.showwxpay = true;
              this.luckshow = false;
              this.$refs.drawer.open("center");
              // return;
            } else {
              this.showToast = true;
              this.luckshow = false;
              this.$refs.drawer.open("center");
              // return;
            }
            //设置lucktype为3
            this.lucktype = 3;
          }
        });
    },
    dhluck() {
      uni.reLaunch({
        url: "../integral/integral",
      });
    },
    getjfnum(isshow) {
      uni
        .request({
          url:
            "https://wk.zdfan.com/userapi.php?do=getjifendb2&openid=" +
            this.openid,
        })
        .then((data) => {
          var [err, res] = data;
          //根据res.data.jfnum/100则为账号余额
          this.jfnum = res.data.jfnum/100;
          if (isshow == 1) {
            uni.showToast({
              title: "刷新完成",
              icon: "success",
              duration: 2000,
            });
          }
        });
    },
    showad() {
      //等待1秒后显示广告
      setTimeout(function () {
        if (interstitialAd) {
          interstitialAd.show().catch((err) => {
            console.error(err);
          });
        }
      }, 1000);
    },
  },
};

export function getCurPage() {
  let pages = getCurrentPages();
  let curPage = pages[pages.length - 1];
  // console.log(curPage.options);
  return curPage;
}

// 获取页面URL参数
export function getLocationParams(name) {
  //获取页面栈
  const pages = getCurrentPages();
  //获取路由参数
  const curPage = pages[pages.length - 1];
  return name ? curPage.options[name] : curPage.options;
}
</script>

<style lang="scss" scoped>
.headad {
  height: 280rpx;
  width: 100vw;
}

.tips {
  font-size: 32rpx;
  color: #0933ef;
  text-align: center;
  //设置上边距为10px
  margin-top: 10rpx;
}

.anniu {
  //设置上边距为10px
  margin-top: 10rpx;
  //两个按钮放在一行
  display: flex;
  //两个按钮水平居中
  justify-content: center;
}

.luck {
  //设置上边距为屏幕的2/8
  margin-top: 40rpx;
}

.getjf {
  background-color: #1aad19;
  color: #fff;
  //宽度为屏幕的2/3
  margin-top: 5rpx;
  width: 30vw;
  font-size: 30rpx;
}

.dhluck {
  background-color: #1aad19;
  color: #fff;
  //宽度为屏幕的2/3
  margin-top: 5rpx;
  width: 30vw;
  font-size: 30rpx;
}

.lottery-alert {
  background: white;
  padding: 20rpx;
  border-radius: 20rpx;
  text-align: center;

  .alert-header {
    font-size: 30rpx;
    color: red;
    margin-bottom: 20rpx;
  }

  .tip {
    color: red;
    margin: 20rpx 0;
  }

  .suptip {
    color: rgb(43, 15, 204);
    margin: 20rpx 0;
    //文本加粗
    font-weight: bold;
  }

  .btnsave {
    background-color: red;
    height: 60rpx;
    width: 100%;
    color: white;
    font-size: 30rpx;
    line-height: 60rpx;
    text-align: center;
    border-radius: 5rpx;
  }

  .btndef {
    background-color: rgb(93, 89, 89);
    height: 120rpx;
    width: 48%;
    color: white;
    font-size: 30rpx;
    line-height: 60rpx;
    text-align: center;
    border-radius: 5rpx;
  }

  .btnsup {
    background-color: rgb(7, 41, 234);
    height: 120rpx;
    width: 48%;
    color: rgb(246, 246, 251);
    font-size: 30rpx;
    line-height: 60rpx;
    text-align: center;
    border-radius: 5rpx;
  }
  .bts {
    //把两个按钮放在一行
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
  }
}
</style>