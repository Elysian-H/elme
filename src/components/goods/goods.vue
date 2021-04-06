<template>
  <div>
    <div class="goods">
      <!--==========-->
      <div class="menu-wrapper" ref="menuWrapper">
        <ul>
          <li v-for="(item, index) in goods" class="menu-item"
            @click="selectMenu(index)"
            ref="menuList"
            :class="{'current': currentIndex === index}">
            <span class="text border-1px">
              <span v-show="item.type>=0" class="icon" :class="classMap[item.type]"></span>
              {{ item.name }}
            </span>
          </li>
        </ul>
      </div>
      <!--===========-->
      <div class="foods-wrapper" ref="foodsWrapper">
        <ul>
          <li v-for="item in goods" class="food-list" ref="foodList">
            <h1 class="title">{{ item.name}}</h1>
            <ul>
              <li v-for="food in item.foods" class="food-item border-1px">
                <div class="icon">
                  <img :src="food.icon" width="57" height="57" />
                </div>
                <div class="content">
                  <h2 class="name">{{ food.name }}</h2>
                  <p class="desc">{{ food.description }}</p>
                  <div class="extra">
                    <span class="count">月售{{ food.sellCount }}份</span>
                    <span>好评率{{ food.rating}}%</span>
                  </div>
                  <div class="price">
                    <span class="now">￥{{ food.price }}</span>
                    <span class="old" v-show="food.oldPrice">￥{{ food.oldPrice}}</span>
                  </div>
                  <div class="cartcontrol-wrapper">
                    <!--这里为购物车控制组件-->
                    <cartcontrol :food="food" @add="addFood"></cartcontrol>
                  </div>
                </div>
              </li>
            </ul>
          </li>
        </ul>
      </div>
      <!--=====-->
      <shopcart :deliveryPrice="seller.deliveryPrice" 
      	:minPrice="seller.minPrice"  ref="shopcart"
        :selectFoods="selectFoods"></shopcart>
    </div>
  </div>
</template>

<script>
import BScroll from 'better-scroll'
import shopcart from 'components/shopcart/shopcart'
import cartcontrol from 'components/cartcontrol/cartcontrol'
	
const ERR_OK = 0
export default {
  //接受参数数据
  props:{
    seller: {
      type:Object
    }
  },
  
  data () {
    return {
      goods: [],
      listHeight: [],
      scrollY: 0
    }
  },
  created(){
    this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee']

    const url = "/api/goods";
    this.$http.get(url).then((response) => {
      response = response.body
      if(response.errno === ERR_OK){
        this.goods = response.data
//      console.log(this.goods)
        this.$nextTick(() => {
          this._initScroll();
          //计算高度
          this._calculateHeight();
        })
      }
    })
  },
  methods: {
    _initScroll(){
      this.menuScroll = new BScroll(this.$refs.menuWrapper,{
        click: true
      });
      this.foodsScroll = new BScroll(this.$refs.foodsWrapper,{
        click: true,
        //希望scroll在滚动时实时告诉我们滚动位置
        probeType: 3
      });
      //监听滚动的实时位置，位置信息放在pos中
      this.foodsScroll.on('scroll', (pos)=>{
//    	console.log(pos)
        if(pos.y <= 0){
        	this.scrollY = Math.abs(Math.round(pos.y))
        }
      })
    },
    //当点击左侧的时候，右侧变化到实时坐标
    selectMenu(index){
      let foodList = this.$refs.foodList
      let el = foodList[index]
      //300秒动画效果
      this.foodsScroll.scrollToElement(el, 300)
    },
    _calculateHeight(){
      let foodList = this.$refs.foodList
      let height = 0
      this.listHeight.push(height)
      for (let i=0; i<foodList.length; i++) {
        let item=foodList[i]
        //console.log(item)
        //clientHeight为每个分类下面的食物列表的高度
        console.log(item.clientHeight)
        height += item.clientHeight
        this.listHeight.push(height)
      }
    },
    _followScroll(index){
      let menuList = this.$refs.menuList;
      let el = menuList[index]
      this.menuScroll.scrollToElement(el, 300, 0, -100)
    },
    //这个addFood方法是cartcontrol组件触发的方法
    addFood(target){
      this._drop(target);
    },
    //小球下落的方法
    _drop(target){
    	//体验优化，异步执行下落动画
    	this.$nextTick(()=>{
    		//this.$refs.shopcart是在goods组件中拿到的shopcart组件
    		//从而可以调用shopcart组件的drop方法
    		this.$refs.shopcart.drop(target);
    	})
    }
  },
  computed: {
    //把scrollY和左侧的索引建立一个映射
    currentIndex(){
      for(let i=0; i<this.listHeight.length; i++){
        let height1 = this.listHeight[i];
        let height2 = this.listHeight[i+1];
        if(!height2 || (this.scrollY >= height1 && this.scrollY < height2)){
          this._followScroll(i)
          return i;
        }
      }
      // 否则默认返回0
      return 0;
    },
    selectFoods(){
      let foods = []
      this.goods.forEach((good)=>{
        good.foods.forEach((food)=>{
          if(food.count){
            foods.push(food)
          }
        })
      })
      return foods
    }
  },
  //注册子组件
  components: {
    shopcart,
    cartcontrol
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus" rel="stylesheet/stylus">
@import "../../common/stylus/mixin"
.goods
  display: flex
  position: absolute
  top: 174px
  bottom: 48px
  /*background: greenyellow*/
  width: 100%
  overflow: hidden
  .menu-wrapper
    flex: 0 0 80px
    width: 80px
    background: #f3f5f7
    .menu-item
      display: table
      height: 54px
      width: 56px
      padding: 0 12px
      line-height: 14px
      &.current
        position: relative
        z-index: 10
        background: #66ccff /*yellowgreen*/
        color: white
        margin-top: -1px
        .text
          border-none()
      .text
        display: table-cell
        width: 56px
        vertical-align: middle
        font-size: 12px
        border-1px(rgba(7,17,27,0.8))
        .icon
          display: inline-block
          vertical-align: top
          width: 12px
          height: 12px
          margin-right: 2px
          background-size: 12px 12px
          background-repeat: no-repeat
          &.decrease
            bg-image('decrease_3')
          &.discount
            bg-image('discount_3')
          &.special
            bg-image('special_3')
          &.invoice
            bg-image('invoice_3')
          &.guarantee
            bg-image('guarantee_3')
  .foods-wrapper
    flex: 1
    .title
      padding-left: 14px
      height: 26px
      line-height: 26px
      border-left: 2px solid #d9dde1
      font-size: 12px
      color:rgb(147, 153, 159)
      background: #f3f5f7
    .food-item
      display: flex
      margin: 18px
      padding-bottom: 18px
      border-1px(rgba(7,17,27,0.1))
      .icon
        flex: 0 0 57px
        width: 57px
        margin-right: 10px
      .content
        flex: 1
        .name
          margin: 2px 0 8px 0
          height: 14px
          line-height: 14px
          font-size: 14px
          color: rgb(7,17,27)
        .desc,.extra
          line-height: 10px
          font-size: 10px
          color:rgb(147,153,159)
        .desc
          line-height: 12px
          margin-bottom: 8px
        .extra
          .count
            margin-right: 12px
        .price
          font-weight: 700
          line-height: 24px
          .now
            margin-right: 8px
            font-size: 14px
            color:rgb(240,20,20)
          .old
            text-decoration: line-through
            font-size: 10px
            color: rgb(147,153,159)
        .cartcontrol-wrapper
          position: absolute
          right: 0
          bottom: 12px
</style>
