<template>
  <div id="detail">
    <detail-nav-bar class="detail-nav" @titleClick="titleClick" ref="nav"/>
    <scroll class="content"
            ref="scroll"
            :probe-type="3"
            @scroll="contentScroll">
      <detail-swiper :top-images="topImages"/>
      <detail-base-info :goods="goods"/>
      <detail-shop-info :shop="shop"/>
      <detail-goods-info :detail-info="detailInfo" @imageLoad="imageLoad"/>
      <detail-param-info ref="params" :param-info="paramInfo"/>
      <detail-commen-info ref="comment" :comment-info="commentInfo"/>
      <goods-list ref="recommend" :goods="recommends"/>
    </scroll>
    <detail-bottom-bar @addCart="addtoCart"/>

    <back-top @click.native="backClick" v-show="isShowBackTop"/>
<!--    <toast :message="message" :show="show"/>-->
  </div>
</template>

<script>
  import DetailNavBar from "./ChildComps/DetailNavBar";
  import DetailSwiper from "./ChildComps/DetailSwiper";
  import DetailBaseInfo from "./ChildComps/DetailBaseInfo";
  import DetailShopInfo from "./ChildComps/DetailShopInfo";
  import DetailGoodsInfo from "./ChildComps/DetailGoodsInfo";
  import DetailParamInfo from "./ChildComps/DetailParamInfo";
  import DetailCommenInfo from "./ChildComps/DetailCommenInfo";
  import DetailBottomBar from "./ChildComps/DetailBottomBar";


  import GoodsList from "components/content/goods/GoodsList";
  import Scroll from "../../components/common/scroll/Scroll";
  // import Toast from "components/common/toast/Toast";

  import {getDetail, Goods, Shop, GoodsParam, getReccommend} from "network/datail";
  import {itemListenerMixin, backTopMixin} from "common/minxin";

  import { mapActions } from 'vuex'

  export default {
    name: "Detail",
    components: {
      DetailNavBar,
      DetailSwiper,
      DetailBaseInfo,
      DetailShopInfo,
      DetailGoodsInfo,
      DetailParamInfo,
      DetailCommenInfo,
      DetailBottomBar,

      Scroll,
      GoodsList,

      // Toast
    },
    mixins: [itemListenerMixin, backTopMixin],
    data() {
      return {
        iid: null,
        topImages: null,
        goods: {},
        shop: {},
        detailInfo: {},
        paramInfo: {},
        commentInfo: {},
        recommends: [],
        themeTopYs: [],
        currentIndex: 0,

        // message: '',
        // show: false
      }
    },
    created() {
      // 1.保存传入的iid
      this.iid = this.$route.params.iid

      // 2.根据iid请求详情数据
      getDetail(this.iid).then(res => {
        const data = res.result
        // console.log(data)
        // 1.获取顶部的图片轮播数据
        this.topImages = res.result.itemInfo.topImages

        // 2.获取商品信息
        this.goods = new Goods(data.itemInfo,data.columns, data.shopInfo.services)

        // 3.创建店铺信息对象
        this.shop = new Shop(data.shopInfo)

        // 4.保存商品详情数据
        this.detailInfo = data.detailInfo

        // 5.获取参数的信息
        this.paramInfo = new GoodsParam(data.itemParams.info, data.itemParams.rule)

        // 6.取出评论信息
        if (data.rate.cRate !== 0) {
          this.commentInfo = data.rate.list[0]
        }

        // 1. 第一次获取 值不对
        // 值不对的原因: this.$refs.params.$el压根没有渲染
        // this.themeTopYs = []
        //
        // this.themeTopYs.push(0);
        // this.themeTopYs.push(this.$refs.params.$el.offsetTop)
        // this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
        // this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)
        //
        // console.log(this.themeTopYs);


        // this.$nextTick(() => {
        //   // 2.第二次获取：值依然不对
        //   // 值不对的原因：图片没有计算在内
        //   // 根据最新的数据， 对于的DOM是已经被渲染出来
        //   // 但是图片依然是没有加载完的
        //   // 也就是说获取的offsetTop高度是不包括图片的
        //   // offsetTop值不对的时候 都是因为图片
        //   this.themeTopYs = []
        //
        //   this.themeTopYs.push(0);
        //   this.themeTopYs.push(this.$refs.params.$el.offsetTop)
        //   this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
        //   this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)
        //
        //   console.log(this.themeTopYs);
        // })
      })

      // 3.请求推荐数据
      getReccommend().then(res => {
        // console.log(res);
        this.recommends = res.data.list
      })
    },
    mounted() {

    },
    methods: {
      ...mapActions(['addCart']),
      imageLoad() {
        this.$refs.scroll.refresh()

        this.themeTopYs = []
        this.themeTopYs.push(0);
        this.themeTopYs.push(this.$refs.params.$el.offsetTop -44)
        this.themeTopYs.push(this.$refs.comment.$el.offsetTop -44)
        this.themeTopYs.push(this.$refs.recommend.$el.offsetTop -44)
        this.themeTopYs.push(Number.MAX_VALUE)

        // console.log(this.themeTopYs);

      },
      titleClick(index) {
        this.$refs.scroll.scrollTo(0, -this.themeTopYs[index], 200)
      },
      contentScroll(position) {
        // 1.获取y值
        const positionY = -position.y

        // 2.positionY和主题值对比
        // 0, 12729, 13708, 13906,
        // console.log(Number.MAX_VALUE);


        // positionY 再0-12329之间 index=0 以此类推
        let length = this.themeTopYs.length
        for(let i = 0; i < length-1;i ++) {
          // console.log(i); // str类型

          // if(positionY > this.themeTopYs[i]
          //   && positionY < this.themeTopYs[i+1]) {
          //   console.log(i)
          // }

          if (this.currentIndex !== i && (positionY >= this.themeTopYs[i] && positionY < this.themeTopYs[i + 1])) {
              this.currentIndex = i
              this.$refs.nav.currentIndex = this.currentIndex
          }

          // if (this.currentIndex !== i && ((i < length - 1 && positionY >= this.themeTopYs[i]
          //   && positionY < this.themeTopYs[i+1]) || (i === length -1 && positionY >= this.themeTopYs[i]))) {
          //   this.currentIndex = i
          //   this.$refs.nav.currentIndex = this.currentIndex
          // }

          // 3.判断是否需要显示回到顶部按钮
          this.listenShowBackTop(position)
        }
      },
      addtoCart() {
        // 1.获取购物车需要展示的信息
        const product = {}
        product.image = this.topImages[0]
        product.title = this.goods.title
        product.desc = this.goods.desc
        product.price = this.goods.realPrice
        product.iid = this.iid

        // 2.将商品添加到购物车(1.Promise,mapActions)
        // this.$store.commit('addCart', product)
        this.addCart(product).then(res => {
          // this.show = true
          // this.message = res
          //
          // setTimeout(() => {
          //   this.show = false
          //   this.message = ''
          // },1500)
          //
          // console.log(res);

          this.$toast.show(res, 1500)

          console.log(this.$toast);
        })

        // this.$store.dispatch('addCart', product).then(res => {
        //   console.log(res)
        // })

      }
    },
    destroyed() {
      this.$bus.$off('itemImgLoad', this.itemImgListener)
    }
  }
</script>

<style scoped>
  #detail {
    position: relative;
    z-index: 9;
    background-color: #fff;
    height: 100vh;
  }

  .content {
    height: calc(100% - 44px - 49px);
  }

  .detail-nav {
    position: relative;
    z-index: 9;
    background-color: #fff;
  }
</style>
