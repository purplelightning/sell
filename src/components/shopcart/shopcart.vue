<template>
  <div class="shopcart">
    <div class="content" @click="toggleList">
      <div class="content-left">
        <div class="logo-wrapper">
          <div class="logo" :class="{'highlight':totalCount>0}">
            <i class="icon-shopping_cart" :class="{'highlight':totalCount>0}"></i>
          </div>
          <div class="num" v-show="totalCount>0">{{totalCount}}</div>
          <div class="logo-have">
            <i class="icon-shopping_cart"></i>
          </div>
        </div>
        <div class="price" :class="{'highlight':totalPrice>0}">
          ¥ {{totalPrice}}
        </div>
        <div class="desc">
          另需配送费{{deliveryPrice}} 元
        </div>
      </div>
      <!--阻止冒泡,阻止默认事件-->
      <div class="content-right" @click.stop.prevent="pay">
        <div class="pay" :class="payClass">
          {{payDesc}}
        </div>
      </div>
    </div>

    <div class="ball-container">
      <div transition="drop" v-for="ball in balls" v-show="ball.show"
           class="ball">
        <div class="inner"></div>
      </div>
    </div>

    <div class="shopcart-list" v-show="listShow" transition="fold">
      <div class="list-header">
        <h1 class="title">购物车</h1>
        <span class="empty" @click="empty">清空</span>
      </div>
      <div class="list-content" v-el:list-content>
        <ul>
          <li class="food" v-for="food in selectFoods">
            <span class="name">{{food.name}}</span>
            <div class="price">
              <span>¥ {{food.price * food.count}}</span>
            </div>
            <div class="cartcontrol-wrapper">
              <cartcontrol :food="food"></cartcontrol>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
  <div class="list-mask" v-show="listShow" transition="fade"
  @click="hideList"></div>
</template>

<script type="text/ecmascript-6">
  import cartcontrol from '../cartcontrol/cartcontrol'
  import BScroll from 'better-scroll'

  export default {
    data() {
      return {
        balls: [
          {
            show: false
          },
          {
            show: false
          },
          {
            show: false
          },
          {
            show: false
          },
          {
            show: false
          },
        ],
        dropBall: [],
        fold: true
      }
    },
    //App.vue传seller到goods,goods把deliveryPrice,minPrice传到shopcart;这里也要接收
    props: {
      selectFoods: {
        type: Array,
        default() {
          return [
            {
              price: 15,
              count: 1
            }
          ];
        }
      },
      deliveryPrice: {
        type: Number,
        default: 0
      },
      minPrice: {
        type: Number,
        default: 0
      }
    },
    computed: {
      totalPrice() {
        let total = 0;
        this.selectFoods.forEach((food) => {
          total += food.price * food.count;
        });
        return total;
      },
      totalCount() {
        let count = 0;
        this.selectFoods.forEach((food) => {
          count += food.count;
        });
        return count;
      },
      payDesc() {
        if (this.totalPrice === 0) {
          return ` ¥${this.minPrice}起送`;//ES6反引号,在有变量的时候,不用加号拼字符串
        } else if (this.totalPrice < this.minPrice) {
          let diff = this.minPrice - this.totalPrice;
          return `还差¥${diff}起送`;
        } else {
          return '去结算';
        }
      },
      payClass() {
        if (this.totalPrice < this.minPrice) {
          return 'not-enough';
        } else {
          return 'enough';
        }
      },
      listShow() {
        if (!this.totalCount) {
          this.fold = true;
          return false;
        }
        let show = !this.fold;
        if (show) {
          this.$nextTick(() => {
            if (!this.scroll) {
              this.scroll = new BScroll(this.$els.listContent, {
                click: true
              });
            } else {
              this.scroll.refresh();//重新计算高度差,决定是否滚动
            }
          });
        }

        return show;
      }
    },
    methods: {
      //这里的el就是goods从cartcontrol获得的target
      drop(el) {
//        console.log(el);//这里不能实现,与dispatch不起作用有关,
        for (let i = 0; i < this.balls.length; i++) {
          let ball = this.balls[i];
          if (!ball.show) {
            ball.show = true;
            ball.el = el;
            this.dropBall.push(ball);
            return;
          }
        }
      },
      toggleList: function () {
        if (!this.totalCount) {
          return;
        }
        this.fold = !this.fold;
      },
      //清空购物车
      empty: function () {
        this.selectFoods.forEach((food) => {
          food.count = 0;
        })
      },
      hideList:function () {
        this.fold=true;//不能直接设置计算属性
      },
      pay:function () {
        if(this.totalPrice<this.minPrice){
          return;
        }else{
          window.alert(`支付${this.totalPrice}元~`);
        }
      }
    },
    components: {
      cartcontrol
    },
    transition: {
      drop: {
        beforeEnter(el) {
          let count = this.balls.length;
          while (count > 0) {
            let ball = this.balls[count];
            if (ball.show) {
              let rect = ball.el.getBoundingClientRect();
              let x = rect.left - 32;
              let y = -(window.innerHeight - rect.top - 22);
              el.style.display = '';
              el.style.webKitTransform = `translate3d(0,${y}px,0)`;
              el.style.transform = `translate3d(0,${y}px,0)`;
            }
            count--;
          }
        },
        enter(el) {

        },
        afterEnter(el) {

        },
      }
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import '../../common/stylus/mixin.styl'

  .shopcart
    position: absolute
    left: 0
    bottom: 0
    z-index: 50
    width: 100%
    height: 48px
    .content
      display: flex
      color: rgba(255, 255, 255, 0.4)
      background: #141d27
      font-size: 0
      .content-left
        flex: 1
        background: #141d27
        .logo-wrapper
          display: inline-block
          position: relative
          vertical-align: top
          top: -10px
          padding: 6px
          margin: 0 12px
          width: 56px
          height: 56px
          box-sizing: border-box
          border-radius: 50%
          background: #141d27
          .logo
            width: 100%
            height: 100%
            text-align: center
            border-radius: 50%
            background: #2b343c
            &.highlight
              background: rgb(0, 160, 220)
              color: #fff
            .icon-shopping_cart
              line-height: 44px
              font-size: 24px
              color: #80858a
              &.highlight
                color: #fff
          .num
            position: absolute
            top: 0
            right: 0
            width: 24px
            height: 16px
            line-height: 16px
            text-align: center
            border-radius: 16px
            font-size: 9px
            font-weight: 700
            color: #ffffff
            background: rgb(240, 20, 20)
            box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.4)
        .price
          display: inline-block
          vertical-align: top
          margin-top: 12px
          font-size: 16px
          font-weight: 700
          line-height: 24px
          padding-right: 12px
          box-sizing: border-box
          border-right: 1px solid rgba(255, 255, 255, 0.1)
          &.highlight
            color: #fff
        .desc
          display: inline-block
          vertical-align: top
          font-size: 10px
          margin: 12px 0 0 12px //这边..
          line-height: 24px

      .content-right
        flex: 0 0 105px
        width: 105px
        .pay
          padding: 0 8px
          height: 48px
          line-height: 48px
          text-align: center
          font-size: 12px
          font-weight: 700
          &.not-enough
            background: #2b333b
          &.enough
            background: #00b43c
            color: #fff
    .ball-container
      .ball
        position: fixed
        left: 32px
        bottom: 22px
        z-index: 200
        &.drop-transition
          transition: all 0.4s cubic-bezier(0.49, -0.29, 0.75, 0.41)
          opacity: 1
          .inner
            width: 16px
            height: 16px
            border-radius: 50%
            background: rgb(0, 160, 220)
            transition: all 0.4s linear
    .shopcart-list
      position: absolute
      z-index: -1
      left: 0
      top: 0
      width: 100%
      &.fold-transition
        transition: all 0.5s
        transform: translate3d(0, -100%, 0)
      &.fold-enter, &.fold-leave
        transform: translate3d(0, 0, 0)
      .list-header
        padding: 0 18px
        line-height: 40px
        height: 40px
        background: #f3f5f7
        border-bottom: 1px solid rgb(7, 17, 27, .1)
        .title
          float: left
          color: rgb(7, 17, 27)
          font-size: 14px
          line-height: 40px
        .empty
          float: right
          font-size: 12px
          color: rgb(0, 160, 220)
      .list-content
        padding: 0 18px
        max-height: 217px
        overflow: hidden
        background: #fff
        .food
          position: relative
          padding: 12px 0
          box-sizing: border-box
          border-1px(rgba(7, 17, 27, 0.1))
          .name
            font-size: 14px
            color: rgb(7, 17, 27)
            line-height: 24px
          .price
            position: absolute
            right: 90px
            bottom: 12px
            font-size: 14px
            line-height: 24px
            color: rgb(240, 20, 20)
            font-weight: 700
          .cartcontrol-wrapper
            position: absolute
            right: 0
            bottom: 6px

  .list-mask
    position: fixed
    top: 0
    left: 0
    height: 100%
    width: 100%
    z-index: 40
    backdrop-filter: blur(10px)
    &.fade-transition
      opacity: 1
      transition: all 0.5s
      background: rgba(7, 17, 27, 0.6)
    &.fade-enter, &.fade-leave
      opacity: 0
      background: rgba(7, 17, 27, 0)
</style>
