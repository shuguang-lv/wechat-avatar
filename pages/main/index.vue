<template>
	<view class="content" @touchmove.stop @click.native="touchAvatarBg(false)">
		<head-bar></head-bar>
		<image src="https://7463-tcb-htqvbkbv9bxahty751ffe-b59e59-1305947755.tcb.qcloud.la/background-image.png" mode="aspectFit" class="background-image"></image>

		<view class="hideCanvas">
			<canvas class="default_PosterCanvasId"
				:style="{ width: imageScale * cansBorder + 'px', height: imageScale * cansBorder + 'px' }"
				canvas-id="default_PosterCanvasId"></canvas>
		</view>

		<view class="image-card">
			<view class="avatar-div" id="avatar-container" @click.stop>
				<image class="img" id="avatar-img" :src="avatarImage || defaultImage" @click="touchAvatarBg(false)">
				</image>
				<image v-if="currentImage && currentImage.image_url" class="avatar-default"
					:class="{ 'avatar-border': showBorder && currentImage.drag_state }" :style="{
						top: maskCenterY - maskSize / 2 - 3 + 'px',
						left: maskCenterX - maskSize / 2 - 3 + 'px',
						transform: 'rotate(' + rotate + 'deg)' + 'scale(' + scale + ')'
					}" id="mask-img" :src="currentImage.image_url" @click="touchAvatarBg(false)"></image>
				<!-- 				<view class="drag-div"
					:style="{ top: handleCenterY - 10 + 'px', left: handleCenterX - 10 + 'px', width: handleWidth + 'rpx', height: handleWidth + 'rpx' }"
					v-if="showBorder && currentImage && currentImage.drag_state">
					<image class="drag-img" id="drag-img" src="/static/images/drag.svg"></image>
				</view> -->
			</view>
		</view>
		
		<div class="load-image-buttons">
			<div class="button-container">
				<button class="load-image-button" hover-class="load-image-button-hover" open-type="getUserInfo" @click.stop="getUserProfile('createImages')"><span>Fetch</span></button>
				<div class="button-descripsion">匹配当前头像<br>Fetch current avatar</div>
			</div>
			<div class="button-container">
				<button v-if="userInfo" class="load-image-button" hover-class="load-image-button-hover" @click.stop="chooseImages('selectedImage')"><span>Select</span></button>
				<button v-else class="load-image-button" hover-class="load-image-button-hover" open-type="getUserInfo" @click.stop="getUserProfile('selectedImage')"><span>Select</span></button>
				<div class="button-descripsion">从相册选择<br>Select from albums</div>
			</div>
		</div>
		
		<l-clipper v-if="showClipper" :image-url="avatarImage" @success="avatarImage = $event.url; showClipper = false"
			@cancel="showClipper = false" />
			
		<view class="image-selector" @click.stop>
			<scroll-view scroll-x style="white-space: nowrap; width: 90%;">
				<view class="image-div">
					<view class="image-margin" v-for="(info, index) in imageList" :key="info._id"
						@click="imageClick(info, index)">
						<image :src="info.image_url" :class="{'image-border': currentIndex === index}"></image>
					</view>
					<view class="image-margin" v-for="(info, index) in imageList" :key="info._id"
						@click="imageClick(info, index)">
						<image :src="info.image_url" :class="{'image-border': currentIndex === index}"></image>
					</view>
				</view>
			</scroll-view>
		</view>
		
		<button class="download-button" hover-class="download-button-hover" @click.stop="shareFc()">
			<span>Download</span>
			<image src="@/static/icon/download-icon.png" mode="aspectFit"></image>
		</button>
	</view>
</template>

<script>
	import _app from '@/util/QS-SharePoster/app.js';
	import {
		getSharePoster
	} from '@/util/QS-SharePoster/QS-SharePoster.js';
	import HeadBar from '@/components/HeadBar.vue'

	export default {
		components: {
			HeadBar
		},
		data() {
			return {
				showClipper: false,
				poster: {},
				posterImage: '',
				canvasId: 'default_PosterCanvasId',
				userInfo: '',
				code: '',
				avatarImage: uni.getStorageSync('avatar_image'),
				defaultImage: 'https://vkceyugu.cdn.bspapp.com/VKCEYUGU-08ecbb66-149e-4d2b-93a0-fa6bc6e0e894/c33782ca-cd2f-4bfc-84eb-0713c52f522f.svg',
				currentImage: {},
				currentIndex: 0,
				imageList: uni.getStorageSync('first_image_list') || [],
				categoriesList: uni.getStorageSync('image_categories_list') || [],
				showBorder: false, // 默认是否显示边框
				maskCenterX: uni.upx2px(590) / 2,
				maskCenterY: uni.upx2px(590) / 2,
				handleCenterX: uni.upx2px(560), // 360
				handleCenterY: uni.upx2px(570), // 360
				handleWidth: 50, // 360
				maskSize: uni.upx2px(590),
				scale: 1,
				rotate: 0,
				mask_center_x: uni.upx2px(590) / 2,
				mask_center_y: uni.upx2px(590) / 2,
				handle_center_x: uni.upx2px(560), // 360
				handle_center_y: uni.upx2px(570), // 360
				scaleCurrent: 1,
				rotateCurrent: 0,
				touch_target: '',
				start_x: 0,
				start_y: 0,
				startInfo: 0,
				cansBorder: uni.upx2px(590), // 宽度 px
				imageScale: 10,
			};
		},
		onLoad() {
			this.init();
			this.getCategoriesList();
		},
		onShow() {
			// this.getShareInfo();
		},
		onReady() {
			const query = wx.createSelectorQuery();
			query.select('.avatar-div').boundingClientRect();
			query.exec(res => {
				this.startInfo = res[0];
			});
		},
		onShareAppMessage() {
			return {
				title: "UNNC 2022 Graduation",
				imageUrl: "/static/images/share-card.png",
				path: "/pages/main/index"
			}
		},
		onShareTimeline() {
			return {
				title: "UNNC 2022 Graduation",
				imageUrl: "/static/images/share-card.png",
				path: "/pages/main/index"
			}
		},
		methods: {
			/**
			 * 获取分类
			 */
			getCategoriesList() {
				uni.showLoading({
					title: 'Loading...',
					mask: true
				});
				uniCloud
					.callFunction({
						name: 'categories',
						data: {
							type: 'mpweixin'
						}
					})
					.then(res => {
						this.categoriesList = res.result.data;
						uni.setStorageSync('image_categories_list', res.result.data);
						if (this.categoriesList.length > 0) {
							this.$set(this.categoriesList[0], 'selected', true);
							this.getImagesList(this.categoriesList[0]._id, '', 'storage');
						}
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
			 * @param {Object} id
			 * 获取头像
			 */
			getImagesList(id, num, state) {
				uni.showLoading({
					title: 'Loading...',
					mask: true
				});
				uniCloud
					.callFunction({
						name: 'images',
						data: {
							type: 'mpweixin',
							categoryId: id
						}
					})
					.then(res => {
						this.imageList = res.result.data;
						state === 'storage' && uni.setStorageSync('first_image_list', res.result.data);
						if (this.imageList && this.imageList.length > 0) {
							if (num < 0) {
								this.currentImage = this.imageList[this.imageList.length - 1];
							} else {
								this.currentImage = this.imageList[0];
							}
							if (!(this.currentImage && this.currentImage.drag_state)) {
								this.initImage();
							}
						}
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
			 * 初始化
			 */
			async init() {
				this.code = await this.getWeixinCode();
				this.userInfo = uni.getStorageSync('user_info');
			},
			/**
			 * @param {Object} item
			 * 切换分类
			 */
			switchCategory(item, num) {
				let info = this.categoriesList.filter(el => el.selected);
				if (info && info.length > 0) {
					info[0].selected = false;
				}
				this.$set(item, 'selected', true);
				this.getImagesList(item._id, num);
			},
			/**
			 * @param {Object} num
			 * 切换图片
			 */
			switchAvatar(num) {
				this.touchAvatarBg(true);
				let currentIndex = this.imageList.findIndex(el => el._id === this.currentImage._id);
				if ((num > 0 && currentIndex < this.imageList.length - 1) || (num < 0 && currentIndex > 0)) {
					currentIndex += num;
					this.currentImage = this.imageList[currentIndex];
				} else {
					let currentType = this.categoriesList.findIndex(data => data.selected);
					currentType += num;
					if (this.categoriesList[currentType] && this.categoriesList[currentType]._id) {
						this.switchCategory(this.categoriesList[currentType], num);
					}
				}
				this.$nextTick(() => {
					if (!(this.currentImage && this.currentImage.drag_state)) {
						this.initImage();
					}
				});
			},
			/**
			 * @param {Object} e
			 * 设置挂件位置
			 */
			touchStart(e) {
				if (!(this.currentImage && this.currentImage.drag_state)) {
					return;
				}
				if (e.target.id == 'mask-img') {
					this.touch_target = 'mask-img';
					this.showBorder = true;
				} else if (e.target.id == 'drag-img') {
					this.touch_target = 'drag-img';
				} else {
					this.touch_target = '';
				}

				if (this.touch_target != '') {
					this.start_x = e.touches[0].clientX;
					this.start_y = e.touches[0].clientY;
				}
			},
			touchMove(e) {
				if (!(this.currentImage && this.currentImage.drag_state)) {
					return;
				}
				let current_x = e.touches[0].clientX;
				let current_y = e.touches[0].clientY;
				let moved_x = current_x - this.start_x;
				let moved_y = current_y - this.start_y;
				if (this.touch_target === 'mask-img') {
					this.maskCenterX = this.maskCenterX + moved_x;
					this.maskCenterY = this.maskCenterY + moved_y;
					this.handleCenterX = this.handleCenterX + moved_x;
					this.handleCenterY = this.handleCenterY + moved_y;
				}
				if (this.touch_target === 'drag-img') {
					this.handleCenterX = this.handleCenterX + moved_x;
					this.handleCenterY = this.handleCenterY + moved_y;
					let diff_x_before = this.handle_center_x - this.mask_center_x;
					let diff_y_before = this.handle_center_y - this.mask_center_y;
					let diff_x_after = this.handleCenterX - this.mask_center_x;
					let diff_y_after = this.handleCenterY - this.mask_center_y;
					let distance_before = Math.sqrt(diff_x_before * diff_x_before + diff_y_before * diff_y_before);
					let distance_after = Math.sqrt(diff_x_after * diff_x_after + diff_y_after * diff_y_after);
					let angle_before = (Math.atan2(diff_y_before, diff_x_before) / Math.PI) * 180;
					let angle_after = (Math.atan2(diff_y_after, diff_x_after) / Math.PI) * 180;
					this.scale = (distance_after / distance_before) * this.scaleCurrent;
					this.rotate = angle_after - angle_before + this.rotateCurrent;
					let width = this.scale * 100;
					if (width > 50) {
						this.handleWidth = 50;
					} else if (width < 10) {
						this.handleWidth = 10;
					} else {
						this.handleWidth = width;
					}
				}
				this.start_x = current_x;
				this.start_y = current_y;
			},
			touchEnd(e) {
				if (!(this.currentImage && this.currentImage.drag_state)) {
					return;
				}
				this.mask_center_x = this.maskCenterX;
				this.mask_center_y = this.maskCenterY;
				this.handle_center_x = this.handleCenterX;
				this.handle_center_y = this.handleCenterY;
				this.touch_target = '';
				this.scaleCurrent = this.scale;
				this.rotateCurrent = this.rotate;
			},
			/**
			 * 不显示border
			 */
			touchAvatarBg(state) {
				this.showBorder = state;
			},
			/**
			 * @param {Object} item
			 * 图片点击事件
			 */
			imageClick(item, index) {
				this.currentImage = item;
				this.currentIndex = index;
				if (!(this.currentImage && this.currentImage.drag_state)) {
					this.initImage();
				}
			},
			/**
			 * 还原设置
			 */
			initImage() {
				this.showBorder = true; // 默认是否显示边框
				this.maskCenterX = uni.upx2px(590) / 2;
				this.maskCenterY = uni.upx2px(590) / 2;
				this.handleCenterX = uni.upx2px(560);
				this.handleCenterY = uni.upx2px(570);
				this.maskSize = uni.upx2px(590);
				this.scale = 1;
				this.rotate = 0;
				this.mask_center_x = uni.upx2px(590) / 2;
				this.mask_center_y = uni.upx2px(590) / 2;
				this.handle_center_x = uni.upx2px(560);
				this.handle_center_y = uni.upx2px(570);
				this.scaleCurrent = 1;
				this.rotateCurrent = 0;
				this.touch_target = '';
				this.start_x = 0;
				this.start_y = 0;
			},
			/**
			 * 获取微信code
			 */
			getWeixinCode() {
				return new Promise((resolve, reject) => {
					uni.login({
						provider: 'weixin',
						success(res) {
							resolve(res.code);
						},
						fail(err) {
							reject(new Error('Fail to login'));
						}
					});
				});
			},
			/**
			 * 生成新头像
			 */
			async shareFc() {
				if (!this.avatarImage) {
					uni.showToast({
						title: 'Please fetch your avatar first',
						icon: 'none'
					});
					return;
				}
				uni.showLoading({
					title: 'Loading...',
					mask: true
				});
				const context = uni.createCanvasContext('default_PosterCanvasId', this);
				let cansBorder = this.cansBorder;
				const mask_size = this.scale * cansBorder;
				context.scale(this.imageScale, this.imageScale);
				context.clearRect(0, 0, this.cansBorder, this.cansBorder);
				uni.getImageInfo({
					src: this.avatarImage,
					success: response1 => {
						context.drawImage(response1.path, 0, 0, cansBorder, cansBorder);
						uni.getImageInfo({
							src: this.currentImage.image_url,
							success: response2 => {
								context.translate(this.mask_center_x, this.mask_center_y);
								context.rotate((this.rotate * Math.PI) / 180);
								context.drawImage(response2.path, -mask_size / 2, -mask_size /
									2, mask_size, mask_size);
								context.draw(false, () => {
									this.saveCans();
								});
							},
							fail: () => {
								context.draw(false, () => {
									this.saveCans();
								});
							}
						});
					}
				});
			},
			saveCans() {
				uni.canvasToTempFilePath({
						x: 0,
						y: 0,
						height: this.cansBorder * this.imageScale,
						width: this.cansBorder * this.imageScale,
						destWidth: this.cansBorder * this.imageScale,
						destHeight: this.cansBorder * this.imageScale,
						canvasId: this.canvasId,
						fileType: 'jpg',
						success: res => {
							this.posterImage = res.tempFilePath;
							this.savefile();
						},
						fail(res) {
							uni.hideLoading();
						}
					},
					this
				);
			},
			/**
			 * 保存图片
			 */
			savefile() {
				//获取相册授权
				let _self = this;
				uni.getSetting({
					success(res) {
						if (!res.authSetting['scope.writePhotosAlbum']) {
							uni.authorize({
								scope: 'scope.writePhotosAlbum',
								success() {
									//这里是用户同意授权后的回调
									_self.saveImgToLocal();
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
							_self.saveImgToLocal();
						}
					}
				});
			},
			/**
			 * @param {Object} e
			 * 保存到相册
			 */
			saveImgToLocal(e) {
				let _self = this;
				uni.saveImageToPhotosAlbum({
					filePath: _self.posterImage,
					success: function() {
						// _self.saveImageInfo();
						uni.setStorageSync('currentImage', _self.posterImage);
						uni.hideLoading();
						uni.navigateTo({
							url: '/pages/save-success/index'
						});
					},
					fail: e => {
						uni.hideLoading();
						uni.showToast({
							title: 'Fail to save',
							icon: 'none'
						});
					}
				});
			},
			/**
			 * 头像
			 */
			saveImageInfo() {
				uniCloud
					.callFunction({
						name: 'images',
						data: {
							imageInfo: this.currentImage,
							type: 'imageUsed'
						}
					})
					.then(res => {
						uni.hideLoading();
						uni.navigateTo({
							url: '/pages/save-success/index'
						});
					});
			},
			/**
			 * @param {Object} type
			 * 获取头像
			 */
			async getUserProfile(type) {
				let _this = this;
				uni.getUserProfile({
					desc: 'Fetch avatar information',
					success(result) {
						let data = {
							code: _this.code,
							signature: result.signature,
							encrypted_data: result.encryptedData,
							iv: result.iv,
							userInfo: result.userInfo
						};
						let info = data.userInfo.avatarUrl;
						if (type === 'createImages') {
							_this.avatarImage = info.substring(0, info.lastIndexOf('/') + 1) + '0';
							uni.setStorageSync('avatar_image', _this.avatarImage);
						}
						_this.postUserInfo(result.userInfo.nickName, type);
					},
					fail(fall) {
						console.log(fall)
						uni.showToast({
							icon: 'error',
							position: 'center',
							title: "Fail to fetch avatar information",
						})
					}
				});
			},
			/**
			 * @param {Object} nickName
			 * 存储用户数据
			 */
			async postUserInfo(nickName, type) {
				let that = this;
				this.code = await this.getWeixinCode();
				if (type === 'userLogin' || type === 'selectedImage') {
					uni.showLoading({
						title: 'Loading...',
						mask: true
					});
				}
				uniCloud
					.callFunction({
						name: 'user_mpweixin',
						data: {
							code: that.code,
							avatarImage: that.avatarImage,
							nickName,
							type: type === 'selectedImage' ? 'userLogin' : type
						}
					})
					.then(res => {
						uni.setStorageSync('user_info', res.result);
						this.userInfo = res.result;
						if (type === 'userLogin') {
							uni.hideLoading();
							this.navOriginal();
						} else if (type === 'selectedImage') {
							this.chooseImages(type);
						}
					})
					.catch(err => {
						console.log(err)
						uni.showToast({
							icon: 'error',
							position: 'center',
							title: "Fail to store user information",
						})
					});
			},
			/**
			 * 选择图片
			 */
			chooseImages(type) {
				uni.hideLoading();
				uni.chooseImage({
					count: 1, //默认9
					sizeType: ['original', 'compressed'], //可以指定是原图还是压缩图，默认二者都有
					sourceType: ['album', 'camera'], //从相册选择
					success: res => {
						// let filePath = res.tempFilePaths[0];
						this.avatarImage = res.tempFilePaths[0];
						this.showClipper = true
						// uniCloud
						// 	.uploadFile({
						// 		filePath: filePath,
						// 		cloudPath: `userChooseImage-${new Date().getTime()}.png`
						// 	})
						// 	.then(res => {
						// 		//获取到上传到云储存的url地址
						// 		uniCloud.getTempFileURL({
						// 			fileList: [res['fileID']]
						// 		}).then(res => {
						// 			this.avatarImage = res.fileList[0].tempFileURL
						// 		})
						// 		//获取到上传到云储存的url地址
						// 		uniCloud
						// 			.callFunction({
						// 				name: 'user_mpweixin',
						// 				data: {
						// 					userId: this.userInfo._id,
						// 					avatarImage: this.avatarImage,
						// 					type
						// 				}
						// 			})
						// 			.then();
						// 	});
					}
				});
			},
			navOriginal() {
				uni.navigateTo({
					url: '/pages/original-avatar/index'
				});
			},
		}
	};
</script>

<style lang="scss" scoped>
	.content {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		width: 100vw;
		height: 100vh;
		padding-top: 20px;
		box-sizing: border-box;

		.background-image {
			position: fixed;
			z-index: -1;
			top: 50vh;
			left: 50vw;
			width: 312px;
			height: 356px;
			transform: translate(-50%, -50%);
		}

		.image-selector {
			display: flex;
			justify-content: center;
			width: 100vw;
			background-color: rgba(255, 255, 255, 0.7);
			padding: 30rpx 0;
			margin-bottom: 40px;
			position: relative;

			.image-div {
				display: flex;
				align-items: center;

				image {
					box-sizing: border-box;
					width: 120rpx;
					height: 120rpx;
					flex-shrink: 0;
				}

				.image-margin:not(:last-child) {
					margin-right: 30rpx;
				}

				.image-border {
					border: 5rpx solid $uni-shadow-color;
				}
			}
		}

		.image-card {
			display: flex;
			justify-content: center;
			align-items: center;
			position: relative;
			background: rgba(255, 255, 255, 0.9);
			width: 80vw;
			height: $avatar-size + 60rpx;

			.avatar-div {
				position: relative;
				height: $avatar-size;
				width: $avatar-size;
				-webkit-box-orient: vertical;
				-webkit-box-direction: normal;
				-webkit-box-pack: center;
				-webkit-box-align: center;
				display: flex;
				flex-direction: column;
				align-items: center;
				justify-content: center;
				z-index: 1;

				.img {
					position: absolute;
					background-color: $uni-bg-color-grey;
					// border-radius: $uni-border-radius-lg;
					height: 100%;
					width: 100%;
					z-index: 0;
				}

				.avatar-default {
					box-sizing: content-box;
					// border-radius: $uni-border-radius-lg;
					height: 100%;
					width: 100%;
					margin: 6rpx;
					position: absolute;
					top: 0;
					left: 0;
					z-index: 10;
				}

				.avatar-border {
					margin: 0;
					border: 6rpx dashed $theme-blue-one;
				}

				.drag-div {
					position: absolute;
					top: $avatar-size;
					right: $avatar-size;
					width: 50rpx;
					height: 50rpx;
					background-color: $theme-blue-three;
					border-radius: $uni-border-radius-circle;
					padding: 10rpx;
					z-index: 9999999;
					display: flex;
					justify-content: center;
					align-items: center;

					.drag-img {
						width: 100%;
						height: 100%;
					}
				}
			}
		}
		
		.load-image-buttons {
			display: flex;
			justify-content: space-between;
			width: 80vw;
			margin: 30px 0;
			
			.button-container {
				display: flex;
				flex-direction: column;
				justify-content: center;
				align-items: center;
				
				.load-image-button {
					width: 120px;
					height: 35px;
					background-color: $theme-yellow;
					font-style: normal;
					font-weight: 600;
					font-size: 16px;
					color: $uni-text-color-black;
					margin-bottom: 10px;
					transition: 0.3s;
				}
				
				.load-image-button-hover {
					transform: scale(0.95);
					transition: 0.3s;
				}
				
				.button-descripsion {
					text-align: center;
					font-size: 12px;
					line-height: 20px;
				}
			}
		}

		.download-button {
			width: 200px;
			height: 40px;
			background-color: $theme-yellow;
			font-style: normal;
			font-weight: 600;
			font-size: 16px;
			color: $uni-text-color-black;
			transition: 0.3s;
			
			span {
				margin-right: 15px;
			}
			
			image {
				width: 18px;
				height: 18px;
			}
		}
		
		.download-button-hover {
			transform: scale(0.95);
			transition: 0.3s;
		}

		.hideCanvas {
			position: fixed;
			top: -99999upx;
			left: -99999upx;
			z-index: -99999;

			.default_PosterCanvasId {
				width: 1180rpx;
				height: 1180rpx;
			}
		}
	}

	@keyframes d-light {
		0% {
			transform: rotate(0deg);
		}

		25% {
			transform: rotate(10deg);
		}

		75% {
			transform: rotate(-10deg);
		}

		10% {
			transform: rotate(0deg);
		}
	}
</style>
