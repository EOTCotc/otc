<template>
  <div class="merchant-info-container">
    <van-nav-bar
      :title="$t('views.merchant.merchant.title')"
      left-arrow
      @click-left="$router.back()"
      @click-right="to_blacklist"
    >
      <template #right>
        <img class="r-img" src="@/assets/tools-svg/hmd.svg" alt="" />
      </template>
    </van-nav-bar>

    <header class="header">
      <van-cell center>
        <template #icon>
          <span class="aut-img">{{ sell_Info.sname.slice(0, 1) }}</span>
        </template>
        <template>
          <div class="merchant-info1">{{ sell_Info.sname }}</div>
          <div class="merchant-info2">
            {{ $t("views.merchant.merchant.recently") }} &nbsp;&nbsp;
            {{ sell_Info.updateDate | transformTime_MDMS }}
          </div>
        </template>
        <template #right-icon>
          <van-button
            v-if="!is_Attention"
            class="btn-Attention"
            type="info"
            round
            size="small"
            @click="Attention"
          >
            {{ $t("views.merchant.merchant.attention") }}
          </van-button>
          <van-button
            v-else
            class="btn-Attention"
            round
            size="small"
            @click="cancel_Attention"
          >
            {{ $t("views.merchant.merchant.yi_attention") }}
          </van-button>
        </template>
      </van-cell>

      <div class="order-info">
        <span>{{ $t("views.merchant.merchant.client") }}</span>
        <p>
          <span>
            <van-button class="btn-order" size="mini">{{
                $t("views.merchant.merchant.buy")
              }}</van-button>
          </span>
          <span>
            <van-button class="btn-order" size="mini">{{
                $t("views.merchant.merchant.sell")
              }}</van-button>
          </span>
        </p>
      </div>
    </header>

    <main class="main">
      <!-- <van-tab v-for="(item, index) in typeList" :title="item" :key="index"> -->
      <van-loading
        size="24px"
        :style="{ marginTop: '25px' }"
        vertical
        v-if="listLoading"
      >{{ $t("views.merchant.merchant.listLoading") }}</van-loading
      >
      <payment_empty
        v-else-if="isShow_empty"
        :title="$t('views.merchant.merchant.not_data')"
      />
      <van-list
        v-model="loading"
        :finished="finished"
        :finished-text="$t('views.merchant.merchant.finished')"
        @load="onLoad"
        v-else
      >
        <!-- 交易列表 start -->
        <van-cell-group inset v-for="(items, i) in list" :key="i">
          <van-cell>
            <template #title>
              <div class="left">
                <div class="aut-img">
                  {{ items.sname.slice(0, 1) }}
                  <div class="online-icon">
                    <p class="online-status"></p>
                  </div>
                </div>
                <span class="text">{{ items.sname }}</span>
              </div>
            </template>
            <template>
              <div class="right">
                <span>{{ items.odid }} | {{ items.chenjiao }}%</span>
              </div>
            </template>
          </van-cell>
          <van-cell>
            <template #title>
              <div class="left">
                {{ $t("views.merchant.merchant.number") }}
                {{ items.num | ThousandSeparator }} {{ item }}
              </div>
            </template>
            <template #label>
              <div>
                {{ $t("views.merchant.merchant.quota") }}
                {{ items.amount1 | ThousandSeparator }} -
                {{ items.amount2 | ThousandSeparator }} CNY
              </div>
              <Pay-Icons :items="items"></Pay-Icons>
            </template>
            <template>
              <div class="right">
                <span> {{ $t("views.merchant.merchant.price") }}</span>
              </div>
            </template>
            <template>
              <div class="right">
                <span class="txt">￥{{ items.cny }}</span>
              </div>
            </template>
            <template>
              <div>
                <van-button
                  class="comfig-button"
                  size="mini"
                  @click="SubmitTrading(i)"
                >
                  {{ method }}
                  <!--   去购买 -->
                </van-button>
              </div>
            </template>
          </van-cell>
        </van-cell-group>
        <!-- 交易列表 end -->
      </van-list>
      <!-- </van-tab> -->
    </main>
  </div>
</template>

<script>
import payment_empty from "@/views/order-gather/payment-empty.vue";
import { toastComponent } from "@/utils/element-config";

import { UserBind } from "@/api/trxRequest";

export default {
  name: "merchant-info",
  components: {
    payment_empty,
  },
  props: ["sell_Info"],
  data() {
    return {
      listLoading: false,
      isShow_empty: true,
      loading: false, // 数据加载
      finished: false,
      is_Attention: false,
    };
  },
  created() {
    console.dir(this.sell_Info);
    this.onLoad();
  },
  methods: {
    async onLoad() {
      const { data } = await UserBind({
        type: 10, //我的关注列表
      });

      this.is_Attention = data.some((user) => user.uid === this.sell_Info.eid);
    },
    async Attention() {
      this.is_Attention = true;
      const { data } = await UserBind({
        gid: this.sell_Info.eid,
        type: 1, //关注
      });
      console.log(data);
    },
    async cancel_Attention() {
      this.Attention();
      this.is_Attention = false;
    },
    to_blacklist() {
      this.$dialog
        .confirm({
          title: this.$t("views.merchant.merchant.hint"),
          message:
            "<span class='activeInfo'>" +
            this.$t("views.merchant.merchant.sure") +
            "</span>",
        })
        .then(async () => {
          await UserBind({
            gid: this.sell_Info.eid,
            type: 2, //关注
          });
          this.$toast.info({
            component: toastComponent,
            props: {
              title: this.$t("views.merchant.merchant.toast_title"),
              content: this.$t("views.merchant.merchant.toast_content"),
              color: "#fff",
            },
          });
          this.$router.replace({
            name: "CurrencyTrading",
          });
        })
        .catch(() => {
          // on cancel  取消选择
        });
    },
  },
};
</script>

<style lang="less" scoped>
.merchant-info-container {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  top: 0;
  z-index: 99;
  background-color: #fff;
  /deep/ .van-divider {
    margin: 0;
  }
  .r-img {
    width: 0.75rem;
    height: 0.75rem;
  }
  .aut-img {
    width: 0.75rem;
    height: 0.75rem;
    line-height: 0.75rem;
    background-color: var(--main--color);
    color: #fff;
    border-radius: 50%;
    text-align: center;
    margin-right: 15px;
  }
  .merchant-info1 {
    font-weight: 700;
  }
  .merchant-info2 {
    color: var(--main-test-color);
  }
  .btn-Attention {
    width: 1.75rem;
  }
  .order-info {
    padding: 30px;
    font-size: 0.4rem;
    font-weight: 700;
    display: flex;
    justify-content: space-between;
    box-shadow: 0 1.5px 1.5px 1px #eee;
    align-items: center;
    .btn-order {
      background-color: #eee;
    }
    button {
      margin-right: 30px;
      width: 1.3rem;
      color: #999;
    }
  }
}
</style>
