<template>
  <div id="qrcodePage">

    <mu-flexbox class="userInfo img-box" :style="{backgroundImage:'url('+require('./../assets/imgs/header_bg.png')+')'}">
      <mu-flexbox-item align="center" justify="center">
        <com-user-info :theme="'white'"></com-user-info>
        <div class="absolute" style="position:absolute;bottom:-35px;right:3%">
          <mu-float-button icon="crop_free" class="qrcode-scan-btn"></mu-float-button>&nbsp;&nbsp;
          <mu-float-button icon="add" class="qrcode-add-btn" @click="openBottomSheet"></mu-float-button>
        </div>
      </mu-flexbox-item>
    </mu-flexbox>

    <div style="background:#f8f9fa">
      <mu-paper style="margin:4em 1em 1em 1em;padding-left:1em;">
        <mu-flexbox>
          <mu-flexbox-item grow='4'>
            <mu-text-field hintText="检查搜索" class="searchBar" :underlineShow='false' />
          </mu-flexbox-item>
          <mu-flexbox-item>
            <mu-icon-button icon="search" @click="handlerSearchChange"></mu-icon-button>
          </mu-flexbox-item>
        </mu-flexbox>
      </mu-paper>
      <mu-flexbox class="totalSection">
        <mu-flexbox-item align="center">
          <div class="title">{{units.length||0}}</div>
          <div class="summary">资产所属单元/间</div>
        </mu-flexbox-item>
        <mu-flexbox-item align="center">
          <div class="title">{{qrcodeItems.length||0}}</div>
          <div class="summary">资产总数/件</div>
        </mu-flexbox-item>
      </mu-flexbox>
    </div>

    <swiper id="unitsSwiper" :options="swiperOption" style="padding:1em 0;">
      <swiper-slide v-for="(e,i) in units" :key="i" class="img-box" :style="{backgroundImage:'url('+e.thumb+')'}" style="border-radius:5px">
        <div class="roomTitle">{{e.title}}</div>
      </swiper-slide>
    </swiper>

    <!-- 权属二维码记录区 -->
    <mu-flexbox v-for="(e,i) in qrcodeItems" v-if="e.visible" :key="i" class="paper paper-round code-list-item" style="padding:1em;border-radius:5px;">
      <mu-flexbox-item>
        <img :src="e.qrcodeImg||null" alt="" style="width:88px">
      </mu-flexbox-item>
      <mu-flexbox-item>
        <div class="title">
          {{e.title||'测试标题'+i}}
        </div>
        <div class="summary">
          测试内容
        </div>
      </mu-flexbox-item>
    </mu-flexbox>
    <!-- 新增物品二维码 -->
    <mu-bottom-sheet :open="bottomSheet" @close="closeBottomSheet">
      <mu-list @itemClick="closeBottomSheet">
        
      </mu-list>
    </mu-bottom-sheet>
  </div>
</template>

<script>
const comUserInfo = () => import("@/apps/public/userInfo.vue");
import Qrcode from "qrcode";
import QrcodeApi from "./../components/api.js";

import { swiper, swiperSlide } from "vue-awesome-swiper";
import "swiper/dist/css/swiper.css";
import $ from "jquery";

export default {
  name: "qrcodePage",
  data() {
    return {
      swiperOption: {
        width: 140,
        slidesPerView: "auto",
        nested: true,
        spaceBetween: 20
      },
      qrcodeItems: [],
      units: [],
      bottomSheet: false
    };
  },
  components: {
    swiper,
    swiperSlide,
    "com-user-info": comUserInfo
  },
  created() {
    let vm = this;
    //  获取二维码 - 20个
    vm.$store.commit("changeQrcodes", QrcodeApi.getQrcodes());
    //  获取单元数量
    vm.units = QrcodeApi.getUnits();
  },
  computed: {
    handlerQrcodeItemsChange: function() {
      //  返回store.Qrcodes
      return this.$store.getters.getQrcodes;
    }
  },
  watch: {
    //  监听store.Qrcodes
    handlerQrcodeItemsChange: function(v) {
      v.map((e, i) => {
        e.infoUrl = "/scan?union_id=" + e.unionid;
        Qrcode.toDataURL(
          "http://wxoa.emking.cn/scan?union_id=" + e.unionid,
          (err, url) => {
            e.qrcodeImg = url;
            e.visible = true;
          }
        );
      });
      this.qrcodeItems = v;
    }
  },
  methods: {
    closeBottomSheet() {
      this.bottomSheet = false;
    },
    openBottomSheet() {
      this.bottomSheet = true;
    },
    handlerSearchChange(v) {
      //此处需要向后台提交搜索的值,当用户输入完毕之后才提交
    }
  }
};
</script>
<style lang="scss" scoped>
#qrcodePage {
  .userInfo {
    box-shadow: 0 0 25px #6373f0;
    position: relative;
  }
  .qrcode-add-btn {
    background: linear-gradient(top, #feb567, #f44272);
  }
  .qrcode-scan-btn {
    background: linear-gradient(top, #35e2b3, #06a7c3);
  }
  & .searchBar {
    padding-bottom: 0;
    margin-bottom: 0;
    margin-top: 8px;
    & .mu-text-field-content {
      padding-bottom: 0 !important;
    }
  }
  & .swiper-slide {
    width: 180px;
    height: 100px;
  }
  & .roomTitle {
    font-size: 48px;
    font-family: "Arial Black";
    color: rgb(255, 255, 255);
    text-shadow: 0 0 10px #333;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  & .totalSection {
    padding: 1em 2em;
    & .title {
      font-size: 40px;
      font-family: "Arial Black";
      color: rgb(94, 121, 239);
    }
    & .summary {
      font-size: 18px;
      font-family: "Lantinghei SC";
      color: rgb(88, 88, 88);
      line-height: 1.25;
    }
  }

  & .code-list-item {
    padding: 0.5em;
    margin: 2%;
    width: 96%;
    & .title {
      color: #333;
      font-size: 24px;
      line-height: 2em;
      font-family: "Arial Black";
    }
    & summary {
      color: #999;
      font-size: 18px;
      line-height: 1.5em;
      font-family: "Lantinghei SC";
    }
  }
}
</style>

