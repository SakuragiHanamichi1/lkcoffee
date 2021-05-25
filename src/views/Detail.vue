<template>
  <div class="detail">
    <!-- 头部导航模块 -->
    <van-nav-bar
      title="商品详情"
      left-text="返回"
      left-arrow
      @click-left="onClickLeft"
      @click-right="showShare = true"
    >
      <template #right>
        <van-icon name="ellipsis" size="20" />
      </template>
    </van-nav-bar>
    <!-- 分享面板 -->
    <van-share-sheet
      v-model="showShare"
      title="立即分享给好友"
      :options="options"
      @select="onSelect"
    >
    </van-share-sheet>

    <!-- 商品详情模块 -->
    <div>
      <div>
        <img class="auto_img" :src="productDetail.large_img" />
      </div>
      <div class="pro_info_box">
        <div class="pro_info">
          <div class="pro_box clearfix">
            <div class="fl">
              <div class="pro_name">{{ productDetail.name }}</div>
              <div class="pro_enname">{{ productDetail.enname }}</div>
            </div>
            <div class="pro_price">￥{{ productDetail.price }}</div>
          </div>

          <!-- 商品规格 -->
          <div class="rule_box">
            <div
              class="rule_item clearfix"
              v-for="(item, index) in productDetail.rules"
              :key="index"
            >
              <div class="fl rule_title">{{ item.title }}</div>
              <div class="fl">
                <div
                  class="fl r_item"
                  :class="{
                    active: item.ruleIndex == i,
                    'large-text': v.title.length >= 4,
                  }"
                  v-for="(v, i) in item.rule"
                  :key="i"
                  @click="toggleRule(item, i)"
                >
                  {{ v.title }}
                </div>
              </div>
            </div>
          </div>

          <!-- 修改商品数量 -->
          <div class="c_box clearfix">
            <div class="select_count fl">选择数量</div>
            <div class="count_box fr">
              <van-stepper
                v-model="product.count"
                theme="round"
                button-size="22"
                disable-input
              />
            </div>
          </div>

          <!-- 商品描述 -->
          <div class="desc_box">
            <div class="pro_desc">商品描述</div>
            <div class="desc_text">
              <div
                class="text_item"
                v-for="(item, index) in productDetail.desc"
                :key="index"
              >
                {{ index + 1 }}、{{ item }}
              </div>
            </div>
          </div>

          <!-- ************************************************************************************************ -->
        </div>
      </div>
    </div>

    <!-- 底部收藏 购买模块 -->
    <van-goods-action>
      <van-goods-action-icon
        :to="{ name: 'Shopbag' }"
        icon="bag"
        text="购物袋"
        :badge="bagCount == 0 ? '' : bagCount"
        :color="bagCount > 0 ? '#00adb5' : '#646566'"
      />
      <van-goods-action-icon
        icon="like"
        :text="isLike ? '已收藏' : '未收藏'"
        :color="isLike ? '#00adb5' : '#646566'"
        @click="toggleLikeProduct"
      />
      <van-goods-action-button
        text="加入购物袋"
        color="#3fc1c9"
        @click="addShopbag()"
      />
      <van-goods-action-button
        text="立即购买"
        color="#00adb5"
        @click="buy(true)"
      />
    </van-goods-action>
  </div>
</template>

<script>
import "../assets/less/detail.less";

export default {
  name: "Detail",
  data() {
    return {
      //商品pid
      pid: "",

      //商品信息
      product: {
        count: 1,
      },

      //购物袋的数量
      bagCount: 0,

      //商品详情数据
      productDetail: {},

      //是否已经收藏商品
      isLike: false,

      // 分享面板
      showShare: false,
      options: [
        [
          { name: "微信", icon: "wechat" },
          { name: "微博", icon: "weibo" },
          { name: "QQ", icon: "qq" },
          { name: "QQ", icon: "qq" },
        ],
        [
          { name: "复制链接", icon: "link" },
          { name: "分享海报", icon: "poster" },
          { name: "二维码", icon: "qrcode" },
          { name: "二维码", icon: "qrcode" },
        ],
      ],
    };
  },

  created() {
    //截取pid
    this.pid = this.$route.params.pid;

    //获取商品详情数据
    this.getProductDetail();

    //查询用户是否已经收藏该商品
    this.findLike();

    //查询用户的购物袋记录数
    this.shopBagCount();
  },

  methods: {
    // NavBar导航模块
    onClickLeft() {
      this.$router.back(-1);
    },
    onSelect(option) {
      Toast(option.name);
      this.showShare = false;
    },

    //获取商品详情数据
    getProductDetail() {
      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
      });

      //发起注册请求
      this.axios({
        //请求类型
        method: "GET",
        //请求路径
        url: "/productDetail",

        //GET请求参数, object
        params: {
          appkey: this.appkey,
          pid: this.pid,
        },
      })
        .then((result) => {
          this.$toast.clear();

          if (result.data.code == 600) {
            let data = result.data.result[0];

            data.desc = data.desc.split(/\n/);

            let rules = [];

            let ruleData = ["tem", "sugar", "milk", "cream"];
            ruleData.map((v) => {
              let r = {};
              r.title = data[v + "_desc"];

              //激活下标
              r.ruleIndex = 0;

              //每项规格
              r.rule = [];

              //获取规格字符串
              let ruleString = data[v].split("/");
              ruleString.map((value) => {
                if (value != "") {
                  r.rule.push({ title: value });
                }
              });

              if (r.rule.length > 0) {
                rules.push(r);
              }
            });

            data.rules = rules;

            this.productDetail = data;
          }
        })
        .catch((err) => {
          this.$toast.clear();
        });
    },

    //返回上一级
    back() {
      this.$router.go(-1);
    },

    //切换规格
    toggleRule(item, i) {
      //
      //
      if (item.ruleIndex == i) {
        return;
      }

      item.ruleIndex = i;
    },

    //查询用户是否收藏该商品
    findLike() {
      //获取token
      let tokenString = localStorage.getItem("__tk");
      //
      if (!tokenString) {
        return;
      }

      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
      });

      //发起查询收藏商品请求
      this.axios({
        method: "GET",
        url: "/findlike",
        params: {
          appkey: this.appkey,
          pid: this.pid,
          tokenString,
        },
      })
        .then((result) => {
          this.$toast.clear();

          if (result.data.code == 1000) {
            if (result.data.result.length > 0) {
              //收藏商品成功
              this.isLike = true;
            }
          }
        })
        .catch((err) => {
          this.$toast.clear();
        });
    },

    //收藏商品
    toggleLikeProduct() {
      //获取token
      let tokenString = localStorage.getItem("__tk");
      //
      if (!tokenString) {
        //跳回登录页面
        this.$toast("请先登录");
        return this.$router.push({ name: "Login" });
      }

      let url = this.isLike ? "/notlike" : "/like";

      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
      });

      //发起收藏商品请求
      this.axios({
        method: "POST",
        url,
        data: {
          appkey: this.appkey,
          pid: this.pid,
          tokenString,
        },
      })
        .then((result) => {
          this.$toast.clear();

          if (result.data.code == 700) {
            //token检验无效,则跳到登录页面
            this.$router.push({ name: "Login" });
          } else if (result.data.code == 800) {
            //收藏商品成功
            this.isLike = true;
          } else if (result.data.code == 900) {
            //取消收藏商品
            this.isLike = false;
          }

          this.$toast(result.data.msg);
        })
        .catch((err) => {
          this.$toast.clear();
        });
    },

    //查询用户的购物袋数量
    shopBagCount() {
      let tokenString = localStorage.getItem("__tk");
      //
      if (!tokenString) {
        return;
      }

      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
      });

      this.axios({
        method: "GET",
        url: "/shopcartRows",
        params: {
          appkey: this.appkey,
          tokenString,
        },
      })
        .then((result) => {
          this.$toast.clear();

          if (result.data.code == 8000) {
            this.bagCount = result.data.result;
          }
        })
        .catch((err) => {
          this.$toast.clear();
        });
    },

    //加入购物袋
    addShopbag(isBuy) {
      //获取tokenString
      //获取商品pid
      //获取商品规格rule
      //获取商品数量count
      //获取appkey

      let tokenString = localStorage.getItem("__tk");
      //
      if (!tokenString) {
        //跳回登录页面
        this.$toast("请先登录");
        return this.$router.push({ name: "Login" });
      }

      //请求参数
      let data = {
        tokenString,
        appkey: this.appkey,
        count: this.product.count,
        pid: this.pid,
      };

      let rs = [];
      //遍历productDetail.rules数组获取选择的商品规格
      this.productDetail.rules.map((v) => {
        rs.push(v.rule[v.ruleIndex].title);
      });

      data.rule = rs.join("/");

      //
      this.$toast.loading({
        message: "加载中...",
        forbidClick: true,
        duration: 0,
      });
      this.axios({
        method: "POST",
        url: "/addShopcart",
        data,
      })
        .then((result) => {
          this.$toast.clear();

          if (result.data.code == 700) {
            //token检验无效,则跳到登录页面
            this.$router.push({ name: "Login" });
          } else if (result.data.code == 3000) {
            if (!isBuy) {
              //加入购物袋
              if (result.data.status == 1) {
                this.bagCount++;
              }
            } else {
              //立即购买
              this.$router.push({
                name: "Pay",
                query: { sids: result.data.sid },
              });
            }
          }

          this.$toast(result.data.msg);
        })
        .catch((err) => {
          this.$toast.clear();
        });
    },

    //立即购买(后台判断购物袋如果存在该商品，则累加数量，然后跳转到订单结算页面，否则先将该商品加入购物袋，再跳转到订单结算页面)
    buy(isBuy) {
      this.addShopbag(isBuy);
    },
  },
};
</script>
