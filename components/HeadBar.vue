<template>
	<view class="head-bar">
		<image src="../static/images/unnc-logo.png" mode="aspectFit" class="logo"></image>
		<button open-type="share" class="share-button" hover-class="share-button-hover" plain @click.stop>
			<span>Share</span>
			<image src="../static/icon/share-icon.png" mode="aspectFit"></image>
		</button>
	</view>
</template>

<script>
	export default {
		name:"HeadBar",
		data() {
			return {
				
			};
		}
	}
</script>

<style lang="scss">
	.head-bar {
		display: flex;
		justify-content: space-between;
		align-items: center;
		position: fixed;
		top: 0;
		left: 0;
		width: 100vw;
		height: auto;
		
		.logo {
			width: 135px;
			height: 50px;
		}
		
		.share-button {
			margin: 0 18px 0 0;
			width: 86px;
			height: 29px;
			border: 1px solid $uni-bg-color;
			font-weight: 500;
			font-size: 12px;
			text-decoration-line: underline;
			color: $uni-text-color-inverse;
			
			span {
				margin-right: 10px;
			}
			
			image {
				width: 18px;
				height: 18px;
			}
		}
		
		.share-button-hover {
			background-color: $uni-bg-color;
			color: $theme-blue;
			transition: 0.3s;
		}
	}
</style>