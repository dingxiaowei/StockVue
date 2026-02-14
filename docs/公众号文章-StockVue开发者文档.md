# 从零到一：用 uni-app + Vue 3 + ECharts 打造股票行情应用 StockVue

> 本文将完整介绍 StockVue 项目的技术架构、核心功能实现与关键代码解析，适合有一定前端基础、想学习 uni-app 跨端开发和 ECharts 数据可视化的开发者阅读。

---

## 一、项目简介

**StockVue** 是一款基于 **uni-app (Vue 3)** 的股票行情演示应用，包含用户登录注册体系和专业级 K 线图表展示。项目支持一套代码多端运行，覆盖 H5、Android、iOS 以及微信/支付宝/百度/头条等多个小程序平台。

### 核心特性

- 基于 **Vue 3 Composition API** 与 uni-app 跨端框架
- **ECharts 5.5** 实现专业 K 线蜡烛图 + MA 均线 + 成交量图表
- 精美的登录/注册界面，支持表单验证、密码强度检测
- 暗色主题 K 线图，沉浸式行情浏览体验
- 完整的多端适配方案

---

## 二、技术栈概览

| 技术 | 版本/说明 |
|------|----------|
| uni-app | 跨端框架 |
| Vue | 3.x (SSR Ready) |
| ECharts | 5.5.0 (CDN) |
| CSS预处理 | SCSS |
| 单位方案 | rpx 响应式 |
| 构建工具 | HBuilderX / uni-app CLI |

---

## 三、项目结构

```
StockVue/
├── App.vue                    # 根组件
├── main.js                    # 应用入口（Vue 2/3 兼容）
├── manifest.json              # 多端配置（App/小程序）
├── pages.json                 # 页面路由与导航栏配置
├── uni.scss                   # 全局 SCSS 变量
├── pages/
│   ├── index/index.vue        # 首页 - 功能导航
│   ├── login/login.vue        # 登录页
│   ├── register/register.vue  # 注册页
│   └── kline/kline.vue        # K线图页
└── static/
    └── logo.png               # 应用 Logo
```

---

## 四、应用入口与初始化

### 4.1 main.js — 兼容 Vue 2/3 的入口设计

```javascript
// Vue 2 模式（条件编译）
// #ifndef VUE3
import Vue from 'vue'
import './uni.promisify.adaptor'
Vue.config.productionTip = false
App.mpType = 'app'
const app = new Vue({ ...App })
app.$mount()
// #endif

// Vue 3 模式
// #ifdef VUE3
import { createSSRApp } from 'vue'
export function createApp() {
  const app = createSSRApp(App)
  return { app }
}
// #endif
```

**要点解析：**

- 使用 uni-app 的 **条件编译**（`#ifdef` / `#ifndef`）实现 Vue 2 和 Vue 3 双版本兼容
- Vue 3 模式采用 `createSSRApp`，天然支持 SSR 场景
- 这种设计让项目可以在不同 Vue 版本间无缝切换

### 4.2 pages.json — 路由与导航栏配置

```json
{
  "pages": [
    {
      "path": "pages/index/index",
      "style": { "navigationBarTitleText": "StockVue" }
    },
    {
      "path": "pages/login/login",
      "style": {
        "navigationBarTitleText": "登录",
        "navigationBarBackgroundColor": "#667eea",
        "navigationStyle": "custom"
      }
    },
    {
      "path": "pages/register/register",
      "style": {
        "navigationBarBackgroundColor": "#43e97b",
        "navigationStyle": "custom"
      }
    },
    {
      "path": "pages/kline/kline",
      "style": {
        "navigationBarBackgroundColor": "#0f0f23",
        "navigationBarTextStyle": "white"
      }
    }
  ]
}
```

**设计亮点：**

- 登录页和注册页使用 `"navigationStyle": "custom"` 实现自定义导航栏，配合渐变背景打造沉浸感
- K 线页使用深色导航栏 `#0f0f23`，与暗色主题图表统一

---

## 五、核心功能实现

### 5.1 登录页 — 精美交互体验

登录页是用户接触应用的第一个关键页面。我们在 UI 和交互细节上做了大量优化。

#### 渐变背景 + 毛玻璃 Logo

```css
.login-page {
  min-height: 100vh;
  background: linear-gradient(180deg, #667eea 0%, #764ba2 100%);
  position: relative;
  overflow: hidden;
}

.logo-box {
  width: 130rpx;
  height: 130rpx;
  background: rgba(255, 255, 255, 0.25);
  border-radius: 32rpx;
  backdrop-filter: blur(10px);
  border: 2rpx solid rgba(255, 255, 255, 0.3);
}
```

紫色渐变背景搭配半透明毛玻璃效果的 Logo，营造高级质感。

#### 输入框焦点动效

```html
<view class="input-wrap" :class="{ 'input-focus': usernameFocus }">
  <text class="input-icon">👤</text>
  <input v-model="username" placeholder="请输入手机号/邮箱"
    @focus="usernameFocus = true"
    @blur="usernameFocus = false" />
</view>
```

```css
.input-wrap.input-focus {
  border-color: #667eea;
  background: #fff;
  box-shadow: 0 0 0 4rpx rgba(102, 126, 234, 0.1);
}
```

通过监听 `focus` / `blur` 事件动态切换类名，实现输入框聚焦时的边框高亮和阴影扩散效果。

#### 登录按钮加载动画

```html
<view class="btn-login" :class="{ 'btn-disabled': !canLogin }" @click="onLogin">
  <text class="btn-text" v-if="!loading">登 录</text>
  <view class="loading-dots" v-else>
    <view class="dot dot1"></view>
    <view class="dot dot2"></view>
    <view class="dot dot3"></view>
  </view>
</view>
```

```css
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

.dot { animation: bounce 1.2s infinite ease-in-out; }
.dot2 { animation-delay: 0.2s; }
.dot3 { animation-delay: 0.4s; }
```

三个圆点依次跳动的加载动画，比传统 loading 图标更有视觉活力。

#### 记住密码 — localStorage 持久化

```javascript
onLoad() {
  const savedUser = uni.getStorageSync('saved_username')
  if (savedUser) {
    this.username = savedUser
    this.rememberMe = true
  }
},
methods: {
  onLogin() {
    // ... 登录逻辑
    if (this.rememberMe) {
      uni.setStorageSync('saved_username', this.username)
    } else {
      uni.removeStorageSync('saved_username')
    }
  }
}
```

利用 `uni.getStorageSync` / `uni.setStorageSync` 实现跨端兼容的本地存储方案。

---

### 5.2 注册页 — 实时密码强度检测

注册页的亮点在于实时密码强度检测算法，给用户即时的安全反馈。

#### 密码强度评估算法

```javascript
computed: {
  passwordStrength() {
    const pwd = this.password
    if (pwd.length === 0) return 0
    let score = 0
    if (pwd.length >= 6) score++      // 长度 >= 6
    if (pwd.length >= 10) score++     // 长度 >= 10
    if (/[a-z]/.test(pwd) && /[A-Z]/.test(pwd)) score++ // 大小写混合
    if (/\d/.test(pwd)) score++       // 包含数字
    if (/[^a-zA-Z0-9]/.test(pwd)) score++ // 包含特殊字符
    return Math.min(score, 4)
  },
  strengthPercent() {
    return this.passwordStrength * 25  // 0% ~ 100%
  },
  strengthColor() {
    const colors = ['#ff4757', '#ff6b6b', '#ffa502', '#2ed573', '#1dd1a1']
    return colors[this.passwordStrength]
  },
  strengthLabel() {
    const labels = ['极弱', '弱', '一般', '强', '极强']
    return labels[this.passwordStrength]
  }
}
```

**五级安全评估体系：**

| 等级 | 颜色 | 条件 |
|------|------|------|
| 极弱 | 红色 | 仅有字符 |
| 弱 | 浅红 | 满足1项规则 |
| 一般 | 橙色 | 满足2项规则 |
| 强 | 绿色 | 满足3项规则 |
| 极强 | 翠绿 | 满足全部4项规则 |

#### 进度条可视化

```html
<view class="pwd-strength" v-if="password.length > 0">
  <view class="strength-bar">
    <view class="strength-fill"
      :style="{ width: strengthPercent + '%', background: strengthColor }">
    </view>
  </view>
  <text class="strength-text" :style="{ color: strengthColor }">
    {{ strengthLabel }}
  </text>
</view>
```

动态宽度 + 渐变色彩，让用户直观感受密码安全等级变化。

#### 验证码倒计时

```javascript
sendCode() {
  if (this.countdown > 0) return
  if (this.phone.trim().length !== 11) {
    uni.showToast({ title: '请输入正确的手机号', icon: 'none' })
    return
  }
  uni.showToast({ title: '验证码已发送', icon: 'success' })
  this.countdown = 60
  this.timer = setInterval(() => {
    this.countdown--
    if (this.countdown <= 0) {
      clearInterval(this.timer)
      this.timer = null
    }
  }, 1000)
}
```

60 秒倒计时防止重复发送，配合 `beforeDestroy` 中的定时器清理，避免内存泄漏。

---

### 5.3 K 线图 — ECharts + renderjs 的深度集成

K 线图是本项目的技术核心，涉及 ECharts 在 uni-app 中的最佳实践。

#### 为什么使用 renderjs？

uni-app 的逻辑层和视图层是隔离的。要在视图层直接操作 DOM 加载 ECharts，需要使用 **renderjs** 技术。这是 uni-app 独有的能力，让我们能够在 App 端和 H5 端高效地使用 ECharts。

#### CDN 动态加载 ECharts

```javascript
// renderjs 模块
module.exports = {
  mounted() {
    if (typeof window.echarts === 'object') {
      this.bindInitChart()
    } else {
      const script = document.createElement('script')
      script.src = 'https://cdn.jsdelivr.net/npm/echarts@5.5.0/dist/echarts.min.js'
      script.onload = () => {
        this.bindInitChart()
      }
      document.head.appendChild(script)
    }
  }
}
```

**设计思路：**
1. 先检测 ECharts 是否已加载（避免重复加载）
2. 动态创建 `<script>` 标签按需加载
3. 加载完成后初始化图表

#### 逻辑层与视图层通信

```html
<view id="klineChart" class="kline-chart"
  :prop="optionData"
  :change:prop="bindEcharts.bindUpdateChart">
</view>
```

- `:prop="optionData"` — 将逻辑层数据传递给 renderjs
- `:change:prop="bindEcharts.bindUpdateChart"` — 数据变化时自动触发 renderjs 方法更新图表

这是 uni-app renderjs 的核心通信机制。

#### K 线蜡烛图配置

```javascript
series: [
  {
    name: 'K线',
    type: 'candlestick',
    data: data.klineData,  // [开盘, 收盘, 最低, 最高]
    itemStyle: {
      color: '#ec0000',        // 阳线填充（涨）
      color0: '#00da3c',       // 阴线填充（跌）
      borderColor: '#ec0000',  // 阳线边框
      borderColor0: '#00da3c'  // 阴线边框
    }
  }
]
```

采用中国股市配色惯例：**红涨绿跌**。

#### MA 均线算法

```javascript
function calcMA(dayCount, klineData) {
  const result = []
  for (let i = 0; i < klineData.length; i++) {
    if (i < dayCount - 1) {
      result.push('-')  // 数据不足时用占位符
      continue
    }
    let sum = 0
    for (let j = 0; j < dayCount; j++) {
      sum += klineData[i - j][1]  // 取收盘价
    }
    result.push((sum / dayCount).toFixed(2))
  }
  return result
}
```

实现了 **MA5（黄色）、MA10（蓝色）、MA20（紫色）** 三条移动平均线，帮助判断股价趋势。

#### 成交量柱状图联动

```javascript
{
  name: '成交量',
  type: 'bar',
  xAxisIndex: 1,   // 绑定第二个 x 轴
  yAxisIndex: 1,   // 绑定第二个 y 轴
  data: data.volumes,
  itemStyle: {
    color: function(params) {
      // 根据当日涨跌决定柱子颜色
      return volumeColors[params.dataIndex]
    }
  }
}
```

成交量柱子颜色与 K 线涨跌保持一致，配合 `dataZoom` 实现双图表联动缩放。

#### 暗色主题

```javascript
option = {
  backgroundColor: '#1a1a2e',
  tooltip: {
    backgroundColor: 'rgba(30,30,50,0.9)',
    borderColor: '#444',
    textStyle: { color: '#eee' }
  },
  // 坐标轴、分割线等均采用暗色调
}
```

暗色主题不仅视觉效果专业，还能降低长时间看盘的视觉疲劳。

---

## 六、多端适配策略

### 6.1 manifest.json 多平台配置

StockVue 通过 `manifest.json` 同时配置了以下平台：

```json
{
  "app-plus": { ... },     // App (Android/iOS)
  "mp-weixin": { ... },    // 微信小程序
  "mp-alipay": { ... },    // 支付宝小程序
  "mp-baidu": { ... },     // 百度小程序
  "mp-toutiao": { ... },   // 头条小程序
  "vueVersion": "3"        // 指定 Vue 3
}
```

### 6.2 rpx 响应式单位

整个项目统一使用 `rpx` 作为尺寸单位：

```css
.logo { width: 160rpx; height: 160rpx; }
.form-card { margin: 60rpx 40rpx 0; border-radius: 28rpx; }
.btn-login { height: 96rpx; border-radius: 48rpx; }
```

`rpx` 是 uni-app 提供的响应式单位，以 750rpx 为基准宽度，在不同屏幕尺寸下自动等比缩放。

---

## 七、UI/UX 设计亮点

### 7.1 页面级配色系统

| 页面 | 主色调 | 设计风格 |
|------|--------|----------|
| 首页 | 浅灰 `#f5f6fa` | 卡片式导航，简洁明快 |
| 登录页 | 紫色渐变 `#667eea → #764ba2` | 毛玻璃装饰，高端大气 |
| 注册页 | 绿色渐变 `#43e97b → #38f9d7` | 清新活力，引导注册 |
| K线页 | 深色 `#0f0f23` | 专业暗色主题，沉浸看盘 |

### 7.2 微交互细节

- **按钮点击反馈**：`transform: scale(0.97)` 轻微缩放
- **输入框聚焦**：边框变色 + 阴影扩散双重反馈
- **密码强度**：实时颜色渐变 + 进度条动画
- **加载状态**：三点跳动动画替代传统 loading
- **装饰圆**：页面背景的半透明圆形装饰，增加层次感

---

## 八、快速上手

### 8.1 环境准备

1. 下载安装 **HBuilderX**（推荐）或使用 uni-app CLI
2. 确保 Node.js >= 14

### 8.2 运行项目

**方式一：HBuilderX**
- 将项目导入 HBuilderX
- 点击「运行」→ 选择目标平台（浏览器 / 模拟器 / 真机）

**方式二：CLI**
```bash
# 安装依赖
npm install

# H5 运行
npm run dev:h5

# 微信小程序
npm run dev:mp-weixin

# App
npm run dev:app
```

### 8.3 项目配置

- **应用信息**：修改 `manifest.json` 中的 `name`、`appid`
- **页面路由**：在 `pages.json` 中添加/修改页面路径
- **全局样式**：通过 `uni.scss` 调整主题变量

---

## 九、后续规划

StockVue 目前是演示版本，以下是后续迭代方向：

- [ ] **接入真实行情 API**（如东方财富、新浪财经接口）
- [ ] **引入 Pinia 状态管理**，统一管理全局用户/行情数据
- [ ] **WebSocket 实时推送**，实现分时图实时更新
- [ ] **自选股列表**，支持股票搜索和收藏
- [ ] **技术指标扩展**，增加 MACD、KDJ、BOLL 等指标
- [ ] **后端 API 对接**，实现真实的登录注册流程
- [ ] **微信小程序登录**，对接微信 `wx.login` 授权体系

---

## 十、总结

StockVue 虽然是一个演示项目，但涵盖了 uni-app 跨端开发中的多个核心知识点：

1. **Vue 3 在 uni-app 中的使用** — SSR App 创建、条件编译
2. **renderjs 与 ECharts 集成** — 跨层通信、CDN 动态加载
3. **专业金融图表** — K 线蜡烛图、均线算法、成交量联动
4. **精美 UI 实现** — 渐变背景、毛玻璃效果、微交互动画
5. **多端适配** — rpx 响应式、多平台配置
6. **表单最佳实践** — 验证、密码强度、倒计时、记住密码

希望这篇文章能帮助你快速了解如何用现代前端技术栈构建一款专业的股票行情应用。如果觉得有用，欢迎 Star 项目仓库！

---

> **项目地址**：GitHub - StockVue
>
> **技术栈**：uni-app · Vue 3 · ECharts 5.5 · SCSS
>
> **作者**：StockVue Team
