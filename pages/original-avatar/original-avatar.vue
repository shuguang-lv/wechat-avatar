<template>
	<view class="content">
		<!-- <view class="iconfont icon-xiangzuo" @click="navBack"></view> -->
		<view class="image-div">
			<view class="image-unit" v-for="(item, index) in imageList" :key="item._id">
				<image :src="item.image_url" class="image-card"></image>
				<view class="btn-card"><button class="primary-btn" @click="downLoadImage(item.image_url)">Save</button>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				imageList: [],
				indexBg: uni.getStorageSync('background_info') ? uni.getStorageSync('background_info').find(el => el
					.code === 'original_bg') : {},
				shareInfo: uni.getStorageSync('shareInfo')
			};
		},
		onLoad() {},
		onShareAppMessage: function() {
			return this.shareInfo;
		},
		onShareTimeline: function() {
			return this.shareInfo;
		},
		onLoad() {
			this.getUserImagesByOpenId();
		},
		methods: {
			navBack() {
				uni.navigateBack({
					delta: 1
				})
			},
			/**
			 * 获取头像
			 */
			getUserImagesByOpenId() {
				uni.showLoading({
					title: 'Loading...',
					mask: true
				});
				let {
					_id: useId
				} = uni.getStorageSync('user_info');
				uniCloud
					.callFunction({
						name: 'user_images',
						data: {
							type: 'getUserImagesByOpenId',
							useId
						}
					})
					.then(res => {
						this.imageList = res.result.data;
					})
					.catch(err => {
						uni.showModal({
							content: err.message || 'Fail to request service',
							showCancel: false
						});
					})
					.finally(() => {
						uni.hideLoading();
					});
			},
			/**
			 * 保存图片
			 */
			downLoadImage(imageUrl) {
				//获取相册授权
				let _self = this;
				uni.getSetting({
					success(res) {
						if (!res.authSetting['scope.writePhotosAlbum']) {
							uni.authorize({
								scope: 'scope.writePhotosAlbum',
								success() {
									//这里是用户同意授权后的回调
									_self.saveImgToLocal(imageUrl);
								},
								fail(e) {
									uni.hideLoading();
									wx.showModal({
										content: 'It is detected that image downloading function is not permitted. Do you want to enable it now ?',
										confirmText: 'Yes',
										cancelText: 'No',
										success: function(res) {
											//点击“确认”时打开设置页面
											if (res.confirm) {
												wx.openSetting();
											}
										}
									});
								}
							});
						} else {
							//用户已经授权过了
							_self.saveImgToLocal(imageUrl);
						}
					}
				});
			},
			/**
			 * @param {Object} imageUrl
			 * 保存到相册
			 */
			saveImgToLocal(imageUrl) {
				let _self = this;
				uni.downloadFile({
					url: imageUrl, //图片地址
					success: res => {
						if (res.statusCode === 200) {
							uni.saveImageToPhotosAlbum({
								filePath: res.tempFilePath,
								success: function() {
									uni.hideLoading();
									uni.showToast({
										title: 'Saved successfully',
										icon: 'none'
									});
								},
								fail: function() {
									uni.hideLoading();
									uni.showToast({
										title: 'Fail to save',
										icon: 'none'
									});
								}
							});
						}
					},
					fail: (res) => {
						console.log(res)
					}
				});
			}
		}
	};
</script>

<style lang="scss" scoped>
	.content {
		display: flex;
		justify-content: center;
		align-items: center;
		// padding: 12vh 30rpx 0;
		height: 100vh;
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		background-size: 100% 100%;

		// background-image: url(https://vkceyugu.cdn.bspapp.com/VKCEYUGU-08ecbb66-149e-4d2b-93a0-fa6bc6e0e894/5c68ee76-16d2-4c1a-ab82-032a3c8abd5d.png);
		.icon-xiangzuo {
			position: fixed;
			top: 6vh;
			left: 30rpx;
			color: $uni-bg-color;
			font-size: 40rpx;
			font-weight: bold;
		}

		.image-div {
			// background-image: url(https://vkceyugu.cdn.bspapp.com/VKCEYUGU-08ecbb66-149e-4d2b-93a0-fa6bc6e0e894/d8596aff-d3ec-4ce8-ae9e-78774efbd1a8.png);
			padding: 10rpx 20rpx 10rpx 20rpx;
			border-radius: $uni-border-radius-base;
			width: 700rpx;
			height: 90vh;
			display: flex;
			justify-content: space-between;
			align-content: flex-start;
			flex-wrap: wrap;
			box-sizing: border-box;
			overflow-y: auto;

			.image-unit {
				width: 300rpx;
				height: 400rpx;
				background-color: $uni-bg-color;
				box-shadow: 0px 5px 10px 0px $uni-shadow-color;
				border-radius: $uni-border-radius-lg;
				overflow: hidden;
				margin-bottom: 50rpx;

				.image-card {
					width: 300rpx;
					height: 300rpx;
				}

				.btn-card {
					padding: 10rpx 30rpx 30rpx;
					box-sizing: border-box;
					display: flex;

					.primary-btn {
						width: 150rpx;
						background: $theme-blue-two;
						// border: 1rpx solid $theme-blue-two;
						border-radius: $uni-border-radius-lg;
						// box-shadow: 0 12rpx 16rpx -8rpx $theme-blue-three;
						color: $uni-text-color-inverse;
						padding: 0;
						font-size: $uni-font-size-base;
						float: right;
					}
				}
			}
		}
	}
</style>
