<template>
  <div
    class="order-container"
    @click="handle_orderRoute(order_type(+order_item.id))"
  >
    <header class="header">
      <div class="header-top-left">
        <van-tag mark type="primary" v-if="order_type(+order_item.id)">
          <span>{{ $t("views.gather.undone.sell") }}</span>
        </van-tag>
        <van-tag mark type="warning" v-else>
          <span>{{ $t("views.gather.undone.purchase") }}</span>
        </van-tag>
        &nbsp;&nbsp;
        <span class="header-top-left-test">{{kind}}</span>
        &nbsp;&nbsp;
        <van-tag
          v-if="order_type(+order_item.id) && initTime() === 0"
          color="#ffe1e1"
          @click.stop="Verify_Coin"
          text-color="#ad0000"
          >{{ $t("views.gather.undone.verify") }}
        </van-tag>
      </div>
      <p class="header-top" v-if="initTime() === 0">
        <span class="pending_coin" v-if="order_type(+order_item.id)">
          {{ $t("views.gather.undone.Velocity") }}&nbsp;&nbsp;
        </span>
        <span class="pending_coin" v-else>
          {{ $t("views.gather.undone.payment") }}&nbsp;&nbsp;</span
        >

        <van-count-down
          v-if="order_type(+order_item.id)"
          :style="{ color: 'crimson' }"
          ref="order_countdown"
          @finish="coutDown_finish(1)"
          :time="order_countdown"
          format="mm:sss"
        />
        <van-count-down
          v-else
          :style="{ color: 'crimson' }"
          ref="order_countdown"
          @finish="coutDown_finish(0)"
          :time="order_countdown"
          format="mm:sss"
        />
      </p>
      <p
        class="header-top"
        v-else-if="order_type(+order_item.id) && initTime() === 2"
      >
        <span class="header-topRight">
          {{ $t("views.gather.undone.Stay") }}</span
        >
      </p>
      <p
        class="header-top"
        v-else-if="order_type(+order_item.id) && initTime() === 1"
      >
        <span class="header-topRight">
          {{ $t("views.gather.undone.wait") }}</span
        >
      </p>

      <p
        class="header-top"
        v-else-if="!order_type(+order_item.id) && initTime() === 1"
      >
        <span class="header-topRight">{{ $t("views.gather.undone.put") }}</span>
      </p>
    </header>

    <main class="main">
      <ul>
        <li class="main-top">
          <p>
            <span
              :style="{ color: '#000' }"
              @click.stop.self="
                handle_showPhone(order_item.id, $t('views.gather.undone.copy'))
              "
              >{{ $t("views.gather.undone.number") }}:{{ order_item.id }}
            </span>
            <span :style="{ color: '#000' }">{{ $t("views.gather.undone.price") }}:{{ order_item.cny }}</span>
            <span>{{ $t("views.gather.undone.count") }}:{{ Number(order_item.num).toFixed(2) }}</span>
            <span>{{ $t("views.gather.undone.service_charge") }}:{{order_item.amount2 }}:{{ order_item.amount2 }} {{kind}}</span>
          </p>
          <p>
            <span> </span>
            <span></span>
            <span class="total_price">{{ order_item.amount1 }} CNY</span>
            <span>{{ $t("views.gather.undone.total") }}</span>
          </p>
        </li>

        <li class="showFullInfo" v-if="!isShowFullInfo">
          <span></span>
          <span class="showFullInfo-btn" @click.stop="showFullInfo">
            {{ $t("views.gather.undone.show") }}
            <van-icon name="arrow-down" />
          </span>
        </li>

        <li v-if="isShowFullInfo">
          <span> {{ $t("views.gather.undone.name") }}</span>
          <span>{{ order_item.mes }}</span>
        </li>
        <li v-if="isShowFullInfo">
          <span> {{ $t("views.gather.undone.way") }}</span>
          <span>
            <b v-if="!isShowPhone" class="mask">{{ order_item.wechat }}</b>
            <a v-else :href="'tel:' + order_item.wechat">{{
              order_item.wechat
            }}</a>
            <van-icon
              @click.stop.self="
                handle_showPhone(order_item.wechat, $t('views.gather.undone.phone'))
              "
              name="eye-o"
              class="eye-o"
            />
          </span>
        </li>
        <li class="payInfo" v-if="isShowFullInfo && order_item.dsx === '2'">
          <!-- 接受现金 -->
          <template
            v-if="is_xj == -1 || (is_xj === 1 && xjstr === '可现金交易')"
          >
            <span :style="{ margin: '10px 0' }">付款方式</span>
            <span
              v-if="pay_Info.includes('交易')"
              :style="{
                display: 'flex',
                alignItems: 'center',
                margin: '10px 0',
              }"
            >
              <img
                class="xj_moeny"
                src="@/assets/currency-icons/moeny-c.png"
                alt="xj"
              />
              &nbsp;&nbsp;
              <span>{{ $t("views.gather.undone.cash") }}</span>
            </span>
            <span
              :style="{ margin: '10px 0' }"
              v-else-if="pay_Info[2].trim() === '支付宝'"
            >
              <i
                v-for="(pay, i) in payList.slice(0, 1)"
                :key="i"
                :class="[
                  'iconfont',
                  `icon-zfb`,
                  `pay-icon1zfb1`,
                  `pay-iconzfb`,
                ]"
              ></i
              >&nbsp;&nbsp;
              <span class="pay-txt">{{
                $t("views.gather.undone.alipay")
              }}</span>
            </span>
            <span
              :style="{ margin: '10px 0' }"
              v-else-if="pay_Info[2].trim() === '微信'"
            >
              <i
                :class="['iconfont', `icon-wx`, `pay-icon1wx2`, `pay-iconwx`]"
              ></i
              >&nbsp;&nbsp;
              <span class="pay-txt">{{
                $t("views.gather.undone.wechat")
              }}</span>
            </span>
            <span :style="{ margin: '10px 0' }" v-else>
              <i
                :class="[
                  'iconfont',
                  `icon-yhk`,
                  `pay-icon1yhk3`,
                  `pay-iconyhk`,
                ]"
              ></i
              >&nbsp;&nbsp;
              <span class="pay-txt">{{ pay_Info[2].trim() }} </span>
            </span>
          </template>

          <template v-if="is_xj === 0">
            <span :style="{ margin: '10px 0' }">付款方式</span>
            <span
              :style="{ margin: '10px 0' }"
              v-if="pay_Info[2].trim() === '支付宝'"
            >
              <i
                v-for="(pay, i) in payList.slice(0, 1)"
                :key="i"
                :class="[
                  'iconfont',
                  `icon-zfb`,
                  `pay-icon1zfb1`,
                  `pay-iconzfb`,
                ]"
              ></i
              >&nbsp;&nbsp;
              <span class="pay-txt">{{
                $t("views.gather.undone.alipay")
              }}</span>
            </span>
            <span
              :style="{ margin: '10px 0' }"
              v-else-if="pay_Info[2].trim() === '微信'"
            >
              <i
                :class="['iconfont', `icon-wx`, `pay-icon1wx2`, `pay-iconwx`]"
              ></i
              >&nbsp;&nbsp;
              <span class="pay-txt">{{
                $t("views.gather.undone.wechat")
              }}</span>
            </span>
            <span :style="{ margin: '10px 0' }" v-else>
              <i
                :class="[
                  'iconfont',
                  `icon-yhk`,
                  `pay-icon1yhk3`,
                  `pay-iconyhk`,
                ]"
              ></i
              >&nbsp;&nbsp;
              <span class="pay-txt">{{ pay_Info[2].trim() }} </span>
            </span>
          </template>
        </li>

        <template v-if="is_xj === 0 || (is_xj === 1 && xjstr === '可现金交易')">
          <li v-if="isShowFullInfo && order_item.dsx === '2'">
            <span>{{ $t("views.gather.undone.account") }}</span>
            <span
              :style="{ margin: '10px 0' }"
              v-if="pay_Info.includes('交易')"
            >
              {{ pay_Info }}
            </span>

            <span
              :style="{ margin: '10px 0' }"
              v-else-if="pay_Info[2].trim() === '支付宝'"
            >
              {{ pay_Info[1].trim() }}
            </span>
            <span
              :style="{ margin: '10px 0' }"
              v-else-if="pay_Info[2].trim() === '微信'"
            >
              {{ pay_Info[1].trim() }}
            </span>
            <span :style="{ margin: '10px 0' }" v-else>
              {{ pay_Info[1].trim() }}
            </span>
          </li>
        </template>
      </ul>
    </main>
    <footer class="footer">
      <p>
        <span>{{ order_item.sname }}</span
        >&nbsp;
        <img class="info-rz" src="@/assets/currency-icons/rz.png" alt="" />
      </p>
      <p>
        {{ order_item.eotc | transformTime_MDMS }}
      </p>
    </footer>
    <!-- start  收款方式 提示弹窗 -->
    <van-popup
      v-model="showOrderSaleInfo"
      :close-on-popstate="true"
      get-container="body"
      position="bottom"
      class="saleInfo"
      :close-on-click-overlay="isclose_on_click_overlay"
      :style="{ height: '350px' }"
    >
      <template>
        <div>
          <header class="header">
            <van-icon name="arrow-left" />
            <span class="header-text">{{
              $t("views.gather.undone.select")
            }}</span>
          </header>
          <main class="main">
            <template
              v-if="is_xj === -1 || (is_xj === 1 && xjstr === '可现金交易')"
            >
              <van-cell center @click="handlePayChange('xj')">
                <template #title>
                  <span class="custom-title"
                    >&nbsp;{{ $t("views.gather.undone.cash_deal") }}</span
                  >
                </template>
                <template #icon>
                  <img
                    class="xj_moeny"
                    src="@/assets/currency-icons/moeny-c.png"
                    alt="xj"
                  />
                </template>
              </van-cell>
            </template>

            <template
              v-if="is_xj === 0 || (is_xj === 1 && xjstr === '可现金交易')"
            >
              <van-cell
                center
                v-for="(payType, i) in playList"
                :key="i"
                @click="handlePayChange(payType)"
              >
                <template #title>
                  <span class="custom-title">{{ getpayType(payType) }}</span>
                </template>
                <template #icon>
                  <van-icon>
                    <i
                      :class="[
                        'iconfont',
                        `icon-${payType}`,
                        `pay-icon${payType}`,
                      ]"
                    ></i>
                  </van-icon>
                </template>
                <template #right-icon> </template>
              </van-cell>
            </template>

            <div class="salePay-info">
              <span class="span1"
                >*<i class="zy-info">{{ $t("views.gather.undone.self") }}</i
                >{{ $t("views.gather.undone.generate") }}</span
              >
            </div>
          </main>
        </div>
      </template>
    </van-popup>
    <!-- end 出售 详细信息  收款方式 提示弹窗  -->
  </div>
</template>

<script>
import { subbuysellorder } from "@/api/trxRequest";
import { getItem, removeItem } from "@/utils/storage";
import { myPayment, Getsjmes } from "@/api/payverification";
import { Buy_cancel } from "@/utils/web3";
import sell_Mixin from "@/mixins/sell_mixins";

import { TemporaryCoinWithdrawal } from "./getCoin_users";

export default {
  name: "order-info",
  props: {
    order_item: {
      require: true,
      type: [Object],
    },
    kind:{
      type:[String]
    }
  },
  mixins: [sell_Mixin],
  created() {
    //console.log(this.order_item);
  },
  methods: {
    // 倒计时完成 订单自动取消
    async coutDown_finish(type) {
      this.$toast.warning(this.$t("views.gather.undone.toast[0]"));
      //console.log(type);
      if (this.order_item.rcoin == 1)
        await Buy_cancel(
          this.order_item.id,
          localStorage.getItem("userIconId")
        );
      subbuysellorder({
        oid: this.order_item.id,
        type,
        selectpayk: "",
      }).then(() => {
        this.$toast.warning(
          <p style="color:red;font-size:16px">
            {this.$t("views.gather.undone.toast[2]")}
          </p>
        );
        if (type === 0) {
          localStorage.setItem("xdnum", "0");
        } else if (type === 1) {
          localStorage.setItem("csnum", "0");
        }
      });
      this.$emit("update_OrderList");
      removeItem(this.order_item.id);
    },
    initTime() {
      let time = this.trsfTime_30timeout(this.order_item.eotc, 30);
      const diff_stime = this.diff_30timeout(
        time,
        this.transformTime_Zh(Date.now())
      );
      this.order_countdown = diff_stime;

      if (this.order_item.dsx == "2") {
        return 2; // dsx==2 商家已付款，待放币，
      } else if (this.order_item.dsx == "0") {
        return 0; // dsx==1 未转币到合约
      } else {
        return 1; //转币到合约，但商家未付款
      }
    },
    async handle_orderRoute(order_Type) {
      console.log(111)
      if (!order_Type) {
        console.log("用户购买", order_Type);
        this.buyer_orderType();
      } else {
        //用户出售
        const $order = this.order_item;

        console.log($order)
        switch ($order.dsx) {
          case "2": {
            let data;
            try {
              data = await Getsjmes($order.odid);
            } catch (err) {
              console.warn(err);
              this.$toast.warning(err.message);
              this.$toast.warning(this.$t("views.gather.undone.toast[1]"));
              return false;
            }
            console.log(data.data.cny);
            console.log($order);

            this.$router.push({
              name: "Payment-details",
              params: {
                item: {
                  sname: $order.sname, // 商家店名
                  id: $order.odid, // 总订单id
                  cny: $order.cny,
                  aipay: $order.ads, // 商家姓名和付款方式
                  oid: $order.id, // 子订单id
                  mail: $order.smes, // 商家姓名
                  service_charge: $order.amount2, //手续费
                  order_time: $order.eotc,
                  cash: data.data.cash, // 是否只接受现金
                  dsx: $order.dsx,
                },
                num: $order.num, // usdt 数量
                totalMoney: $order.amount1, // 总金额
                curpaymentterm: this.disposePayMethod(), //当前收款方式
                sellerMthods: myPayment(),
                MerchanInfo: {
                  bank: data.data.cny,
                  odid: $order.id,
                  payment_account: [
                    this.pay_Info[2]?.trim(),
                    this.pay_Info[1]?.trim(),
                  ],
                  wechat: data.data.amount1,
                },
              },
              query: {
                id: $order.id,
                inTrading: true,
              },
            });
            console.log([this.pay_Info[2]?.trim(), this.pay_Info[1]?.trim()]);
            break;
          }
          case "1": {
            let data;
            if ($order.updateDate === "1") {
              this.$router.push({
                name: "TemporaryCoinRefunds",
                params: {
                  order_item: $order,
                },
              });
              return false;
            }

            try {
              data = await Getsjmes($order.odid);
            } catch (err) {
              console.warn(err);
              this.$toast.warning(err.message);
            }
            let item = {
              sname: $order.sname, // 商家店名
              id: $order.odid, // 总订单id
              cny: $order.cny, // 单价
              oid: $order.id, // 子订单id
              mail: $order.smes, // 商家姓名
              mes: data.data.mes,
              smes: data.data.smes,
              service_charge: $order.amount2, //手续费
              order_time: $order.eotc, // 下单时间
              bank: data.data.bank, //商家银行卡
              aipay: data.data.aipay, // 商家支付宝
              wechat: data.data.wechat, // 商家微信
              ads: data.data.cny, // 商家钱包地址
              cash: data.data.cash, // 是否只接受现金
              dsx: $order.dsx,
            };
            this.$router.push({
              name: "Payment-waterbill",
              params: {
                item,
                num: $order.num, // usdt 数量
                totalMoney: $order.amount1, // 总金额
                sellerMthods: myPayment(), // 我的收付款方式
                MerchanInfo: {
                  bank: data.data.cny, // 商家钱包地址
                  odid: $order.id, //子订单id
                  amount1: $order.amount1, //总金额
                  aipay: data.data.amount2,
                  sname: item.sname,
                  wechat: data.data.amount1, // 电话号码
                },
              },
              query: {
                id: $order.id,
                role: "buyer",
                inTrading: true,
              },
            });
            break;
          }
          case "0": {
            let data;
            try {
              console.log(222)
              data = await Getsjmes($order.odid);
            } catch (err) {
              console.warn(err);
              this.$toast.warning(err.message);
              this.$toast.warning(this.$t("views.gather.undone.toast[1]"));
            }
            let item = {
              sname: $order.sname, // 商家店名
              id: $order.odid, // 总订单id
              cny: $order.cny, // 单价
              oid: $order.id, // 子订单id
              mail: $order.smes, // 商家姓名
              mes: data.data.mes,
              smes: data.data.smes,
              service_charge: $order.amount2, //手续费
              order_time: $order.eotc, // 下单时间
              bank: data.data.bank, //商家银行卡
              aipay: data.data.aipay, // 商家支付宝
              wechat: data.data.wechat, // 商家微信
              ads: data.data.cny, // 商家钱包地址
              cash: data.data.cash, // 是否只接受现金
            };
            this.$router.push({
              name: "outflows-currency",
              params: {
                item,
                num: $order.num, // usdt 数量
                totalMoney: $order.amount1, // 总金额
                sellerMthods: myPayment(), // 我的收付款方式
                MerchanInfo: {
                  bank: data.data.cny, // 商家钱包地址
                  odid: $order.id, //子订单id
                  amount1: $order.amount1, //总金额
                  aipay: data.data.amount2,
                  sname: item.sname,
                  wechat: data.data.amount1, // 电话号码
                },
              },
              query: {
                id: $order.id,
                inTrading: true,
              },
            });
            break;
          }
        }
      }
    },
    // 用户购买
    async buyer_orderType() {
      const $order = this.order_item;
      let data;
      try {
        data = await Getsjmes($order.odid);
      } catch (err) {
        console.warn(err);
        this.$toast.warning(this.$t("views.gather.undone.toast[1]"));
        return false;
      }
      switch ($order.dsx) {
        case "0": {
          data = data.data;
          const item = {
            sname: $order.sname,
            mes: data.mes,
            cny: $order.cny,
            id: $order.odid, //大订单id
            rcoin: $order.rcoin,
            dsx: $order.dsx,
          };
          const params = {
            item,
            nowTime: $order.eotc,
            num: $order.num,
            cuePayType: this.get_buyOrderPayMethod($order.ads),
            money: $order.amount1,
            sellerMthods: myPayment(),
            servicefee: $order.amount2,
            MerchanInfo: {
              bank: data.cny.trim(), // 商家钱包地址
              odid: $order.id,
              wechat: data.amount1, //商家电话号码
              aipay: data.amount2, // 商家邮箱
              sname: $order.mes,
              cur_payData: $order.ads, // "清茶树&现金交易&现金"
            },
          };
          console.log(params)
          this.$router.push({
            name: 'order-pay',
            params: params,
            query: {
              id: $order.id,
              inTrading: true,
            },
          });
          break;
        }
        case "1": {
          data = data.data;
          const item = {
            sname: $order.sname,
            dsx: $order.dsx,
          };
          this.$router.push({
            name: "water-bill",
            params: {
              money: $order.amount1,
              item,
              odid: $order.id,
              MerchanInfo: {
                wechat: data.amount1,
                aipay: data.amount2,
              },
            },
            query: {
              role: "buyer",
            },
          });

          break;
        }
      }
    },
    async clear_dsx2_order(timeOver) {
      if (timeOver <= 0) {
        await subbuysellorder({
          oid: this.order_item.id,
          type: 1,
          selectpayk: getItem("mysign"),
        });
        this.$toast.warning(this.$t("views.gather.undone.toast[0]"));
        this.$emit("update_OrderList");
        removeItem(this.order_item.id);
        localStorage.setItem("csnum", "0");
      }
    },
    get_buyOrderPayMethod(value) {
      let val = value?.split("&")[2] || "";
      if (val.includes("现金")) {
        return "xj";
      } else if (val.includes("支付宝")) {
        return "zfb";
      } else if (val.includes("微信")) {
        return "wx";
      } else {
        return "yhk";
      }
    },
    disposePayMethod() {
      const bank = this.order_item.bank;
      //console.log(bank);
      if (bank === "1") {
        return "xj";
      }
      const value = this.order_item.bank?.split("&");
      if (value[2] && value[2].includes("支付宝")) {
        return "zfb";
      } else if (value[2].includes("微信")) {
        return "wx";
      } else if (value[2].includes("现金")) {
        return "xj";
      } else {
        return "yhk";
      }
    },
  },
  computed: {
    pay_Info() {
      if (this.order_item.ads.includes("交易")) {
        return this.order_item.ads;
      }
      console.log(this.order_item.ads);
      if (!this.order_item.ads) {
        return " & & ".split("&");
      }
      return this.order_item.ads?.split("&") ?? " &  & ";
    },
  },
};
</script>
<style lang="less" scoped>
.order-container {
  p {
    margin: 1em !important;
  }
  font-size: 0.4rem;
  background-color: #eee;
  display: flex;
  margin: 25px;
  border-radius: 15px;
  padding: 0 25px 0 0;
  flex-direction: column;
  .header {
    display: flex;
    align-items: center;
    line-height: 40px;
    justify-content: space-between;
    span {
      line-height: 40px;
    }
    .header-top-left {
      display: flex;
      align-items: center;
      .header-top-left-test {
        color: rgb(12, 177, 121);
        font-weight: 700;
      }
    }
    .header-top {
      font-size: 0.31rem;
      display: flex;
      align-items: center;
      color: rgb(243, 151, 85);
      .pending_coin {
        font-size: 0.4rem;
        color: crimson;
        font-weight: 700;
      }
      div {
        color: rgb(243, 151, 85);
      }
      .header-topRight {
        color: rgb(235, 137, 64);
        font-size: 0.4rem;
        font-weight: 700;
      }
    }
  }
  .main {
    .eye-o {
      font-size: 0.5rem;
      margin-left: 3px;
    }
    li {
      font-size: 0.32rem;
      display: flex;
      justify-content: space-between;
      span {
        height: 40px;
        line-height: 40px;
      }
      span:first-child {
        color: var(--main-test-color);
      }
      .total_price {
        color: var(--main--color) !important;
        font-weight: 700;
        font-size: 0.5rem;
      }
      .price {
        color: rgb(245, 40, 40);
      }

      .showFullInfo-btn {
        border: 3px solid #999;
        padding: 10px;
        font-size: 0.3rem;
        font-weight: 700;
        border-radius: 45px;
        color: var(--main-test-color);
      }
    }
    .main-top {
      p {
        display: flex;
        margin: 0;
        flex-direction: column;
      }
      p:last-child {
        span:nth-child(n + 3) {
          border-left: 2px solid #999;
          padding-left: 25px;
        }
      }
      span {
        padding: 8px 0;
      }
    }
    .showFullInfo {
      margin: 25px 0 !important;
    }
  }
  .footer {
    display: flex;
    padding: 0 25px;
    align-items: center;
    justify-content: space-between;
    p {
      height: 40px;
      align-items: center;
      display: flex;
    }
    .info-rz {
      width: 40px;
      height: 40px;
    }
    p:last-child {
      color: var(--main-test-color);
    }
  }
  .payInfo {
    span:last-child {
      display: flex;
      align-items: center;
    }
  }
}

.iconfont {
  margin-right: 15px;
}

.saleInfo {
  background-color: #fff !important;
  box-sizing: border-box;
  h6 {
    margin: 0;
    font-size: 36px;
  }
  .header {
    font-size: 36px;
    color: #000;
    font-weight: 700;
    padding: 15px;
    border-bottom: 2px solid var(--main-test-color);
    .header-text {
      margin-left: 15px;
    }
  }
  .salePay-info {
    display: flex;
    margin: 45px 25px 25px;
    flex-direction: column;
    background-color: rgb(249, 249, 249);
    padding: 15px;
    border-radius: 15px;
    .span1 {
      margin: 10px;
      flex: 1;
      line-height: 40px;
    }
    .zy-info {
      color: #ee0a24;
    }
    font-size: 24px;
  }
  .sell-info {
    font-size: 26px;
    ul {
      display: flex;
      flex-direction: column;
      li {
        padding: 15px;
        display: flex;
        justify-content: space-between;
        span:last-child {
          font-weight: 700;
        }
      }
    }
  }
}
</style>
