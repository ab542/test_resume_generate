# 小程序开发基础与实战 Demo

---

## 一、小程序技术架构

### 双线程模型

```
┌─────────────────────────────────────┐
│            微信客户端 (Native)        │
│  ┌──────────────┐  ┌──────────────┐ │
│  │  渲染层       │  │  逻辑层       │ │
│  │  (WebView)   │  │  (JSCore)    │ │
│  │  WXML/WXSS   │  │  JS 逻辑      │ │
│  │  多个页面实例  │  │  单线程       │ │
│  └──────┬───────┘  └──────┬───────┘ │
│         │     setData      │         │
│         └──────────────────┘         │
│           通过 Native 桥通信          │
└─────────────────────────────────────┘
```

**关键结论**：

| 特点 | 说明 | 测试关注点 |
|------|------|-----------|
| 渲染和逻辑分离 | WebView 渲染页面，JSCore 跑 JS | 两端通信有延迟，不能操作 DOM |
| setData 是唯一的数据通道 | 逻辑层改数据 → Native 中转 → 渲染层更新 | 频率和大小有上限 |
| 单逻辑线程 | 所有页面共享一个 JS 线程 | 一个页面卡死，所有页面都卡 |
| 基础库随微信版本 | 用户微信版本决定可用 API | 低版本基础库要测降级 |

---

## 二、项目结构（必须会）

```
miniprogram-demo/
├── app.js              # 小程序入口，注册全局 App
├── app.json            # 全局配置（页面路由、窗口样式、tabBar）
├── app.wxss            # 全局样式
├── pages/              # 页面目录
│   └── home/
│       ├── home.js     # 页面逻辑（Page({})）
│       ├── home.json   # 页面配置
│       ├── home.wxml   # 页面结构（类 HTML）
│       └── home.wxss   # 页面样式（类 CSS）
├── components/         # 自定义组件
│   └── card/
│       ├── card.js     # 组件逻辑（Component({})）
│       ├── card.json   # 组件配置
│       ├── card.wxml
│       └── card.wxss
├── utils/              # 工具函数
│   └── api.js          # 请求封装
└── project.config.json # 项目配置文件
```

### app.json —— 全局配置

```json
{
  "pages": [
    "pages/home/home",
    "pages/detail/detail",
    "pages/user/user"
  ],
  "window": {
    "navigationBarTitleText": "测试Demo",
    "navigationBarBackgroundColor": "#ffffff"
  },
  "tabBar": {
    "list": [
      { "pagePath": "pages/home/home", "text": "首页" },
      { "pagePath": "pages/user/user", "text": "我的" }
    ]
  }
}
```

> **测试注意**：pages 数组第一项是首页，tabBar 最多 5 个，这些配置错误会导致小程序无法启动。

---

## 三、核心开发知识

### 3.1 页面 —— Page({})

```javascript
// pages/demo/demo.js
Page({
  // 页面数据（相当于 Vue 的 data）
  data: {
    title: '测试页面',
    count: 0,
    list: [],
    isLoading: false
  },

  // 生命周期：页面加载（只执行一次）
  onLoad(options) {
    // options 是页面参数，比如 /pages/demo/demo?id=123
    console.log('页面参数:', options)
    this.fetchList()
  },

  // 生命周期：页面显示（每次切回来都执行）
  onShow() {
    console.log('页面显示了')
  },

  // 生命周期：页面隐藏
  onHide() {
    console.log('页面隐藏了')
  },

  // 生命周期：页面卸载（redirectTo / navigateBack 触发）
  onUnload() {
    // 清理定时器、解绑事件等
    if (this.timer) clearInterval(this.timer)
  },

  // 下拉刷新
  onPullDownRefresh() {
    this.fetchList().then(() => {
      wx.stopPullDownRefresh()
    })
  },

  // 上拉加载更多
  onReachBottom() {
    if (!this.data.isLoading) {
      this.loadMore()
    }
  },

  // 自定义方法
  fetchList() {
    this.setData({ isLoading: true })
    return wx.request({
      url: 'https://api.example.com/list',
      success: (res) => {
        this.setData({ list: res.data, isLoading: false })
      }
    })
  },

  handleTap() {
    this.setData({ count: this.data.count + 1 })
  }
})
```

### 3.2 模板语法 —— WXML

```html
<!-- pages/demo/demo.wxml -->

<!-- 数据绑定 -->
<view class="container">
  <text>{{title}}</text>
  <text>计数: {{count}}</text>
</view>

<!-- 条件渲染 -->
<view wx:if="{{isLoading}}">加载中...</view>
<view wx:elif="{{list.length === 0}}">暂无数据</view>
<view wx:else>
  <!-- 列表渲染 -->
  <view wx:for="{{list}}" wx:key="id" wx:for-item="item" wx:for-index="idx">
    <text>{{idx + 1}}. {{item.name}}</text>
  </view>
</view>

<!-- 事件绑定 -->
<button bindtap="handleTap">点击 +1</button>
<button bindtap="handleReset" data-action="reset">重置</button>

<!-- 双向绑定（基础库 2.9.3+） -->
<input model:value="{{inputValue}}" placeholder="请输入" />
```

> **wx:if vs hidden**：`wx:if` 控制是否渲染（销毁/重建），`hidden` 只是 display:none。频繁切换用 hidden，不常切换用 wx:if。

### 3.3 样式 —— WXSS

```css
/* pages/demo/demo.wxss */

/* 与 CSS 基本一致，但多了 rpx 单位 */
/* 750rpx = 屏幕宽度，自动适配不同屏幕 */

.container {
  padding: 20rpx;
  background: #f5f5f5;
}

.title {
  font-size: 32rpx;
  color: #333;
  /* 多行省略 */
  display: -webkit-box;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 2;
  overflow: hidden;
}

/* 不支持 * 通配符选择器 */
/* 不支持父子组合选择器（如 .a > .b），用后代选择器即可 */
```

> **测试注意**：WXSS 不支持部分 CSS 属性（如 `backdrop-filter` 在低版本），iPhone X 以上需要适配 safe-area。

### 3.4 页面跳转

```javascript
// 保留当前页，跳转到新页（最多 10 层）
wx.navigateTo({
  url: '/pages/detail/detail?id=123&type=product'
})

// 关闭当前页，跳转到新页
wx.redirectTo({
  url: '/pages/login/login'
})

// 关闭所有页面，跳转到 tabBar 页面
wx.switchTab({
  url: '/pages/home/home'
})

// 返回上一页
wx.navigateBack({ delta: 1 })

// 重启小程序到指定页面
wx.reLaunch({
  url: '/pages/home/home'
})
```

> **测试重点**：页面栈最多 10 层，超过 10 层 navigateTo 会失败。redirectTo 会把当前页从栈中移除。

---

## 四、自定义组件 —— Component({})

页面和组件语法几乎一样，区别是组件用 `Component({})`。

```javascript
// components/product-card/product-card.js
Component({
  // 从父页面/父组件接收的属性
  properties: {
    product: {
      type: Object,
      value: {}
    },
    showPrice: {
      type: Boolean,
      value: true
    }
  },

  data: {
    isLiked: false
  },

  // 组件生命周期
  lifetimes: {
    attached() {
      // 组件挂载到页面时
      console.log('组件挂载，收到数据:', this.properties.product)
    },
    detached() {
      // 组件销毁时
      console.log('组件销毁')
    }
  },

  // 监听 properties 变化
  observers: {
    'product.name'(newName) {
      console.log('商品名变化:', newName)
    }
  },

  methods: {
    onTap() {
      // 向父组件传值（自定义事件）
      this.triggerEvent('select', {
        productId: this.properties.product.id
      })
    }
  }
})
```

在页面中使用：

```html
<!-- pages/home/home.wxml -->
<product-card
  wx:for="{{productList}}"
  wx:key="id"
  product="{{item}}"
  showPrice="{{true}}"
  bind:select="onProductSelect"
/>
```

```json
// pages/home/home.json —— 必须注册才能用
{
  "usingComponents": {
    "product-card": "/components/product-card/product-card"
  }
}
```

---

## 五、常用 API 速查

### 5.1 网络请求

```javascript
wx.request({
  url: 'https://api.example.com/user/info',
  method: 'GET',
  data: { userId: 123 },
  header: {
    'content-type': 'application/json',
    'Authorization': 'Bearer ' + wx.getStorageSync('token')
  },
  timeout: 10000,
  success: (res) => {
    if (res.statusCode === 200 && res.data.code === 0) {
      console.log('成功:', res.data)
    }
  },
  fail: (err) => {
    console.error('请求失败:', err)
  },
  complete: () => {
    // 不管成功失败都会执行
  }
})
```

> **测试重点**：域名必须在微信后台配置白名单，且只支持 HTTPS。开发阶段可以在开发者工具勾选"不校验域名"。

### 5.2 本地存储

```javascript
// 同步读写（推荐，不会阻塞）
wx.setStorageSync('key', 'value')
const value = wx.getStorageSync('key')

// 异步读写
wx.setStorage({ key: 'key', data: 'value' })
wx.getStorage({
  key: 'key',
  success: (res) => { console.log(res.data) }
})

// 删除
wx.removeStorageSync('key')
wx.clearStorageSync()  // 清除所有

// 获取当前存储占用
wx.getStorageInfo({
  success: (res) => {
    console.log('已用', res.currentSize, 'KB')
    console.log('上限', res.limitSize, 'KB')  // 上限 10MB
  }
})
```

> **测试重点**：单个 key 上限 1MB，总上限 10MB。超限写入会失败，需要验证异常处理。

### 5.3 系统信息

```javascript
// 同步获取（常用）
const info = wx.getSystemInfoSync()
console.log(info)
// {
//   brand: 'iPhone',          // 设备品牌
//   model: 'iPhone 15',        // 设备型号
//   system: 'iOS 17.0',        // 操作系统版本
//   platform: 'ios',           // 平台 ios/android/devtools
//   SDKVersion: '3.5.7',       // 基础库版本
//   version: '8.0.48',         // 微信版本
//   screenWidth: 393,          // 屏幕宽度 px
//   screenHeight: 852,         // 屏幕高度 px
//   pixelRatio: 3,             // 设备像素比
//   safeArea: {                // 安全区域（刘海屏适配）
//     top: 59, bottom: 34, left: 0, right: 393
//   },
//   benchmarkLevel: 50,        // 设备性能等级（-1 未知, 1~100）
//   fontSizeSetting: 16        // 用户设置的字体大小
// }

// 网络状态
wx.getNetworkType({
  success: (res) => {
    console.log(res.networkType) // wifi / 4g / 3g / 2g / none / unknown
  }
})
```

### 5.4 用户交互

```javascript
// Toast 提示
wx.showToast({ title: '保存成功', icon: 'success', duration: 2000 })

// 模态对话框
wx.showModal({
  title: '确认删除',
  content: '删除后不可恢复',
  success: (res) => {
    if (res.confirm) { /* 执行删除 */ }
  }
})

// Loading
wx.showLoading({ title: '加载中...' })
wx.hideLoading()

// 操作菜单
wx.showActionSheet({
  itemList: ['拍照', '从相册选择'],
  success: (res) => {
    console.log('选择了第', res.tapIndex, '项')
  }
})
```

---

## 六、实战 Demo 1：商品列表 + 详情（核心流程）

### 6.1 列表页

```javascript
// pages/goods-list/goods-list.js
Page({
  data: {
    goodsList: [],
    page: 1,
    hasMore: true,
    isLoading: false
  },

  onLoad() {
    this.loadData()
  },

  loadData() {
    if (!this.data.hasMore || this.data.isLoading) return
    this.setData({ isLoading: true })

    wx.request({
      url: 'https://api.example.com/goods/list',
      data: { page: this.data.page, pageSize: 10 },
      success: (res) => {
        const newList = [...this.data.goodsList, ...res.data.list]
        this.setData({
          goodsList: newList,
          page: this.data.page + 1,
          hasMore: res.data.hasMore,
          isLoading: false
        })
      },
      fail: () => {
        wx.showToast({ title: '加载失败，下拉重试', icon: 'none' })
        this.setData({ isLoading: false })
      }
    })
  },

  onReachBottom() {
    this.loadData()
  },

  onPullDownRefresh() {
    this.setData({ page: 1, hasMore: true, goodsList: [] })
    this.loadData()
    wx.stopPullDownRefresh()
  },

  goDetail(e) {
    const id = e.currentTarget.dataset.id
    wx.navigateTo({ url: `/pages/goods-detail/goods-detail?id=${id}` })
  }
})
```

```html
<!-- pages/goods-list/goods-list.wxml -->
<view class="container">
  <view wx:if="{{goodsList.length === 0 && !isLoading}}" class="empty">
    <text>暂无商品</text>
    <button bindtap="loadData">点击重试</button>
  </view>

  <view wx:for="{{goodsList}}" wx:key="id" class="goods-card"
        bindtap="goDetail" data-id="{{item.id}}">
    <image src="{{item.image}}" mode="aspectFill" class="goods-img" />
    <view class="goods-info">
      <text class="goods-name">{{item.name}}</text>
      <text class="goods-price">￥{{item.price}}</text>
    </view>
  </view>

  <view class="load-more">
    <text wx:if="{{isLoading}}">加载中...</text>
    <text wx:elif="{{!hasMore}}">—— 没有更多了 ——</text>
  </view>
</view>
```

### 6.2 详情页

```javascript
// pages/goods-detail/goods-detail.js
Page({
  data: {
    goods: null,
    isLoading: true,
    currentImageIndex: 0
  },

  onLoad(options) {
    if (!options.id) {
      wx.showToast({ title: '参数错误', icon: 'none' })
      wx.navigateBack()
      return
    }
    this.loadDetail(options.id)
  },

  loadDetail(id) {
    wx.request({
      url: `https://api.example.com/goods/detail/${id}`,
      success: (res) => {
        this.setData({ goods: res.data, isLoading: false })
      },
      fail: () => {
        wx.showToast({ title: '加载失败', icon: 'none' })
        this.setData({ isLoading: false })
      }
    })
  },

  onSwiperChange(e) {
    this.setData({ currentImageIndex: e.detail.current })
  },

  // 分享
  onShareAppMessage() {
    return {
      title: this.data.goods.name,
      path: `/pages/goods-detail/goods-detail?id=${this.data.goods.id}`,
      imageUrl: this.data.goods.image
    }
  }
})
```

```html
<!-- pages/goods-detail/goods-detail.wxml -->
<view wx:if="{{isLoading}}" class="loading">加载中...</view>

<block wx:elif="{{goods}}">
  <!-- 轮播图 -->
  <swiper indicator-dots="{{true}}" autoplay="{{true}}"
          bindchange="onSwiperChange">
    <swiper-item wx:for="{{goods.images}}" wx:key="index">
      <image src="{{item}}" mode="aspectFill" class="banner-img" />
    </swiper-item>
  </swiper>

  <!-- 商品信息 -->
  <view class="info">
    <text class="name">{{goods.name}}</text>
    <text class="price">￥{{goods.price}}</text>
    <text class="desc">{{goods.description}}</text>
  </view>

  <!-- 底部操作栏 -->
  <view class="footer">
    <button class="btn-cart" bindtap="addToCart">加入购物车</button>
    <button class="btn-buy" bindtap="buyNow">立即购买</button>
  </view>
</block>
```

---

## 七、实战 Demo 2：表单 + 校验（测试高频场景）

```javascript
// pages/order-form/order-form.js
Page({
  data: {
    form: {
      name: '',
      phone: '',
      address: '',
      quantity: 1
    },
    errors: {},
    isSubmitting: false
  },

  // 表单输入绑定
  onInput(e) {
    const field = e.currentTarget.dataset.field
    const value = e.detail.value
    this.setData({
      [`form.${field}`]: value,
      [`errors.${field}`]: ''  // 输入时清除错误
    })
  },

  onQuantityChange(e) {
    const type = e.currentTarget.dataset.type
    let qty = this.data.form.quantity
    qty = type === 'plus' ? qty + 1 : Math.max(1, qty - 1)
    this.setData({ 'form.quantity': qty })
  },

  validate() {
    const { name, phone, address } = this.data.form
    const errors = {}

    if (!name.trim()) {
      errors.name = '请输入收货人姓名'
    }
    if (!/^1[3-9]\d{9}$/.test(phone)) {
      errors.phone = '请输入正确的手机号'
    }
    if (!address.trim()) {
      errors.address = '请输入收货地址'
    } else if (address.trim().length < 5) {
      errors.address = '地址不能少于5个字'
    }

    this.setData({ errors })
    return Object.keys(errors).length === 0
  },

  submit() {
    if (!this.validate()) return
    if (this.data.isSubmitting) return // 防重复提交

    this.setData({ isSubmitting: true })

    wx.request({
      url: 'https://api.example.com/order/create',
      method: 'POST',
      data: this.data.form,
      success: (res) => {
        if (res.data.code === 0) {
          wx.showToast({ title: '提交成功' })
          setTimeout(() => { wx.navigateBack() }, 1500)
        } else {
          wx.showToast({ title: res.data.msg || '提交失败', icon: 'none' })
        }
      },
      fail: () => {
        wx.showToast({ title: '网络异常', icon: 'none' })
      },
      complete: () => {
        this.setData({ isSubmitting: false })
      }
    })
  }
})
```

```html
<!-- pages/order-form/order-form.wxml -->
<view class="form">
  <!-- 姓名 -->
  <view class="form-item">
    <text class="label">收货人</text>
    <input placeholder="请输入姓名" value="{{form.name}}"
           bindinput="onInput" data-field="name" />
    <text class="error" wx:if="{{errors.name}}">{{errors.name}}</text>
  </view>

  <!-- 手机号 -->
  <view class="form-item">
    <text class="label">手机号</text>
    <input type="number" maxlength="11" placeholder="请输入手机号"
           value="{{form.phone}}"
           bindinput="onInput" data-field="phone" />
    <text class="error" wx:if="{{errors.phone}}">{{errors.phone}}</text>
  </view>

  <!-- 地址 -->
  <view class="form-item">
    <text class="label">地址</text>
    <textarea placeholder="请输入详细地址" value="{{form.address}}"
              bindinput="onInput" data-field="address" />
    <text class="error" wx:if="{{errors.address}}">{{errors.address}}</text>
  </view>

  <!-- 数量 -->
  <view class="form-item">
    <text class="label">数量</text>
    <view class="quantity-control">
      <button bindtap="onQuantityChange" data-type="minus">-</button>
      <text>{{form.quantity}}</text>
      <button bindtap="onQuantityChange" data-type="plus">+</button>
    </view>
  </view>

  <button class="submit-btn" bindtap="submit"
          disabled="{{isSubmitting}}"
          loading="{{isSubmitting}}">
    {{isSubmitting ? '提交中...' : '提交订单'}}
  </button>
</view>
```

---

## 八、实战 Demo 3：一个迷你购物车（数据管理 + 组件通信）

```javascript
// utils/cart.js —— 全局购物车管理（放在 app.js 或 utils 中管理）
// 这是测试中最常用到的"全局状态管理"模式

const CART_KEY = 'cart_data'

function getCart() {
  return wx.getStorageSync(CART_KEY) || []
}

function saveCart(cart) {
  wx.setStorageSync(CART_KEY, cart)
}

function addToCart(product) {
  const cart = getCart()
  const exist = cart.find(item => item.id === product.id)
  if (exist) {
    exist.quantity += 1
  } else {
    cart.push({ ...product, quantity: 1, checked: true })
  }
  saveCart(cart)
  return cart
}

function removeFromCart(productId) {
  let cart = getCart()
  cart = cart.filter(item => item.id !== productId)
  saveCart(cart)
  return cart
}

function toggleCheck(productId) {
  const cart = getCart()
  const item = cart.find(i => i.id === productId)
  if (item) item.checked = !item.checked
  saveCart(cart)
  return cart
}

function getTotal() {
  const cart = getCart()
  return cart
    .filter(item => item.checked)
    .reduce((sum, item) => sum + item.price * item.quantity, 0)
}

function clearCart() {
  wx.removeStorageSync(CART_KEY)
}

module.exports = {
  getCart, addToCart, removeFromCart,
  toggleCheck, getTotal, clearCart
}
```

购物车页面：

```javascript
// pages/cart/cart.js
const cartUtil = require('../../utils/cart')

Page({
  data: {
    cartList: [],
    totalPrice: 0,
    isEmpty: true
  },

  onShow() {
    this.refreshCart()
  },

  refreshCart() {
    const list = cartUtil.getCart()
    this.setData({
      cartList: list,
      totalPrice: cartUtil.getTotal(),
      isEmpty: list.length === 0
    })
  },

  onCheckChange(e) {
    cartUtil.toggleCheck(e.currentTarget.dataset.id)
    this.refreshCart()
  },

  onRemove(e) {
    wx.showModal({
      title: '确认删除',
      content: '确定要移除这件商品吗？',
      success: (res) => {
        if (res.confirm) {
          cartUtil.removeFromCart(e.currentTarget.dataset.id)
          this.refreshCart()
        }
      }
    })
  }
})
```

---

## 九、测试工程师必须知道的坑

### 9.1 setData 的学问

```javascript
// ❌ 错误：传了整个大对象，实际上只改了 name
this.setData({ userInfo: { ...userInfo, name: 'test' } })

// ✅ 正确：只传要改的字段
this.setData({ 'userInfo.name': 'test' })

// ❌ 错误：没等 setData 完成就读数据
this.setData({ count: 10 })
console.log(this.data.count) // 可能还是旧值

// ✅ 正确：在回调里操作
this.setData({ count: 10 }, () => {
  console.log(this.data.count) // 一定是新值
})
```

### 9.2 异步问题

```javascript
// ❌ 错误：默认认为顺序执行
wx.request({ url: '/api/A' })
wx.request({ url: '/api/B' }) // A 和 B 谁先回来不确定

// ✅ 如果需要 A → B 串行
wx.request({
  url: '/api/A',
  success: (resA) => {
    wx.request({ url: '/api/B', data: resA.data })
  }
})

// ✅ 用 Promise 包装
function requestAsync(options) {
  return new Promise((resolve, reject) => {
    wx.request({ ...options, success: resolve, fail: reject })
  })
}
```

### 9.3 兼容性坑

```javascript
// 日期解析——iOS 不支持 "2024-01-01" 格式
const date = new Date('2024-01-01')  // iOS: Invalid Date
const date = new Date('2024/01/01')  // 都支持

// 或统一用 replace 处理
const date = new Date('2024-01-01'.replace(/-/g, '/'))

// CSS 兼容：flex 布局在低版本 WebView 上需要加 -webkit- 前缀
display: -webkit-box;
display: -webkit-flex;
display: flex;
```

### 9.4 小程序特有的限制

| 限制 | 数值 | 测试场景 |
|------|------|---------|
| 主包大小 | 2MB | 包体积接近上限时各种行为 |
| 总包大小 | 20MB（含分包） | 分包加载失败时的降级 |
| 页面栈深度 | 10层 | 连续跳转 10 个页面后 navigateTo 失败 |
| setData 频率 | < 20次/秒 | 高频 setData 的 warn |
| 并发请求 | 10个 | 同时发起 11 个请求看报错行为 |
| 本地存储 | 10MB | 满了之后的写入行为 |
| 单个 key | 1MB | 超大 value 的存储和读取 |

---

## 十、开发工具使用（必须会）

### 10.1 微信开发者工具

```
核心功能面板：

① 模拟器
  - 机型切换：iPhone 14 / 小米 13 / 自定义
  - 网络模拟：WiFi / 4G / 3G / 离线
  - 页面参数：可以在编译模式里预设 query 参数
  - 基础库版本切换：详情 → 本地设置 → 调试基础库

② 调试器（等同于 Chrome DevTools）
  - Console：看日志、warn、error
  - Sources：断点调试 JS
  - Network：看请求耗时、状态码、返回体
  - Storage：查看 Storage 数据
  - AppData：实时查看当前页面的 data（测试神器）
  - Wxml：查看 WXML 节点树，右键"显示节点信息"

③ 性能工具
  - Audits（体验评分）：一键扫描性能/体验问题
  - Performance：录制运行时性能
  - Memory：录制内存使用情况
```

### 10.2 vConsole（真机调试用）

```javascript
// 在 app.js 中按需开启
// 开发版/体验版可以保留，正式版关掉
if (wx.getAccountInfoSync().miniProgram.envVersion !== 'release') {
  const vConsole = require('./utils/vconsole.min')
  vConsole.init()
}
```

### 10.3 真机调试

```
步骤：
1. 开发者工具点击「真机调试」
2. 手机微信扫码
3. 手机上的操作日志会实时显示在开发者工具的调试面板

用途：
- 本地开发时不能模拟的硬件功能（蓝牙、NFC、摄像头）
- 真机性能表现和模拟器差异大，必须真机验证
- iOS/Android 表现差异排查
```

---

## 十一、快速上手路线

```
Day 1：跑起一个 demo
  - 安装开发者工具 → 新建项目 → 跑起 Hello World
  - 理解 Page({}) 和 setData
  - 做一个按钮 + 计数器的交互

Day 2：网络 + 列表
  - wx.request 请求真实接口（用 httpbin.org 或 mock API）
  - 写一个列表页（上拉加载 + 下拉刷新）
  - 理解 wx:for / wx:if

Day 3：表单 + 存储
  - 写一个表单页面（姓名/手机/地址 + 校验）
  - 用 Storage 存表单草稿（页面退出再回来数据还在）
  - 写一个简单组件，理解父子通信

Day 4：完整流程
  - 商品列表 → 商品详情 → 购物车（用上面的 demo）
  - 用 minium 或手动测试完整跑一遍
  - 重点体会"哪些点需要做兼容性测试"
```
