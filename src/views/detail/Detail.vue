<template>
	<div class="detail">
		<detail-nav-bar @titleClick="titleClick" ref="nav"></detail-nav-bar>
		<scroll class="content" ref="scroll" @scroll="contentScroll" :probeType="3">
			<detail-swiper :top-images="topImages"></detail-swiper>
			<detail-base-info :goods="goods"></detail-base-info>
			<detail-shop-info :shop="shop"></detail-shop-info>
			<detail-goods-info :detail-info="detailInfo" @imgLoad="imgLoad"></detail-goods-info>
			<detail-param-info :param-info="paramInfo" ref="paramInfo"></detail-param-info>
			<detail-comment-info :comment-info="commentInfo" ref="commentInfo"></detail-comment-info>
			<goods-list :goods="recommends" ref="recommends"></goods-list>
		</scroll>
		<detail-bottom-nav @addToCart="addToCart"></detail-bottom-nav>
		<back-top @click.native="backClick" v-show="isShowBackTop"></back-top>
		<toast :message="message" :toastShow="toastShow"/>
	</div>
</template>

<script>
	import DetailNavBar from "./childComps/DetailNavBar";
	import DetailSwiper from "./childComps/DetailSwiper";
	import DetailBaseInfo from "./childComps/DetailBaseInfo";
	import DetailShopInfo from "./childComps/DetailShopInfo";
	import DetailGoodsInfo from "./childComps/DetailGoodsInfo";
	import DetailParamInfo from "./childComps/DetailParamInfo";
	import DetailCommentInfo from "./childComps/DetailCommentInfo";
	import DetailBottomNav from "./childComps/DetailBottomNav";

	import Scroll from "components/common/scroll/Scroll";
	import GoodsList from "components/content/goods/GoodsList";
	import BackTop from "components/content/backtop/BackTop";
	import Toast from "../../components/common/toast/Toast";

	import {getDetail,Goods,Shop,GoodsParam,getRecommend} from "network/detail";

	import {debounce} from "common/utils";
	import {itemListenerMixin} from "common/mixin";

	export default {
		name: "Detail",
		components:{
			DetailNavBar,
			DetailSwiper,
			DetailBaseInfo,
			DetailShopInfo,
			DetailGoodsInfo,
			DetailParamInfo,
			DetailCommentInfo,
			DetailBottomNav,
			Scroll,
			GoodsList,
			BackTop,
			Toast
		},
		mixins:[
				itemListenerMixin
		],
		data(){
			return{
				iid:null,
				topImages:[],
				goods:{},
				shop:{},
				detailInfo:{},
				paramInfo:{},
				commentInfo:{},
				recommends:[],
				navBarClickY:[],
				getNavBarY:null,
				currentIndex:0,
				isShowBackTop:false,
				message:'',
				toastShow:false
			}
		},
		created() {
			//1.保存传入的iid
			this.iid = this.$route.params.iid;
			// console.log(this.$route.params.iid);

			//2.根据iid请求详情数据
			getDetail(this.iid).then(res => {
				// console.log(res);
				const data = res.result;
				//1.获取顶部的轮播图片数据
				this.topImages = data.itemInfo.topImages;

				//2.获取并整合商品信息到Goods类中
				this.goods = new Goods(data.itemInfo,data.columns,data.shopInfo.services);

				//3.创建店铺信息的对象
				this.shop = new Shop(data.shopInfo);

				//4.保存商品的详情数据
				this.detailInfo = data.detailInfo;

				//5.获取参数信息
				this.paramInfo = new GoodsParam(data.itemParams.info,data.itemParams.rule);

				//6.取出评论信息
				if (data.rate.cRate){
					this.commentInfo = data.rate.list[0]
				}
				//对给navBarClickY赋值的操作进行防抖
				this.getNavBarY = debounce(()=>{
					this.navBarClickY = []
					this.navBarClickY.push(0)
					this.navBarClickY.push(this.$refs.paramInfo.$el.offsetTop)
					this.navBarClickY.push(this.$refs.commentInfo.$el.offsetTop)
					this.navBarClickY.push(this.$refs.recommends.$el.offsetTop)
					this.navBarClickY.push(Number.MAX_VALUE)
					// console.log(this.navBarClickY);
				},100)
			});

			//3.请求推荐数据
			getRecommend().then(res => {
				// console.log(res);
				this.recommends = res.data.list
			})
		},
		mounted() {

		},
		destroyed() {
			this.$bus.$off('itemImgLoad',this.itemImgListener)
		},
		methods:{
			imgLoad(){
				this.refresh();
				this.getNavBarY()
			},
			titleClick(index){
				// console.log(index);
				this.$refs.scroll.scrollTo(0,-this.navBarClickY[index],200)
			},
			contentScroll(position){
				// console.log(position.y);
				//判断滚动位置以及对应的index
				const positionY = -position.y;
				let length = this.navBarClickY.length
				// for (let i = 0;i <= length;i++){
				// 	if (this.currentIndex !== i && ((i < length -1 && positionY >= this.navBarClickY[i] && positionY < this.navBarClickY[i+1]) ||
				// 			(i === length-1 && positionY >= this.navBarClickY[i]))){
				// 		this.currentIndex = i
				// 		// console.log(this.currentIndex);
				// 		this.$refs.nav.currentIndex = this.currentIndex
				// 	}
				// }
				for (let i = 0;i <= length;i++){
					if (this.currentIndex !== i && (positionY < this.navBarClickY[i+1] && positionY > this.navBarClickY[i])){
						this.currentIndex = i
						// 		// console.log(this.currentIndex);
								this.$refs.nav.currentIndex = this.currentIndex
					}
				}
				//展示返回顶部
				this.isShowBackTop = -(position.y) > 1000
			},
			backClick(){
				this.$refs.scroll.scrollTo(0,0,200)
			},
			addToCart(){
				//1.获取购物车需要展示的信息
				const item = {};
				item.image = this.topImages[0];
				item.title = this.goods.title;
				item.desc = this.goods.desc;
				item.price = this.goods.realPrice;
				item.iid = this.iid
				// console.log(item);
				//2.将商品添加到购物车里
				this.$store.dispatch('addCart',item).then(res => {
					// console.log(res);
					// this.toastShow = true;
					// this.message = res;
					// setTimeout(() => {
					// 	this.toastShow = false
					// },2000)
					this.$toast.toastShow(res,3000)
				})
			}
		}
	}
</script>

<style scoped>
.detail{
	position: relative;
	z-index: 9;
	background-color: #fff;
	height: 100vh;
}
	.content{
		overflow: hidden;
		position: absolute;
		top: 44px;
		bottom: 49px;
		left: 0;
		right: 0;
	}
</style>