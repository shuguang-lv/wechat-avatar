<script>
	export default {
		globalData: {
			shareTitle: 'UNNC Summer 2022 Graduation'
		},
		data() {
			return {
				shareInfo: uni.getStorageSync('shareInfo')
			}
		},
		onLaunch: function() {
			uni.removeStorageSync('avatar_image');
			uni.removeStorageSync('user_info');
		},
		onShow: function() {
			this.getShareInfo()
		},
		onHide: function() {},
		onShareAppMessage: function() {
			return this.shareInfo;
		},
		onShareTimeline: function() {
			return this.shareInfo;
		},
		methods: {
			/**
			 * 获取分享字典
			 */
			getShareInfo() {
				uni.showLoading({
					title: 'Loading...',
					mask: true
				});
				uniCloud
					.callFunction({
						name: 'code-mag',
						data: {
							type: 'mpweixinGet'
						}
					})
					.then(res => {
						if (res.result.data && res.result.data.length > 0) {
							this.shareInfo = res.result.data.find(el => el.code === 'mpwx_share');
							uni.setStorageSync('shareInfo', this.shareInfo);
							uni.setStorageSync('background_info', res.result.data);
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
		}
	}
</script>

<style lang="scss">
	/*每个页面公共css*/
	@import url("./static/icon/icon.css");

	page {
		background-color: $theme-blue;
		color: $uni-text-color-inverse;
	}

	button {
		display: flex;
		align-items: center;
		justify-content: center;
		border-radius: 0;
		border: none;
		margin: 0;
	}
	
	button::after {
	 border: none;
	}
</style>
