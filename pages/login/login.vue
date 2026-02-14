<template>
	<view class="login-page">
		<!-- 顶部装饰背景 -->
		<view class="top-bg">
			<view class="bg-circle c1"></view>
			<view class="bg-circle c2"></view>
			<view class="bg-circle c3"></view>
		</view>

		<!-- Logo 区域 -->
		<view class="logo-area">
			<view class="logo-box">
				<text class="logo-text">S</text>
			</view>
			<text class="app-name">StockVue</text>
			<text class="app-slogan">智能股票行情 · 尽在掌握</text>
		</view>

		<!-- 表单区域 -->
		<view class="form-card">
			<view class="input-group">
				<view class="input-wrap" :class="{ 'input-focus': usernameFocus }">
					<text class="input-icon">👤</text>
					<input class="uni-input" type="text" v-model="username" placeholder="请输入手机号/邮箱"
						placeholder-style="color:#bbb" @focus="usernameFocus = true"
						@blur="usernameFocus = false" />
				</view>
				<view class="input-wrap" :class="{ 'input-focus': passwordFocus }">
					<text class="input-icon">🔒</text>
					<input class="uni-input" :password="!showPassword" v-model="password" placeholder="请输入密码"
						placeholder-style="color:#bbb" @focus="passwordFocus = true"
						@blur="passwordFocus = false" />
					<text class="toggle-pwd" @click="showPassword = !showPassword">
						{{ showPassword ? '🙈' : '👁️' }}
					</text>
				</view>
			</view>

			<!-- 记住密码 & 忘记密码 -->
			<view class="form-options">
				<view class="remember-me" @click="rememberMe = !rememberMe">
					<view class="checkbox" :class="{ checked: rememberMe }">
						<text v-if="rememberMe" class="check-mark">✓</text>
					</view>
					<text class="option-text">记住密码</text>
				</view>
				<text class="forget-pwd" @click="onForgetPwd">忘记密码?</text>
			</view>

			<!-- 登录按钮 -->
			<view class="btn-login" :class="{ 'btn-disabled': !canLogin }" @click="onLogin">
				<text class="btn-text" v-if="!loading">登 录</text>
				<view class="loading-dots" v-else>
					<view class="dot dot1"></view>
					<view class="dot dot2"></view>
					<view class="dot dot3"></view>
				</view>
			</view>

			<!-- 注册入口 -->
			<view class="register-link">
				<text class="register-text">还没有账号?</text>
				<text class="register-btn" @click="goToRegister">立即注册</text>
			</view>
		</view>

		<!-- 第三方登录 -->
		<view class="third-party">
			<view class="divider">
				<view class="divider-line"></view>
				<text class="divider-text">其他登录方式</text>
				<view class="divider-line"></view>
			</view>
			<view class="third-party-icons">
				<view class="tp-icon-wrap" @click="onThirdLogin('wechat')">
					<view class="tp-icon wechat">
						<text class="tp-icon-text">微</text>
					</view>
					<text class="tp-label">微信</text>
				</view>
				<view class="tp-icon-wrap" @click="onThirdLogin('qq')">
					<view class="tp-icon qq">
						<text class="tp-icon-text">Q</text>
					</view>
					<text class="tp-label">QQ</text>
				</view>
				<view class="tp-icon-wrap" @click="onThirdLogin('apple')">
					<view class="tp-icon apple">
						<text class="tp-icon-text">A</text>
					</view>
					<text class="tp-label">Apple</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				username: '',
				password: '',
				showPassword: false,
				rememberMe: false,
				loading: false,
				usernameFocus: false,
				passwordFocus: false
			}
		},
		computed: {
			canLogin() {
				return this.username.trim().length > 0 && this.password.trim().length > 0
			}
		},
		onLoad() {
			// 读取记住的账号
			const savedUser = uni.getStorageSync('saved_username')
			if (savedUser) {
				this.username = savedUser
				this.rememberMe = true
			}
		},
		methods: {
			onLogin() {
				if (!this.canLogin || this.loading) return

				// 简单校验
				if (this.username.trim().length < 3) {
					uni.showToast({ title: '请输入正确的账号', icon: 'none' })
					return
				}
				if (this.password.trim().length < 6) {
					uni.showToast({ title: '密码至少6位', icon: 'none' })
					return
				}

				this.loading = true

				// 模拟登录请求
				setTimeout(() => {
					this.loading = false

					// 记住密码
					if (this.rememberMe) {
						uni.setStorageSync('saved_username', this.username)
					} else {
						uni.removeStorageSync('saved_username')
					}

					uni.showToast({ title: '登录成功', icon: 'success' })

					setTimeout(() => {
						uni.reLaunch({ url: '/pages/index/index' })
					}, 1000)
				}, 1500)
			},
			goToRegister() {
				uni.navigateTo({ url: '/pages/register/register' })
			},
			onForgetPwd() {
				uni.showToast({ title: '请联系客服重置密码', icon: 'none' })
			},
			onThirdLogin(type) {
				const names = { wechat: '微信', qq: 'QQ', apple: 'Apple' }
				uni.showToast({ title: names[type] + '登录开发中...', icon: 'none' })
			}
		}
	}
</script>

<style>
	.login-page {
		min-height: 100vh;
		background: linear-gradient(180deg, #667eea 0%, #764ba2 100%);
		position: relative;
		overflow: hidden;
	}

	/* 顶部装饰圆 */
	.top-bg {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 600rpx;
	}

	.bg-circle {
		position: absolute;
		border-radius: 50%;
		opacity: 0.12;
		background: #fff;
	}

	.c1 {
		width: 400rpx;
		height: 400rpx;
		top: -120rpx;
		right: -80rpx;
	}

	.c2 {
		width: 250rpx;
		height: 250rpx;
		top: 60rpx;
		left: -60rpx;
	}

	.c3 {
		width: 150rpx;
		height: 150rpx;
		top: 200rpx;
		right: 100rpx;
	}

	/* Logo */
	.logo-area {
		display: flex;
		flex-direction: column;
		align-items: center;
		padding-top: 140rpx;
		position: relative;
		z-index: 1;
	}

	.logo-box {
		width: 130rpx;
		height: 130rpx;
		background: rgba(255, 255, 255, 0.25);
		border-radius: 32rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		backdrop-filter: blur(10px);
		border: 2rpx solid rgba(255, 255, 255, 0.3);
	}

	.logo-text {
		font-size: 64rpx;
		color: #fff;
		font-weight: bold;
		font-style: italic;
	}

	.app-name {
		font-size: 44rpx;
		color: #fff;
		font-weight: bold;
		margin-top: 20rpx;
		letter-spacing: 4rpx;
	}

	.app-slogan {
		font-size: 24rpx;
		color: rgba(255, 255, 255, 0.7);
		margin-top: 10rpx;
	}

	/* 表单卡片 */
	.form-card {
		margin: 60rpx 40rpx 0;
		background: #fff;
		border-radius: 28rpx;
		padding: 50rpx 40rpx 40rpx;
		box-shadow: 0 20rpx 60rpx rgba(0, 0, 0, 0.15);
		position: relative;
		z-index: 1;
	}

	.input-group {
		margin-bottom: 10rpx;
	}

	.input-wrap {
		display: flex;
		align-items: center;
		border: 2rpx solid #eee;
		border-radius: 16rpx;
		padding: 0 24rpx;
		height: 100rpx;
		margin-bottom: 24rpx;
		background: #fafafa;
		transition: all 0.3s;
	}

	.input-wrap.input-focus {
		border-color: #667eea;
		background: #fff;
		box-shadow: 0 0 0 4rpx rgba(102, 126, 234, 0.1);
	}

	.input-icon {
		font-size: 36rpx;
		margin-right: 16rpx;
	}

	.uni-input {
		flex: 1;
		font-size: 30rpx;
		color: #333;
		height: 100rpx;
	}

	.toggle-pwd {
		font-size: 36rpx;
		padding: 10rpx;
	}

	/* 选项行 */
	.form-options {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 40rpx;
		padding: 0 4rpx;
	}

	.remember-me {
		display: flex;
		align-items: center;
	}

	.checkbox {
		width: 36rpx;
		height: 36rpx;
		border: 2rpx solid #ccc;
		border-radius: 8rpx;
		margin-right: 12rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		transition: all 0.2s;
	}

	.checkbox.checked {
		background: #667eea;
		border-color: #667eea;
	}

	.check-mark {
		color: #fff;
		font-size: 22rpx;
		font-weight: bold;
	}

	.option-text {
		font-size: 26rpx;
		color: #888;
	}

	.forget-pwd {
		font-size: 26rpx;
		color: #667eea;
	}

	/* 登录按钮 */
	.btn-login {
		height: 96rpx;
		background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		border-radius: 48rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		box-shadow: 0 10rpx 30rpx rgba(102, 126, 234, 0.4);
		transition: all 0.2s;
	}

	.btn-login:active {
		transform: scale(0.97);
		opacity: 0.9;
	}

	.btn-disabled {
		opacity: 0.5;
		box-shadow: none;
	}

	.btn-text {
		font-size: 34rpx;
		color: #fff;
		font-weight: bold;
		letter-spacing: 12rpx;
	}

	/* 加载动画 */
	.loading-dots {
		display: flex;
		align-items: center;
		gap: 12rpx;
	}

	.dot {
		width: 16rpx;
		height: 16rpx;
		border-radius: 50%;
		background: #fff;
		animation: bounce 1.2s infinite ease-in-out;
	}

	.dot2 {
		animation-delay: 0.2s;
	}

	.dot3 {
		animation-delay: 0.4s;
	}

	@keyframes bounce {
		0%, 80%, 100% {
			transform: scale(0.6);
			opacity: 0.4;
		}

		40% {
			transform: scale(1);
			opacity: 1;
		}
	}

	/* 注册入口 */
	.register-link {
		display: flex;
		justify-content: center;
		align-items: center;
		margin-top: 32rpx;
	}

	.register-text {
		font-size: 26rpx;
		color: #999;
	}

	.register-btn {
		font-size: 26rpx;
		color: #667eea;
		font-weight: 600;
		margin-left: 8rpx;
	}

	/* 第三方登录 */
	.third-party {
		margin-top: 60rpx;
		padding: 0 60rpx 80rpx;
		position: relative;
		z-index: 1;
	}

	.divider {
		display: flex;
		align-items: center;
		margin-bottom: 40rpx;
	}

	.divider-line {
		flex: 1;
		height: 1rpx;
		background: rgba(255, 255, 255, 0.3);
	}

	.divider-text {
		font-size: 24rpx;
		color: rgba(255, 255, 255, 0.6);
		padding: 0 24rpx;
	}

	.third-party-icons {
		display: flex;
		justify-content: center;
		gap: 80rpx;
	}

	.tp-icon-wrap {
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.tp-icon {
		width: 96rpx;
		height: 96rpx;
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
		margin-bottom: 12rpx;
	}

	.tp-icon.wechat {
		background: #07c160;
	}

	.tp-icon.qq {
		background: #12b7f5;
	}

	.tp-icon.apple {
		background: #333;
	}

	.tp-icon-text {
		font-size: 36rpx;
		color: #fff;
		font-weight: bold;
	}

	.tp-label {
		font-size: 22rpx;
		color: rgba(255, 255, 255, 0.7);
	}
</style>
