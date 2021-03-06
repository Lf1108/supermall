<template>
  <div id="home">
    <!-- 头部 -->
    <nav-bar class="home-nav">
      <div slot="center">购物街</div>
    </nav-bar>
    <!-- 预备吸顶 -->
    <tab-control
      :titles="showList"
      @tabClick="tabClick"
      ref="tabControl1"
      class="tab-control"
      v-show="isFixed"
    />
    <!-- 滚动包含元素 -->
    <Scroll
      ref="scroll"
      :probe-type="3"
      @scroll="contentScroll"
      :pull-up-load="true"
      @pullingUp="loadMore"
    >
      <!-- 轮播 -->
      <home-swiper :banners="banners.list" @swiperImgLoad="swiperImgLoad"></home-swiper>
      <!-- 推荐 -->
      <recommend-view :recommends="recommends"></recommend-view>
      <!-- 流行 -->
      <feature-view />
      <!-- 首页导航 -->
      <tab-control
        :class="{fixed: isFixed}"
        :titles="showList"
        @tabClick="tabClick"
        ref="tabControl2"
      />
      <!-- 商品列表 -->
      <goods-list :goods="showGoods" />
    </Scroll>
    <!-- 回到顶部 -->
    <back-top @click.native="backClick" v-show="isShow"></back-top>
  </div>
</template>

<script scoped>
// 组件导入
import NavBar from "components/common/navbar/NavBar";
import TabControl from "components/content/tabControl/TabControl";
import GoodsList from "components/content/goods/GoodsList";
import HomeSwiper from "./child-comps/HomeSwiper";
import RecommendView from "./child-comps/RecommendView";
import FeatureView from "./child-comps/FeatureView";
import Scroll from "components/common/scroll/Scroll";
import BackTop from "components/common/back-top/BackTop";

// 网络请求
import { getHomeMulitData, getHomeGoods } from "network/home";

export default {
  name: "Home",
  components: {
    NavBar,
    TabControl,
    GoodsList,
    HomeSwiper,
    RecommendView,
    FeatureView,
    Scroll,
    BackTop
  },
  data() {
    return {
      banners: [],
      recommends: [],
      goods: {
        pop: { page: 0, list: [] },
        new: { page: 0, list: [] },
        sell: { page: 0, list: [] }
      },
      currentType: "pop",
      isShow: false,
      tabOffsetTop: 0,
      isFixed: false,
      saveY: 0
    };
  },
  computed: {
    showGoods() {
      return this.goods[this.currentType].list;
    },
    showList() {
      return ["流行", "新款", "精选"];
    }
  },
  created() {
    // 1.请求多个数据
    this.getHomeMulitData();
    // 2.请求商品数据
    this.getHomeGoods("pop");
    this.getHomeGoods("new");
    this.getHomeGoods("sell");
  },
  mounted() {},
  activated() {
    //获取状态并展示
    this.$refs.scroll.scroll.scrollTo(0, -this.saveY, 0);
  },
  deactivated() {
    // 切换页面记录状态
    this.saveY = this.$refs.scroll.scroll.getScrollY;
    this.$refs.scroll.scroll.refresh();
  },
  methods: {
    // 1.请求多个数据
    getHomeMulitData() {
      getHomeMulitData().then(res => {
        this.banners = res.data.banner;
        this.recommends = res.data.recommend.list;
      });
    },
    // 2.请求商品数据
    getHomeGoods(type) {
      let page = this.goods[type].page + 1;

      getHomeGoods(type, page).then(res => {
        this.goods[type].list.push(...res.data.list);
        this.goods[type].page += 1;

        this.finishUp();
      });
    },
    // 切换分类
    tabClick(index) {
      switch (index) {
        case 0:
          this.currentType = "pop";
          break;
        case 1:
          this.currentType = "new";
          break;
        case 2:
          this.currentType = "sell";
          break;
      }
      this.$refs.tabControl1.currentType = index;
      this.$refs.tabControl2.currentType = index;
    },
    // 返回顶部
    backClick() {
      this.$refs.scroll.scroll.scrollTo(0, 0, 500);
    },
    // 监听位置
    contentScroll(position) {
      // 1.是否显示
      this.isShow = -position.y > 1000;
      // 2.是否吸顶
      this.isFixed = -position.y > this.tabOffsetTop;
    },
    // 上拉加载更多
    loadMore() {
      // 指定分类目录
      this.getHomeGoods(this.currentType);
    },
    // 数据是否加载完成
    finishUp() {
      this.$refs.scroll.scroll.finishPullUp();
    },
    // 判断图片是否加载完成并计算高度
    // 为是否吸顶
    swiperImgLoad() {
      this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop;
    }
  }
};
</script>

<style scoped>
#home {
  padding-top: 44px;
}

.home-nav {
  color: #fff;
  background-color: deeppink;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 9;
}

.fixed {
  position: fixed;
  top: 44px;
  left: 0;
  right: 0;
}

.tab-control {
  position: relative;
  z-index: 9;
}
</style>