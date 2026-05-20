# 小程序基础库测试 Demo 实例

---

## 核心思路

测试基础库的 demo 和业务小程序有本质区别——demo 本身是**测试夹具**，每个页面针对一组 API，覆盖正常 + 异常 + 边界 + 版本兼容。

| | 业务小程序 | 测试 Demo |
|------|-----------|----------|
| **目的** | 给用户用 | 给基础库做验证 |
| **关注点** | 业务流程对不对 | API 边界行为、版本兼容、异常恢复 |
| **数据** | 真实业务数据 | 刻意构造的边界/异常数据 |
| **结果判定** | 功能正确 | API 返回值、错误码、性能指标、行为一致性 |
| **页面结构** | 正常 UI | 每个页面是一组测试场景，按钮是测试用例的触发器 |
| **可自动化性** | 次要 | 必须——暴露 `getApp().globalData` 给 minium 采集 |

---

## 实例一：测 `wx.request` 的网络请求行为

**测试目标**：验证基础库在不同条件下 `wx.request` 的超时、重试、并发行为。

```javascript
// pages/request-test/request-test.js
Page({
  data: {
    results: []
  },

  onLoad() {
    this.testCases = [
      { name: '正常GET', url: 'https://httpbin.org/get', method: 'GET', timeout: 10000 },
      { name: '超时测试', url: 'https://httpbin.org/delay/10', method: 'GET', timeout: 3000 },
      { name: '404响应', url: 'https://httpbin.org/status/404', method: 'GET', timeout: 10000 },
      { name: '无效域名', url: 'https://notexist.example.com', method: 'GET', timeout: 10000 },
      {
        name: 'POST大body',
        url: 'https://httpbin.org/post',
        method: 'POST',
        timeout: 10000,
        data: { payload: 'x'.repeat(1024 * 500) }
      },
    ]
  },

  runAll() {
    const results = []
    const promises = this.testCases.map(tc => {
      return new Promise((resolve) => {
        const start = Date.now()
        wx.request({
          url: tc.url,
          method: tc.method,
          data: tc.data,
          timeout: tc.timeout,
          success: (res) => {
            resolve({ name: tc.name, status: res.statusCode, time: Date.now() - start, ok: true })
          },
          fail: (err) => {
            resolve({ name: tc.name, error: err.errMsg, time: Date.now() - start, ok: false })
          }
        })
      })
    })

    Promise.all(promises).then(res => {
      this.setData({ results: res })
      getApp().globalData.testResults = res
    })
  }
})
```

**这个 demo 能测什么**：
- 不同基础库版本下超时时间是否真的生效
- 并发请求上限（基础库限制最多 10 个并发）
- 大 body 传输是否会被截断
- HTTPS 证书校验行为

---

## 实例二：测 `setData` 的数据传输限制

**测试目标**：基础库文档说 setData 单次不能超过 256KB，实际行为是什么？不同版本有差异吗？

```javascript
// pages/setdata-test/setdata-test.js
Page({
  data: {
    singleResult: '',
    freqResults: []
  },

  // 测试1：不同大小的 setData，找到真实的传输上限
  testSingleSize(e) {
    const kb = parseInt(e.currentTarget.dataset.kb)
    const bigStr = 'a'.repeat(kb * 1024)
    const start = Date.now()

    this.setData({ bigData: bigStr }, () => {
      const cost = Date.now() - start
      wx.showToast({ title: `${kb}KB 耗时${cost}ms`, icon: 'none' })
    })
  },

  // 测试2：高频 setData——找到触发 Warning 的阈值
  testHighFreq() {
    const start = Date.now()
    let count = 0
    const timer = setInterval(() => {
      this.setData({ freqCount: count++ })
      if (count >= 60) {
        clearInterval(timer)
        const cost = Date.now() - start
        this.setData({ freqResults: [`60次 setData 总耗时: ${cost}ms`] })
      }
    }, 16)
  },

  // 测试3：跨页面 setData——测基础库的跨页面数据传递
  testCrossPage() {
    this.setData({ crossData: { id: Date.now() } }, () => {
      wx.navigateTo({ url: '/pages/result/result' })
    })
  }
})
```

```html
<!-- pages/setdata-test/setdata-test.wxml -->
<view class="container">
  <view class="section">
    <text>单次传输大小测试</text>
    <button data-kb="64" bindtap="testSingleSize">传输 64KB</button>
    <button data-kb="128" bindtap="testSingleSize">传输 128KB</button>
    <button data-kb="256" bindtap="testSingleSize">传输 256KB（上限）</button>
    <button data-kb="512" bindtap="testSingleSize">传输 512KB（超限）</button>
    <button data-kb="1024" bindtap="testSingleSize">传输 1MB（超限）</button>
  </view>

  <view class="section">
    <text>高频调用测试（60次/s）</text>
    <button bindtap="testHighFreq">开始高频 setData</button>
    <view wx:for="{{freqResults}}" wx:key="index">{{item}}</view>
  </view>
</view>
```

**这个 demo 能测什么**：
- 基础库对 setData 大小的实际限制（256KB 是建议还是硬限制？）
- 超限后的表现：报错？截断？还是静默失败？
- 不同基础库版本的 setData 性能差异
- 高频调用的 warn 机制是否正常触发

---

## 实例三：测组件在不同基础库版本的表现

**测试目标**：有些组件行为会随基础库版本变化，比如 `<swiper>` 的 circular 属性在 `2.0.0` 才支持，`<scroll-view>` 的 enhanced 属性在 `2.12.0` 才加入。demo 的目标是自动检测当前基础库版本下哪些属性不可用。

```javascript
// pages/component-test/component-test.js
Page({
  data: {
    sdkVersion: '',
    unsupportedList: [],
    componentMatrix: [
      { name: 'swiper-circular',           api: 'swiper',      prop: 'circular',                 minSdk: '2.0.0' },
      { name: 'scroll-view-enhanced',      api: 'scroll-view', prop: 'enhanced',                  minSdk: '2.12.0' },
      { name: 'video-picture-in-picture',  api: 'video',       prop: 'picture-in-picture-mode',   minSdk: '2.20.0' },
      { name: 'camera-frame',              api: 'camera',      prop: 'frame-size',                minSdk: '2.7.0' },
      { name: 'input-auto-fill',           api: 'input',       prop: 'enableAutoFill',            minSdk: '2.21.0' },
    ]
  },

  onLoad() {
    const sdkVersion = wx.getSystemInfoSync().SDKVersion
    this.setData({ sdkVersion })

    const unsupported = this.data.componentMatrix.filter(item => {
      return this.compareVersion(sdkVersion, item.minSdk) < 0
    })
    this.setData({ unsupportedList: unsupported })
  },

  compareVersion(v1, v2) {
    const a = v1.split('.').map(Number)
    const b = v2.split('.').map(Number)
    for (let i = 0; i < Math.max(a.length, b.length); i++) {
      if ((a[i] || 0) > (b[i] || 0)) return 1
      if ((a[i] || 0) < (b[i] || 0)) return -1
    }
    return 0
  }
})
```

```html
<!-- pages/component-test/component-test.wxml -->
<view class="container">
  <view class="banner">
    当前基础库版本：<text class="highlight">{{sdkVersion}}</text>
  </view>

  <!-- 实际渲染这些组件，看降级后的实际表现 -->
  <swiper circular="{{true}}" indicator-dots="{{true}}">
    <swiper-item wx:for="{{3}}" wx:key="index">
      <view class="swiper-item">第{{index + 1}}页</view>
    </swiper-item>
  </swiper>

  <scroll-view scroll-y enhanced="{{true}}" show-scrollbar="{{false}}">
    <view wx:for="{{50}}" wx:key="index">第{{index}}行</view>
  </scroll-view>

  <!-- 不支持的属性列表（同时暴露给自动化脚本做断言） -->
  <view class="unsupported">
    <text>以下属性当前版本不支持：</text>
    <view wx:for="{{unsupportedList}}" wx:key="name">
      {{item.api}}.{{item.prop}}（需要基础库 ≥ {{item.minSdk}}）
    </view>
  </view>
</view>
```

**这个 demo 能测什么**：
- 在低版本基础库上，不支持的高级属性是否静默忽略（不崩溃）
- 版本比较逻辑是否准确
- 不同 iOS/Android 上相同基础库版本的组件表现是否一致

---

## 实例四：完整的自动化回归套件（minium）

把上面三个 demo 串起来，用 minium 写自动化用例，每次基础库升级跑一遍全量回归：

```python
# test_basic_library.py
import minium


class TestBasicLibrary(minium.MiniTest):

    # ========== wx.request 相关 ==========

    def test_request_concurrent_limit(self):
        """验证并发请求上限（基础库限制 10 个）"""
        self.app.navigate_to('/pages/request-test/request-test')
        page = self.app.get_current_page()
        page.get_element('.run-all-btn').tap()
        page.wait_for(15)

        results = page.evaluate('getApp().globalData.testResults')
        fail_count = sum(1 for r in results if not r['ok'])
        assert fail_count == 0, f"有 {fail_count} 个请求失败: {results}"

    def test_request_timeout(self):
        """验证超时行为——超时请求应失败且错误信息包含 timeout"""
        self.app.navigate_to('/pages/request-test/request-test')
        page = self.app.get_current_page()
        page.get_element('.run-all-btn').tap()
        page.wait_for(15)

        results = page.evaluate('getApp().globalData.testResults')
        timeout_result = [r for r in results if r['name'] == '超时测试'][0]

        assert not timeout_result['ok'], "超时请求应该失败"
        assert 'timeout' in timeout_result['error'].lower(), \
            f"错误信息应包含 timeout: {timeout_result}"
        assert timeout_result['time'] < 5000, \
            f"实际超时时间不应偏离设置值太多: {timeout_result['time']}ms"

    # ========== setData 相关 ==========

    def test_setdata_max_size(self):
        """验证 setData 大小限制——超限应有 warn"""
        self.app.navigate_to('/pages/setdata-test/setdata-test')
        page = self.app.get_current_page()

        # 256KB 应在正常范围
        page.get_element('[data-kb="256"]').tap()
        page.wait_for(3)

        # 1MB 预期触发 warn
        page.get_element('[data-kb="1024"]').tap()
        page.wait_for(3)

        logs = self.app.get_logs(level='WARN')
        size_warnings = [l for l in logs if 'setData' in l.get('msg', '')]
        assert len(size_warnings) > 0, "超限 setData 应该触发 Warning"

    # ========== 组件兼容性 ==========

    def test_component_version_matrix(self):
        """验证组件属性版本检测——高版本基础库不应有 unsupported 属性"""
        self.app.navigate_to('/pages/component-test/component-test')
        page = self.app.get_current_page()

        sdk_version = page.evaluate('wx.getSystemInfoSync().SDKVersion')
        unsupported = page.evaluate('getCurrentPages()[0].data.unsupportedList')

        if self.compare_sdk(sdk_version, '2.21.0') >= 0:
            assert len(unsupported) == 0, \
                f"SDK {sdk_version} 应支持所有属性，但标记了不支持: {unsupported}"

    @staticmethod
    def compare_sdk(v1, v2):
        parts1 = [int(x) for x in v1.split('.')]
        parts2 = [int(x) for x in v2.split('.')]
        return (parts1 > parts2) - (parts1 < parts2)
```

---

## Demo 项目结构建议

```
basic-lib-test-demo/
├── app.js                          # globalData 暴露测试结果
├── app.json                        # 注册所有测试页面
├── pages/
│   ├── index/                      # 首页仪表盘——列出所有测试套件 + pass/fail
│   │   ├── index.js
│   │   ├── index.wxml
│   │   └── index.wxss
│   ├── request-test/               # wx.request 行为测试
│   │   ├── request-test.js
│   │   └── request-test.wxml
│   ├── setdata-test/               # setData 传输限制测试
│   │   ├── setdata-test.js
│   │   └── setdata-test.wxml
│   ├── component-test/             # 组件版本兼容性测试
│   │   ├── component-test.js
│   │   └── component-test.wxml
│   ├── navigation-test/            # 页面跳转 + 参数传递测试
│   ├── storage-test/               # Storage API 边界测试（容量上限、并发读写）
│   └── lifecycle-test/             # App/Page 生命周期行为测试
├── utils/
│   └── sdk-version.js              # 版本比较工具函数
└── tests/                          # minium 自动化用例
    ├── conftest.py
    ├── test_request.py
    ├── test_setdata.py
    └── test_component.py
```

首页仪表盘示例：

```html
<!-- pages/index/index.wxml -->
<view class="dashboard">
  <view class="header">
    <text>基础库版本: {{sdkVersion}}</text>
    <text>微信版本: {{wechatVersion}}</text>
    <text>系统: {{platform}} {{system}}</text>
  </view>

  <view class="suite {{requestStatus}}">
    <text>wx.request 测试套件</text>
    <text class="badge">{{requestStatus}}</text>
    <button bindtap="runRequestSuite">运行</button>
  </view>

  <view class="suite {{setdataStatus}}">
    <text>setData 测试套件</text>
    <text class="badge">{{setdataStatus}}</text>
    <button bindtap="runSetdataSuite">运行</button>
  </view>

  <view class="suite {{componentStatus}}">
    <text>组件兼容性测试套件</text>
    <text class="badge">{{componentStatus}}</text>
    <button bindtap="runComponentSuite">运行</button>
  </view>
</view>
```

这样设计有几个好处：
- **手工测试时**：打开仪表盘一目了然，逐套件点"运行"即可
- **自动化时**：minium 直接 navigateTo + evaluate 拿结果，不需要解析页面 UI
- **CI 集成时**：每次基础库升级跑一遍全量套件，版本间的行为差异自动暴露
