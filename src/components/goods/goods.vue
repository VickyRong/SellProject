<template>
  <div class="goods">
    <!--左侧导航-->
    <div class="menu-wrapper" v-el:menu-wrapper>
      <ul>
        <li v-for="item in goods" class="menu-item" :class="{'current':currentIndex === $index}" @click="_selectMenu($index,$event)">
          <span class="text">
            <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>
            {{item.name}}
          </span>
        </li>
      </ul>
    </div>
    <!--右侧菜品-->
    <div class="foods-wrapper" v-el:foods-wrapper>
      <ul>
        <li v-for="item in goods" class="food-list food-list-hook">
          <h1 class="title">{{item.name}}</h1>
          <ul>
            <li @click="selectFood(food,$event)" v-for="food in item.foods"  class="food-item border-1px">
              <div class="icon">
                <img :src="food.icon" width="57" height="57">
              </div>
              <div class="content">
                <h2 class="name">{{food.name}}</h2>
                <p class="desc">{{food.description}}</p>
                <div class="extra">
                  <span class="count">月售{{food.sellCount}}份</span>
                  <span>好评率{{food.rating}}%</span>
                </div>
                <div class="price">
                  <span class="now">￥{{food.price}}</span><span v-show="food.oldPrice" class="old">￥{{food.oldPrice}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <!--计数加减组件-->
                  <cartcontrol :food="food"></cartcontrol>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <!--购物车组件-->
    <shopcart v-ref:shopcart :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcart>
  </div>
  <!--food组件-->
  <food :food = "selectedFood" v-ref:food></food>
</template>
<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/styles/mixin.styl";
  @import "goods.styl";
</style>
<script type="text/ecmascript-6">
  import shopcart from '../shopcart/shopcart.vue';
  import cartcontrol from '../cartcontrol/cartcontrol.vue';
  import food from '../food/food.vue';
  import BScroll from 'better-scroll';

  const ERR_OK = 0;
  export default {
    props: {
      seller: {
        type: Object
      }
    },
    data() {
      return {
        goods: [],
        listHeight: [],
        scrollY: 0,
        selectedFood: {}
      };
    },
    computed: {
      currentIndex() {
        for (let i = 0; i < this.listHeight.length; i++) {
          let height1 = this.listHeight[i];
          let height2 = this.listHeight[i + 1];
          if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
            // console.info(i);
            return i;
          }
        }
        return 0;
      },
      selectFoods() {
        let foods = [];
        this.goods.forEach((good) => {
          good.foods.forEach((food) => {
            if (food.count) {
              foods.push(food);
            }
          });
        });
        return foods;
      }
    },
    created() {
      this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];

      this.$http.get('/api/goods').then((response) => {
        response = response.body;
        if (response.errno === ERR_OK) {
          this.goods = response.data;
          this.$nextTick(() => {
            this._initScroll();
            this._calculateHeight();
          });
        }
      });
    },
    methods: {
      _selectMenu: function (index, event) { // 点击左侧导航联动
        if (!event._constructed) {
          return;   //  防止pc端派发两次事件
        }
        let foodList = this.$els.foodsWrapper.getElementsByClassName('food-list-hook');
        let el = foodList[index];
        this.foodsScroll.scrollToElement(el, 300);
      },
      _initScroll: function () {  // 初始化滚动条插件
        this.menuScroll = new BScroll(this.$els.menuWrapper, {
          click: true //  scroll插件会禁止掉原有的点击触摸等属性，所以要重新恢复一下
        });
        this.foodsScroll = new BScroll(this.$els.foodsWrapper, {
          click: true, //  scroll插件会禁止掉原有的点击触摸等属性，所以要重新恢复一下
          probeType: 3
        });
        this.foodsScroll.on('scroll', (pos) => {
          this.scrollY = Math.abs(Math.round(pos.y));
          // console.log(this.scrollY);
        });
      },
      _calculateHeight: function () { // 计算滚动时页面高度
        let foodList = this.$els.foodsWrapper.getElementsByClassName('food-list-hook');
        let height = 0;
        this.listHeight.push(height);
        for (let i = 0; i < foodList.length; i++) {
          let item = foodList[i];
          height += item.clientHeight;
          this.listHeight.push(height); // 得到每个foodList的高度值，并存在数组里面
        }
        // console.log(this.listHeight);
      },
      _drop(target) { // 将接收到target事件传给子组件shopcart
        // 体验优化，异步执行动画
        this.$nextTick(() => {
          this.$refs.shopcart.drop(target);
        });
      },
      selectFood(food, event) {
        if (!event._constructed) {
          return;   //  防止pc端派发两次事件
        }
        this.selectedFood = food;
        this.$refs.food.show();
        console.log(food);
        console.log(event);
      }
    },
    components: {
      shopcart,
      cartcontrol,
      food
    },
    events: {
      'cart.add'(target) { // 接收cartcontrol组件传过来的target事件
        // console.info(target);
        this._drop(target);
      }
    }
  };
</script>
