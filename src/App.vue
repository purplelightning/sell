<template>
  <div>
    <v-header :seller="seller"></v-header>
    <div class="tab border-1px topborder-1px">
      <div class="tab-item">
        <a v-link="{path:'/goods'}">商品</a>
      </div>
      <div class="tab-item">
        <a v-link="{path:'/ratings'}">评论</a>
      </div>
      <div class="tab-item">
        <a v-link="{path:'/seller'}">商家</a>
      </div>
    </div>
  <!--keep-alive保留组件的状态 -->
    <router-view :seller="seller" keep-alive></router-view>

  </div>
</template>

<script>
  import {urlParse} from './common/js/util'

  import header from './components/header/header'

  const ERR_OK = 0;//ok的状态码

  export default {
    data() {
      return {
        seller: {
          id: (() => {//立即执行函数
            let queryParam = urlParse();
//            console.log(queryParam);
            return queryParam.id;
          })()
        }
      }
    },
    created() {
      this.$http.get('/api/seller?id='+this.seller.id).then((res) => {
        res = res.body;//调用.json,变成json格式
        if (res.errno === ERR_OK) {
//          this.seller = res.data;
          this.seller=Object.assign({},this.seller,res.data);//扩展id属性
//          console.log(this.seller.id);
        }

      })
    },
    components: {
      'v-header': header,//用来起别名

    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "./common/stylus/mixin.styl"

  .tab
    display: flex;
    width: 100%;
    height: 40px;
    line-height: 40px;
    /*border-bottom:1px solid rgba(7,17,27,0.1) 为了在所有dpi不同的设备上都显示1像素*/
    border-1px(rgba(7, 17, 27, 0.1))
    topborder-1px(rgba(7, 17, 27, 0.1))
    .tab-item
      flex: 1
      text-align: center
      & > a
        display: block
        font-size: 14px
        color: rgb(77, 85, 93)
        &.active
          color: rgb(240, 20, 20)
</style>
