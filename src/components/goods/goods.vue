<template>
    <div>
        <div class="goods">
            <div class="menu-wrapper" ref="menuWrapper">
                <ul>
                    <li v-for="(item,index) in goods" class="menu-item" :class="{'current': currentIndex === index}"
                        @click="selectMenu(index, $event)">
                        <span class="text border-1px">
                            <span v-show="item.type>0" class="icon" :class="classMap[item.type]"></span>
                            {{item.name}}
                        </span>
                    </li>
                </ul>
            </div>
            <div class="foods-wrapper" ref="foodsWrapper">
                <ul>
                    <li v-for="(item,index) in goods" class="food-list food-list-hook">
                        <h1 class="title">{{item.name}}</h1>
                        <ul>
                            <li @click="selectFood(food, $event)" v-for="food in item.foods"
                                class="food-item border-1px">
                                <div class="icon">
                                    <img :src="food.icon" width="57" height="57">
                                </div>
                                <div class="content">
                                    <h2 class="name">{{food.name}}</h2>
                                    <p class="desc">{{food.description}}</p>
                                    <div class="extra">
                                        <span class="count">月售{{food.sellCount}}</span><span>好评率{{food.rating}}%</span>
                                    </div>
                                    <div class="price">
                                        <span class="now">{{food.price}}元</span><span class="old"
                                                                                      v-show="food.oldPrice">{{food.oldPrice}}元</span>
                                    </div>
                                    <div class="cartcontrol-wrapper">
                                        <cartcontrol :food="food"></cartcontrol>
                                    </div>
                                </div>
                            </li>
                        </ul>
                    </li>
                </ul>
            </div>
            <shopcart ref="shopCart" :select-foods="selectFoods" :delivery-price="seller.deliveryPrice"
                      :min-price="seller.minPrice"></shopcart>
        </div>
        <food :food="selectedFood" ref="food" class="myfood"></food>
    </div>
</template>

<script type="text/ecmascript-6">
    import {vueData} from 'common/js/vueData.js';
    import BScroll from 'better-scroll';
    import shopcart from 'components/shopcart/shopcart.vue';
    import cartcontrol from 'components/cartcontrol/cartcontrol.vue';
    import food from 'components/food/food.vue';
    // const ERR_OK = 0;
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
        created() {
            this.classMap = ['decrease', 'discount', 'special', 'invoice', 'guarantee'];
            this.goods = vueData()['goods'];
            this.$nextTick(() => {
                this._initScroll();
                this._calculateHeight();
            });
            // this.$http.get('/api/goods').then((response) => {
            //     // console.log(response.body);
            //     response = response.body;
            //     // console.log(response[0]['name']);
            //     if (response.errno === ERR_OK) {
            //         this.goods = response.data;
            //         console.log(this.goods);
            //         console.log(this.goods[0].name);
            //         // 保证页面已渲染节点后再执行
            //         this.$nextTick(() => {
            //             this._initScroll();
            //             this._calculateHeight();
            //         });
            //     }
            // });
            this.$on('cart-add', this._drop);
        },
        computed: {
            currentIndex() {
                for (let i = 0; i < this.listHeight.length; i++) {
                    let height1 = this.listHeight[i];
                    let height2 = this.listHeight[i + 1];
                    if (!height2 || (this.scrollY >= height1 && this.scrollY < height2)) {
                        return i;
                    }
                }
                return 0;
            },
            selectFoods() {
                let foodsArr = [];
                this.goods.forEach((good) => {
                    good.foods.forEach((food) => {
                        if (food.count) {
                            foodsArr.push(food);
                        }
                    });
                });
                return foodsArr;
            }
        },
        methods: {
            _initScroll() {
                this.menuScroll = new BScroll(this.$refs.menuWrapper, {
                    click: true
                });
                this.foodsScroll = new BScroll(this.$refs.foodsWrapper, {
                    // 监测实时滚动的位置
                    probeType: 3,
                    click: true
                });
                this.foodsScroll.on('scroll', (pos) => {
                    this.scrollY = Math.abs(Math.round(pos.y));
                    let scrollTop = this.scrollY;
                    if (scrollTop % 100 === 0) {
                        console.log(scrollTop);
                        console.log(this.currentIndex);
                    }
                });
            },
            _calculateHeight() {
                let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
                let height = 0;
                this.listHeight.push(height);
                for (let i = 0; i < foodList.length; i++) {
                    height += foodList[i].clientHeight;
                    this.listHeight.push(height);
                }
            },
            selectMenu(index, event) {
                // 不是vue.js原生的事件，就阻止掉
                if (!event._constructed) {
                    return;
                }
                let foodList = this.$refs.foodsWrapper.getElementsByClassName('food-list-hook');
                let el = foodList[index];
                this.foodsScroll.scrollToElement(el, 300);
            },
            _drop(target) {
                console.log('体验优化，异步执行下落动画');
                this.$nextTick(() => {
                    this.$refs.shopCart.drop(target);
                });
            },
            selectFood(food, event) {
                if (!event._constructed) {
                    return;
                }
                this.selectedFood = food;
                this.$refs.food.show();
            }
        },
        components: {
            shopcart,
            cartcontrol,
            food
        }
    };

</script>

<style lang="stylus" rel="stylesheet/stylus">
    @import "../../common/stylus/mixin.styl"
    .goods
        display: flex
        position: absolute
        width: 100%
        top: 174px
        bottom: 46px
        overflow: hidden
        .menu-wrapper
            flex: 0 0 80px
            width: 80px
            background: #f3f5f7
            .menu-item
                display: table
                height: 54px
                width: 56px
                line-height: 14px
                padding: 0 12px
                &.current
                    position: relative
                    margin-top: -1px
                    z-index: 10
                    background: #ffffff
                    font-weight: 700
                    .text
                        border-none()
                .icon
                    display: inline-block
                    width: 12px
                    height: 12px
                    vertical-align: top
                    margin-right: 2px
                    background-size: 12px 12px
                    background-repeat: no-repeat
                    &.decrease
                        bg-image('decrease_3')
                    &.discount
                        bg-image('discount_3')
                    &.guarantee
                        bg-image('guarantee_3')
                    &.invoice
                        bg-image('invoice_3')
                    &.special
                        bg-image('special_3')
                .text
                    font-size: 12px
                    display: table-cell
                    vertical-align: middle
                    width: 56px
                    border-1px(rgba(7, 17, 27, 0.5))
        .foods-wrapper
            flex: 1
            .title
                padding-left: 14px
                height: 26px
                line-height: 26px
                border-left: 2px solid #d9dde1
                font-size: 12px
                color: rgb(147, 153, 159)
                background: #f3f5f7

            .food-item
                display: flex
                margin: 18px
                padding-bottom: 18px
                border-1px(rgba(7, 17, 27, 0.1))
                &:last-child
                    border-none()
                    margin-bottom: 0
                .icon
                    flex: 0 0 57px
                    margin-right: 10px
                .content
                    flex: 1
                    .name
                        margin: 2px 0 5px 0
                        height: 14px
                        line-height: 14px
                        font-size: 14px
                        color: rgb(7, 17, 27)
                    .desc, .extra
                        line-height: 10px
                        font-size: 10px
                        color: rgb(147, 153, 159)
                    .desc
                        margin-bottom: 5px
                        line-height: 15px
                    .extra
                        .count
                            margin-right: 12px
                    .price
                        font-weight: 700
                        line-height: 24px
                        .now
                            margin-right: 8px
                            font-size: 14px
                            color: rgb(240, 20, 20)
                        .old
                            text-decoration: line-through
                            font-size: 10px
                            color: rgb(147, 153, 159)
                    .cartcontrol-wrapper
                        position: absolute
                        right: 0
                        bottom: 12px
</style>
