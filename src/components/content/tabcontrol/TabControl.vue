<template>
	<div class="tab-control">
		<div v-for="(item,index) in titles"
				 class="tab-control-item"
				 :class="{active:currentIndex == index}"
				 @click="tabClick(index)">
			<span>{{item}}</span>
		</div>
	</div>
</template>

<script>
	export default {
		name: "TabControl",
		props:{
			titles:{
				type:Array,
				default(){
					return []
				}
			}
		},
		data(){
			return{
				currentIndex:0
			}
		},
		methods:{
			tabClick(index){
				this.currentIndex = index
				this.$emit('tabClick',index)
				// console.log(index);
			}
		}
	}
</script>

<style scoped>
	.tab-control{
		display: flex;
		text-align: center;
		font-size: 15px;
	}
	.tab-control-item{
		flex: 1;
		height: 40px;
		line-height: 40px;
	}
	.tab-control-item span{
		padding: 5px;
	}
	.active{
		color: red;
	}
	.active span{
		border-bottom: 3px solid red;
	}
</style>