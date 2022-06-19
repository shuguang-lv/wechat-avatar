<template>
	<view class="content">
		<head-bar></head-bar>
		<image src="https://7463-tcb-htqvbkbv9bxahty751ffe-b59e59-1305947755.tcb.qcloud.la/background-image.png" mode="aspectFit" class="background-image"></image>
		<view class="image-div">
			<image :src="currentImage" class="image-card" mode="aspectFit"></image>
		</view>
		<image src="../../static/icon/download-icon-gold.png" class="download-icon" mode="aspectFit"></image>
		<div class="success-msg">Successfully<br>Downloaded!</div>
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
				currentImage: '',
				avatarImage: uni.getStorageSync('avatar_image'),
			};
		},
		onLoad() {
			this.currentImage = uni.getStorageSync('currentImage');
			uni.removeStorageSync('currentImage');
		},
		onShow() {
			if (this.avatarImage) {
				uni.setStorageSync('avatar_image', this.avatarImage)
			}
		},
		onShareAppMessage() {
			return {
				title: "UNNC 2022 Graduation",
				imageUrl: "/static/images/share-card.png",
				path: "/pages/save-success/index"
			}
		},
		onShareTimeline() {
			return {
				title: "UNNC 2022 Graduation",
				imageUrl: "/static/images/share-card.png",
				path: "/pages/save-success/index"
			}
		},
		methods: {
			/**
			 * 返回上一页
			 */
			navBack() {
				uni.navigateBack({
					delta: 1
				})
			},
			/**
			 * 跳到首页
			 */
			navIndex() {
				uni.reLaunch({
					url: '/pages/index/index'
				});
			}
		}
	};
</script>

<style lang="scss" scoped>
	.content {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		height: 100vh;
		
		.background-image {
			position: fixed;
			z-index: -1;
			top: 50vh;
			left: 50vw;
			width: 312px;
			height: 356px;
			transform: translate(-50%, -50%);
		}

		.image-div {
			display: flex;
			align-items: center;
			justify-content: center;
			flex-direction: column;
			width: $avatar-size + 100rpx;
			height: $avatar-size + 100rpx;
			background-color: rgba(255, 255, 255, 0.9);
			margin-bottom: 10vh;

			.image-card {
				height: $avatar-size + 50rpx;
				width: $avatar-size + 50rpx;
			}
		}
		
		.download-icon {
			width: 35px;
			height: 35px;
			margin-bottom: 16px;
		}

		.success-msg {
			font-size: 20px;
			line-height: 30px;			
			text-align: center;
			color: $theme-yellow;
		}
	}
</style>
