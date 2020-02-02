<template>
  <div id="detail">
    <detail-nav-bar class="detail-nav" @titleClick="titleClick" ref="nav"/>
    <scroll class="content" ref="scroll"
            @scroll="contentScroll"
            :probe-type="2">
      <detail-swiper :topImages="topImages"/>
      <DetailBaseInfo :goods="goods"/>
      <DetailShopInfo :shop="shop"/>
      <DetailGoodsInfo :detail-info="detailInfo" @imageLoad="imageLoad"/>
      <DetailParamInfo ref="params" :paramInfo="paramInfo"/>
      <DetailCommentInfo ref="comment" :commentInfo="commentInfo"/>
      <GoodsList ref="recommend" :goods="recommends"/>
      </scroll>
      <DetailBottomBar @addCart="addToCart"/>
      <Toast :message="message" :show="show"/>
  </div>
</template>

<script>
  import DetailNavBar from './childComps/DetailNavBar.vue'
  import DetailSwiper from "./childComps/DetailSwiper";
  import DetailBaseInfo from "./childComps/DetailBaseInfo";
  import DetailShopInfo from "./childComps/DetailShopInfo";
  import DetailGoodsInfo from "./childComps/DetailGoodsInfo";
  import DetailParamInfo from "./childComps/DetailParamInfo";
  import DetailCommentInfo from "./childComps/DetailCommentInfo";
  import DetailBottomBar from "./childComps/DetailBottomBar";
  import Toast from "../../components/common/toast/Toast";

  import Scroll from "../../components/common/scroll/Scroll";
  import GoodsList from "../../components/content/goods/GoodsList";

  import {getDetail, Goods, Shop, GoodsParam, getRecommend} from "../../network/detail";
  import {debounce} from "../../common/utils";

  import {mapActions} from 'vuex'

  export default {
    name: "Detail",
    components: {
      DetailNavBar,
      DetailSwiper,
      DetailBaseInfo,
      DetailShopInfo,
      Scroll,
      DetailGoodsInfo,
      GoodsParam,
      DetailParamInfo,
      DetailCommentInfo,
      GoodsList,
      DetailBottomBar,
      Toast
    },
    data(){
      return{
        iid: null,
        topImages:[],
        goods: {},
        shop: {},
        detailInfo: {},
        paramInfo:{},
        commentInfo: {},
        recommends: [],
        themeTopYs: [],
        currentIndex: 0,
        message:'',
        show: false
      }
    },
    created() {
      // 1.保存传入的iid
      this.iid = this.$route.params.iid
      // 2.根据iid去请求数据
      getDetail(this.iid).then(res => {
      // 1.获取数据
       const data = res.result
      // 2.获取顶部的图片轮播
       this.topImages = data.itemInfo.topImages
      // 3.获取商品信息
       this.goods = new Goods(data.itemInfo, data.columns, data.shopInfo.services)
      // 4.创建店铺信息的对象
        this.shop = new Shop(data.shopInfo)
      // 5.保存商品的详情数据
        this.detailInfo = data.detailInfo
      // 6.获取参数信息
        this.paramInfo = new GoodsParam(data.itemParams.info, data.itemParams.rule)
      // 7.取出评论信息
        if(data.rate.cRate !==0){
          this.commentInfo = data.rate.list[0]
        }
        // this.$nextTick(() =>{
        //   //根据最新的数据, 对应的额DOM是已经被加载完的
        //   //但是图片依然是没有加载完(目前的offsetTop是不包含图片的)
        //   //offsetTop值不对,都是因为图片的问题
        //   this.themeTopYs = []
        //   this.themeTopYs.push(0)
        //   this.themeTopYs.push(this.$refs.params.$el.offsetTop)
        //   this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
        //   this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)
        //   console.log(this.themeTopYs);
        // })
      })
      // 3.请求推荐数据
      getRecommend().then(res => {
        this.recommends = res.data.list
      })
    },
    mounted() {
      //1.图片加载完成的事件监听
      const refresh = debounce(this.$refs.scroll.refresh, 400)
      this.$bus.$on('detailItemImageLoad', () => {
        refresh()
      })
    },
    methods:{

      ...mapActions(['addCart']),

      imageLoad(){
        this.$refs.scroll.refresh()
        this.themeTopYs = []
        this.themeTopYs.push(0)
        this.themeTopYs.push(this.$refs.params.$el.offsetTop)
        this.themeTopYs.push(this.$refs.comment.$el.offsetTop)
        this.themeTopYs.push(this.$refs.recommend.$el.offsetTop)
        this.themeTopYs.push(Number.MAX_VALUE)
      },
      addToCart(){
        //1.获取购物车需要展示的信息
        const product = {}
        product.image = this.topImages[0]
        product.title = this.goods.title
        product.desc = this.goods.desc
        product.price = this.goods.realPrice
        product.iid = this.iid
        //2.将商品添加到购物车里
        // this.$store.dispatch('addCart', product).then(res =>{
        //   console.log(res);
        // })
        this.addCart(product).then(res =>{
          this.show = true
          this.message = res
          setTimeout(() => {
            this.show = false
            this.message = ''
          }, 1500)
        })
      },
      titleClick(index){
        this.$refs.scroll.scrollTo(0, -this.themeTopYs[index], 200)
      },
      contentScroll(position){

        const positionY = -position.y
        let length = this.themeTopYs.length

        for(let i = 0; i < length - 1; i++){
          // if(this.currentIndex !== i && ((i < length - 1 && positionY >= this.themeTopYs[i] && positionY
          //   < this.themeTopYs[i + 1]) || (i === length - 1 && positionY >= this.themeTopYs[i]))){
          //   this.currentIndex = i
          //   this.$refs.nav.currentIndex = this.currentIndex
          // }
            if(this.currentIndex !== i && (positionY >= this.themeTopYs[i] && positionY <= this.themeTopYs[i + 1])){
              this.currentIndex = i
              this.$refs.nav.currentIndex = this.currentIndex
            }
        }
      }
    }
  }
</script>

<style scoped>
  #detail{
    position: relative;
    z-index: 9;
    background-color: #fff;
    height: 100vh;
  }

  .detail-nav{
    position: relative;
    z-index: 9;
    background-color: #fff;
  }

  .content{
    height: calc(100% - 44px - 58px);
    /*position: absolute;*/
    /*top: 44px;*/
    /*bottom: 58px;*/
    /*left: 0;*/
    /*right: 0;*/
  }
</style>
