<template>
	<div class="nav-bar">
		<div class="left"><slot name="left"></slot></div>
		<div class="center"><slot name="center"></slot></div>
		<div class="right"><slot name="right"></slot></div>
	</div>
</template>

<script>
	export default {
		name: "NavBar"
	}
</script>

<style scoped>
	.nav-bar {
		display: flex;
		line-height: 44px;
		height: 44px;
		text-align: center;
		box-shadow: 0px 5px 10px rgba(100,100,100,.2);
	}
	.left,.right {
		width: 60px;
	}
	.center {
		flex: 1;
	}
</style>