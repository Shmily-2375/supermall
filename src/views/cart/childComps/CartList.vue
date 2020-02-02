<template>
  <div class="cart-list">
    <scroll class="content" ref="scroll">
      <CartListItem v-for="(item, index) in cartList"
                    :key="index"
                    :itemInfo="item"/>
    </scroll>
  </div>
</template>

<script>
  import {mapGetters} from 'vuex'

  import Scroll from "../../../components/common/scroll/Scroll";
  import CartListItem from "./CartListItem";

  export default {
    name: "CartList",
    components:{
      Scroll,
      CartListItem
    },
    computed:{
      ...mapGetters(['cartList'])
    },
    activated() {
      this.$refs.scroll.refresh()
    }
  }
</script>

<style scoped>
  .cart-list{
    height: calc(100% - 44px - 49px - 40px);
  }
  .content{
    height: 100%;
    overflow: hidden;
  }
</style>
