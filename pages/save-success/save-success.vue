<template>
	<view class="content">
		<!-- <view class="iconfont icon-xiangzuo" @click="navBack"></view> -->
		<view class="image-div">
			<view class="icon-xiazai iconfont"></view>
			<view class="tip-div">Congrats! Your avatar was saved to the photo album successfully.</view>
			<image :src="currentImage" class="image-card" mode="widthFix"></image>
		</view>
		<view class="btn-card">
			<button class="action-btn" @click="navIndex()">New avatar</button>
			<button open-type="share" class="primary-btn">Share</button>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				currentImage: '',
				indexBg: uni.getStorageSync('background_info') ? uni.getStorageSync('background_info').find(el => el
					.code === 'saveSucess_bg') : {},
				avatarImage: uni.getStorageSync('avatar_image'),
				shareInfo: uni.getStorageSync('shareInfo')
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
		onShareAppMessage: function() {
			return this.shareInfo;
		},
		onShareTimeline: function() {
			return this.shareInfo;
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

		// .icon-xiangzuo {
		// 	position: fixed;
		// 	top: 6vh;
		// 	left: 30rpx;
		// 	color: #ffa462;
		// 	font-size: 40rpx;
		// 	font-weight: bold;
		// }
		.image-div {
			display: flex;
			align-items: center;
			justify-content: center;
			flex-direction: column;

			.icon-xiazai {
				color: $theme-blue-two;
				font-size: 120rpx;
				margin-bottom: 40rpx;
			}

			.tip-div {
				background-color: $theme-blue-two;
				color: $uni-text-color-inverse;
				padding: 8rpx 30rpx;
				border-radius: $uni-border-radius-lg;
				font-size: $uni-font-size-lg;
				width: 70vw;
			}

			.image-card {
				margin: 80rpx;
				width: 340rpx;
				border-radius: $uni-border-radius-lg;
				border: 3rpx solid $uni-border-color;
			}
		}

		.btn-card {
			padding: 10rpx 30rpx 30rpx;
			box-sizing: border-box;
			display: flex;
			justify-content: space-between;
			width: 100vw;

			.action-btn {
				width: 320rpx;
				background: $uni-bg-color;
				border: 5rpx solid $uni-border-color;
				border-radius: $uni-border-radius-lg;
				box-shadow: 0 12rpx 16rpx -8rpx rgba(0, 0, 0, 0.1);
				color: $uni-text-color-grey;
				padding: 0;
				font-size: $uni-font-size-lg;
				font-weight: bold;
			}

			.primary-btn {
				width: 320rpx;
				background: linear-gradient(97.71deg, $theme-blue-two, $theme-blue-four 88.36%);
				border-radius: $uni-border-radius-lg;
				box-shadow: 0 12rpx 16rpx -8rpx $theme-blue-three;
				color: $uni-text-color-inverse;
				padding: 0;
				font-size: $uni-font-size-lg;
				font-weight: bold;
			}
		}
	}
</style>
