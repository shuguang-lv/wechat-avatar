<template>
	<view class="content">
		<head-bar></head-bar>
		<image src="../../static/images/welcome-image.png" mode="aspectFit" class="welcome-image"></image>
		<div class="welcome-msg">Class of 2022 Congratulations!</div>
		<div class="introduction">Wearing special avatar frame and celebrate graduation together</div>
		<button class="start-button" hover-class="start-button-hover" @click="navMain"><span>Start</span></button>
	</view>
</template>

<script>
	import HeadBar from '@/components/HeadBar.vue'
	
	export default {
		components: {
			HeadBar
		},
		data() {
			return {
				
			};
		},
		onShareAppMessage() {
			return {
				title: getApp().globalData.shareTitle,
				imageUrl: "/static/images/share-card.png",
				path: "/pages/welcome/index"
			}
		},
		onShareTimeline() {
			return {
				title: getApp().globalData.shareTitle,
				imageUrl: "/static/images/share-card.png",
				path: "/pages/welcome/index"
			}
		},
		methods: {
			navMain() {
				uni.navigateTo({
					url: '/pages/main/index'
				});
			}
		}
	}
</script>

<style lang="scss">
	.content {
		height: 100vh;
		width: 100vw;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		
		.welcome-image {
			width: 50vw;
		}
		
		.welcome-msg {
			font-style: normal;
			font-weight: 700;
			font-size: 28px;
			line-height: 35px;
			color: $theme-yellow;
			text-align: center;
			margin-bottom: 10vh;
		}
		
		.introduction {
			width: 179px;
			height: 50px;
			font-style: normal;
			font-weight: 500;
			font-size: 15px;
			line-height: 20px;
			color: #FFFFFF;
			text-align: center;
			margin-bottom: 30px;
		}
		
		.start-button {
			width: 230px;
			height: 40px;
			background-color: $theme-yellow;
			font-style: normal;
			font-weight: 600;
			font-size: 20px;
			color: $uni-text-color-black;
			transition: 0.3s;
		}
		
		.start-button-hover {
			transform: scale(0.95);
			transition: 0.3s;
		}
	}
</style>
