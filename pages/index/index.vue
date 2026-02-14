<template>
	<view class="content">
		<image class="logo" src="/static/logo.png"></image>
		<view class="text-area">
			<text class="title">StockVue Demo</text>
		</view>
		<view class="menu-list">
			<view class="menu-item" @click="goToLogin">
				<view class="menu-icon">🔐</view>
				<view class="menu-info">
					<text class="menu-title">登录</text>
					<text class="menu-desc">账号密码登录 · 第三方登录</text>
				</view>
				<text class="menu-arrow">›</text>
			</view>
			<view class="menu-item" @click="goToRegister">
				<view class="menu-icon">📝</view>
				<view class="menu-info">
					<text class="menu-title">注册</text>
					<text class="menu-desc">手机验证码注册 · 密码强度检测</text>
				</view>
				<text class="menu-arrow">›</text>
			</view>
			<view class="menu-item" @click="goToKline">
				<view class="menu-icon">📈</view>
				<view class="menu-info">
					<text class="menu-title">股票K线图</text>
					<text class="menu-desc">ECharts 蜡烛图 + MA均线 + 成交量</text>
				</view>
				<text class="menu-arrow">›</text>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {}
		},
		onLoad() {},
		methods: {
			goToLogin() {
				uni.navigateTo({ url: '/pages/login/login' })
			},
			goToRegister() {
				uni.navigateTo({ url: '/pages/register/register' })
			},
			goToKline() {
				uni.navigateTo({ url: '/pages/kline/kline' })
			}
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		padding-top: 60rpx;
		min-height: 100vh;
		background-color: #f5f6fa;
	}

	.logo {
		height: 160rpx;
		width: 160rpx;
		margin-bottom: 30rpx;
	}

	.text-area {
		display: flex;
		justify-content: center;
		margin-bottom: 60rpx;
	}

	.title {
		font-size: 40rpx;
		color: #333;
		font-weight: bold;
	}

	.menu-list {
		width: 90%;
	}

	.menu-item {
		display: flex;
		align-items: center;
		background-color: #ffffff;
		border-radius: 20rpx;
		padding: 30rpx;
		margin-bottom: 24rpx;
		box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.06);
		transition: transform 0.2s;
	}

	.menu-item:active {
		transform: scale(0.98);
		opacity: 0.9;
	}

	.menu-icon {
		font-size: 48rpx;
		margin-right: 24rpx;
	}

	.menu-info {
		flex: 1;
		display: flex;
		flex-direction: column;
	}

	.menu-title {
		font-size: 32rpx;
		color: #333;
		font-weight: 600;
		margin-bottom: 6rpx;
	}

	.menu-desc {
		font-size: 24rpx;
		color: #999;
	}

	.menu-arrow {
		font-size: 40rpx;
		color: #ccc;
		font-weight: 300;
	}
</style>
