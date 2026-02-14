<template>
	<view class="register-page">
		<!-- 顶部装饰背景 -->
		<view class="top-bg">
			<view class="bg-circle c1"></view>
			<view class="bg-circle c2"></view>
		</view>

		<!-- 标题区域 -->
		<view class="header-area">
			<text class="page-title">创建账号</text>
			<text class="page-subtitle">注册 StockVue，开启智能投资之旅</text>
		</view>

		<!-- 表单卡片 -->
		<view class="form-card">
			<!-- 手机号 -->
			<view class="input-wrap" :class="{ 'input-focus': phoneFocus }">
				<text class="input-icon">📱</text>
				<input class="uni-input" type="number" v-model="phone" placeholder="请输入手机号" maxlength="11"
					placeholder-style="color:#bbb" @focus="phoneFocus = true" @blur="phoneFocus = false" />
			</view>

			<!-- 验证码 -->
			<view class="input-wrap" :class="{ 'input-focus': codeFocus }">
				<text class="input-icon">🔑</text>
				<input class="uni-input" type="number" v-model="code" placeholder="请输入验证码" maxlength="6"
					placeholder-style="color:#bbb" @focus="codeFocus = true" @blur="codeFocus = false" />
				<view class="code-btn" :class="{ 'code-disabled': countdown > 0 }" @click="sendCode">
					<text class="code-btn-text">{{ countdown > 0 ? countdown + 's' : '获取验证码' }}</text>
				</view>
			</view>

			<!-- 用户名 -->
			<view class="input-wrap" :class="{ 'input-focus': nicknameFocus }">
				<text class="input-icon">👤</text>
				<input class="uni-input" type="text" v-model="nickname" placeholder="请设置用户名"
					placeholder-style="color:#bbb" @focus="nicknameFocus = true"
					@blur="nicknameFocus = false" />
			</view>

			<!-- 密码 -->
			<view class="input-wrap" :class="{ 'input-focus': passwordFocus }">
				<text class="input-icon">🔒</text>
				<input class="uni-input" :password="!showPassword" v-model="password" placeholder="请设置密码（至少6位）"
					placeholder-style="color:#bbb" @focus="passwordFocus = true"
					@blur="passwordFocus = false" />
				<text class="toggle-pwd" @click="showPassword = !showPassword">
					{{ showPassword ? '🙈' : '👁️' }}
				</text>
			</view>

			<!-- 确认密码 -->
			<view class="input-wrap" :class="{ 'input-focus': confirmFocus }">
				<text class="input-icon">🔒</text>
				<input class="uni-input" :password="!showConfirm" v-model="confirmPassword"
					placeholder="请确认密码" placeholder-style="color:#bbb" @focus="confirmFocus = true"
					@blur="confirmFocus = false" />
				<text class="toggle-pwd" @click="showConfirm = !showConfirm">
					{{ showConfirm ? '🙈' : '👁️' }}
				</text>
			</view>

			<!-- 密码强度指示器 -->
			<view class="pwd-strength" v-if="password.length > 0">
				<view class="strength-bar">
					<view class="strength-fill" :style="{ width: strengthPercent + '%', background: strengthColor }">
					</view>
				</view>
				<text class="strength-text" :style="{ color: strengthColor }">{{ strengthLabel }}</text>
			</view>

			<!-- 用户协议 -->
			<view class="agreement" @click="agreeTerms = !agreeTerms">
				<view class="checkbox" :class="{ checked: agreeTerms }">
					<text v-if="agreeTerms" class="check-mark">✓</text>
				</view>
				<text class="agreement-text">我已阅读并同意</text>
				<text class="agreement-link" @click.stop="showAgreement">《用户服务协议》</text>
				<text class="agreement-text">和</text>
				<text class="agreement-link" @click.stop="showPrivacy">《隐私政策》</text>
			</view>

			<!-- 注册按钮 -->
			<view class="btn-register" :class="{ 'btn-disabled': !canRegister }" @click="onRegister">
				<text class="btn-text" v-if="!loading">注 册</text>
				<view class="loading-dots" v-else>
					<view class="dot dot1"></view>
					<view class="dot dot2"></view>
					<view class="dot dot3"></view>
				</view>
			</view>

			<!-- 返回登录 -->
			<view class="login-link">
				<text class="link-text">已有账号?</text>
				<text class="link-btn" @click="goToLogin">返回登录</text>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				phone: '',
				code: '',
				nickname: '',
				password: '',
				confirmPassword: '',
				showPassword: false,
				showConfirm: false,
				agreeTerms: false,
				loading: false,
				countdown: 0,
				timer: null,
				phoneFocus: false,
				codeFocus: false,
				nicknameFocus: false,
				passwordFocus: false,
				confirmFocus: false
			}
		},
		computed: {
			canRegister() {
				return (
					this.phone.trim().length === 11 &&
					this.code.trim().length >= 4 &&
					this.nickname.trim().length > 0 &&
					this.password.trim().length >= 6 &&
					this.confirmPassword === this.password &&
					this.agreeTerms
				)
			},
			passwordStrength() {
				const pwd = this.password
				if (pwd.length === 0) return 0
				let score = 0
				if (pwd.length >= 6) score++
				if (pwd.length >= 10) score++
				if (/[a-z]/.test(pwd) && /[A-Z]/.test(pwd)) score++
				if (/\d/.test(pwd)) score++
				if (/[^a-zA-Z0-9]/.test(pwd)) score++
				return Math.min(score, 4)
			},
			strengthPercent() {
				return this.passwordStrength * 25
			},
			strengthColor() {
				const colors = ['#ff4757', '#ff6b6b', '#ffa502', '#2ed573', '#1dd1a1']
				return colors[this.passwordStrength]
			},
			strengthLabel() {
				const labels = ['极弱', '弱', '一般', '强', '极强']
				return labels[this.passwordStrength]
			}
		},
		beforeDestroy() {
			if (this.timer) clearInterval(this.timer)
		},
		methods: {
			sendCode() {
				if (this.countdown > 0) return

				if (this.phone.trim().length !== 11) {
					uni.showToast({ title: '请输入正确的手机号', icon: 'none' })
					return
				}

				// 模拟发送验证码
				uni.showToast({ title: '验证码已发送', icon: 'success' })
				this.countdown = 60
				this.timer = setInterval(() => {
					this.countdown--
					if (this.countdown <= 0) {
						clearInterval(this.timer)
						this.timer = null
					}
				}, 1000)
			},
			onRegister() {
				if (!this.canRegister || this.loading) return

				if (this.password !== this.confirmPassword) {
					uni.showToast({ title: '两次密码不一致', icon: 'none' })
					return
				}

				this.loading = true

				// 模拟注册请求
				setTimeout(() => {
					this.loading = false
					uni.showToast({ title: '注册成功', icon: 'success' })

					setTimeout(() => {
						uni.navigateBack()
					}, 1000)
				}, 1500)
			},
			goToLogin() {
				uni.navigateBack()
			},
			showAgreement() {
				uni.showModal({
					title: '用户服务协议',
					content: '这是用户服务协议的内容示例。实际项目中请替换为真实协议内容。',
					showCancel: false
				})
			},
			showPrivacy() {
				uni.showModal({
					title: '隐私政策',
					content: '这是隐私政策的内容示例。实际项目中请替换为真实隐私政策内容。',
					showCancel: false
				})
			}
		}
	}
</script>

<style>
	.register-page {
		min-height: 100vh;
		background: linear-gradient(180deg, #43e97b 0%, #38f9d7 50%, #667eea 100%);
		position: relative;
		overflow: hidden;
		padding-bottom: 60rpx;
	}

	/* 装饰圆 */
	.top-bg {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 500rpx;
	}

	.bg-circle {
		position: absolute;
		border-radius: 50%;
		opacity: 0.1;
		background: #fff;
	}

	.c1 {
		width: 350rpx;
		height: 350rpx;
		top: -100rpx;
		left: -80rpx;
	}

	.c2 {
		width: 200rpx;
		height: 200rpx;
		top: 100rpx;
		right: -40rpx;
	}

	/* 标题 */
	.header-area {
		padding-top: 120rpx;
		padding-left: 60rpx;
		position: relative;
		z-index: 1;
		margin-bottom: 40rpx;
	}

	.page-title {
		font-size: 52rpx;
		color: #fff;
		font-weight: bold;
		display: block;
	}

	.page-subtitle {
		font-size: 26rpx;
		color: rgba(255, 255, 255, 0.75);
		margin-top: 12rpx;
		display: block;
	}

	/* 表单卡片 */
	.form-card {
		margin: 0 36rpx;
		background: #fff;
		border-radius: 28rpx;
		padding: 44rpx 36rpx 36rpx;
		box-shadow: 0 20rpx 60rpx rgba(0, 0, 0, 0.12);
		position: relative;
		z-index: 1;
	}

	.input-wrap {
		display: flex;
		align-items: center;
		border: 2rpx solid #eee;
		border-radius: 16rpx;
		padding: 0 24rpx;
		height: 96rpx;
		margin-bottom: 22rpx;
		background: #fafafa;
		transition: all 0.3s;
	}

	.input-wrap.input-focus {
		border-color: #43e97b;
		background: #fff;
		box-shadow: 0 0 0 4rpx rgba(67, 233, 123, 0.12);
	}

	.input-icon {
		font-size: 34rpx;
		margin-right: 14rpx;
	}

	.uni-input {
		flex: 1;
		font-size: 28rpx;
		color: #333;
		height: 96rpx;
	}

	.toggle-pwd {
		font-size: 34rpx;
		padding: 10rpx;
	}

	/* 验证码按钮 */
	.code-btn {
		background: linear-gradient(135deg, #43e97b, #38f9d7);
		border-radius: 12rpx;
		padding: 0 24rpx;
		height: 60rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		white-space: nowrap;
		flex-shrink: 0;
	}

	.code-disabled {
		opacity: 0.5;
	}

	.code-btn-text {
		font-size: 24rpx;
		color: #fff;
		font-weight: 600;
	}

	/* 密码强度 */
	.pwd-strength {
		display: flex;
		align-items: center;
		margin-bottom: 24rpx;
		padding: 0 4rpx;
	}

	.strength-bar {
		flex: 1;
		height: 8rpx;
		background: #eee;
		border-radius: 4rpx;
		overflow: hidden;
		margin-right: 16rpx;
	}

	.strength-fill {
		height: 100%;
		border-radius: 4rpx;
		transition: all 0.3s;
	}

	.strength-text {
		font-size: 22rpx;
		font-weight: 600;
		flex-shrink: 0;
	}

	/* 用户协议 */
	.agreement {
		display: flex;
		align-items: center;
		flex-wrap: wrap;
		margin-bottom: 32rpx;
		padding: 0 4rpx;
	}

	.checkbox {
		width: 34rpx;
		height: 34rpx;
		border: 2rpx solid #ccc;
		border-radius: 8rpx;
		margin-right: 10rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		transition: all 0.2s;
		flex-shrink: 0;
	}

	.checkbox.checked {
		background: #43e97b;
		border-color: #43e97b;
	}

	.check-mark {
		color: #fff;
		font-size: 20rpx;
		font-weight: bold;
	}

	.agreement-text {
		font-size: 24rpx;
		color: #999;
	}

	.agreement-link {
		font-size: 24rpx;
		color: #667eea;
	}

	/* 注册按钮 */
	.btn-register {
		height: 96rpx;
		background: linear-gradient(135deg, #43e97b 0%, #38f9d7 100%);
		border-radius: 48rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		box-shadow: 0 10rpx 30rpx rgba(67, 233, 123, 0.35);
		transition: all 0.2s;
	}

	.btn-register:active {
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

	/* 返回登录 */
	.login-link {
		display: flex;
		justify-content: center;
		align-items: center;
		margin-top: 28rpx;
	}

	.link-text {
		font-size: 26rpx;
		color: #999;
	}

	.link-btn {
		font-size: 26rpx;
		color: #667eea;
		font-weight: 600;
		margin-left: 8rpx;
	}
</style>
