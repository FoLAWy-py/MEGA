你这个判断基本对：如果目标真的是“美团 / 淘宝闪购里的消费者端选购 + 比价 + 下单”，直接照搬你之前浏览器 agent 那套 Patchright + DOM 会很难，因为这两个场景的核心体验都在移动端 App，网页并不是主要交互面。

但我不建议第一版就做成：

1 user = 1 云 VM = 1 Android Studio Emulator = 1 agent

这会非常重，而且模拟器环境也更容易触发设备环境、登录、风控方面的问题。更好的架构是：Android runtime 是执行器，不是“整台云电脑”。

⸻

我会优先考虑这套

核心：

Android Emulator / Android VM + ADB + UiAutomator2 / Accessibility tree + screenshot fallback

而不是：

Android Emulator + Vision Model 全程看屏幕

Android 本身其实也有类似“DOM”的东西，只不过不叫 DOM。

Android Accessibility 会把 UI 暴露成一棵 AccessibilityNodeInfo tree，包括文本、节点、状态以及部分可执行 action。Android 官方文档明确把屏幕内容暴露成 accessibility node tree；Appium 的 UiAutomator2 driver 也是专门做 Android native / hybrid / web App 自动化的。 

所以你可以得到类似：

FrameLayout
 ├── TextView
 │    text="麦当劳"
 ├── TextView
 │    text="月售 3000+"
 ├── TextView
 │    text="配送费 ¥3"
 └── Button
      text="去结算"

然后 Agent 操作：

find(text="麦当劳")
click(node)
find(text="去结算")
click(node)

这和：

page.getByText("麦当劳").click()

其实思维模型非常接近。

只是浏览器是：

HTML → DOM → selector

Android 是：

View / Accessibility → UI tree → UiSelector

⸻

所以 Vision 不应该是主路径

我会做：

Layer 1：结构化 UI Tree

优先使用：

UiAutomator2 / AccessibilityNodeInfo

提取：

{
  "text": "¥18.9",
  "resourceId": "...",
  "class": "android.widget.TextView",
  "bounds": [80, 440, 220, 490],
  "clickable": false
}

Agent 绝大多数时候都只看这种 compact representation。

例如你完全可以转换成你自己的 agent DOM：

<restaurant id="r12">
  <name>麦当劳</name>
  <rating>4.8</rating>
  <delivery_fee>3</delivery_fee>
  <item id="i44">
    <name>麦辣鸡腿堡套餐</name>
    <price>18.9</price>
    <action id="a19">Add</action>
  </item>
</restaurant>

然后给 LLM 的不是整棵 XML，而是压缩之后的：

[12] 麦当劳
  [44] 麦辣鸡腿堡套餐 ¥18.9
       action=[19:Add]
[13] 肯德基
  [51] 香辣鸡腿堡套餐 ¥21.5
       action=[21:Add]

成本会非常低。

⸻

Layer 2：Screenshot + 传统 CV

有时候 App 会出现：

* Canvas
* 自绘组件
* WebView
* 图片文字
* Accessibility tree 缺节点
* 动画弹层

这时候不要立刻 Vision LLM。

先做：

Screenshot + coordinates + OCR / image matching

例如：

adb exec-out screencap

然后检测你已知的 UI。

如果：

tree 里找不到“提交订单”

但 screenshot 上明显存在按钮，就进入 fallback。

⸻

Layer 3：Vision Model

只有：

Accessibility tree 不够
AND OCR/CV 不确定

才调用 Vision。

所以实际运行可能是：

90% UI tree
 8% screenshot/CV
 2% vision model

而不是每一步：

screenshot
→ GPT vision
→ tap
→ screenshot
→ GPT vision
→ tap

后者成本和 latency 都会很难受。

⸻

更重要的问题：不要把 LLM 放到每个 click 上

你之前 Browser Agent 很容易变成：

Observe DOM
↓
LLM
↓
click
↓
Observe DOM
↓
LLM
↓
click

MEGA 更适合：

LLM Planner
     ↓
   Skills
     ↓
Android Executor

例如你定义 deterministic skills：

search_food(query)
open_restaurant(id)
get_menu()
add_item(item_id)
get_cart()
apply_coupon()
get_checkout_quote()
submit_order()

Agent 只决定：

我要调用哪个 skill。

而 skill 内部：

search_food("麦辣鸡腿堡")

自己执行：

tap search
input text
tap search button
wait
parse tree
return structured results

中间可能 5～15 次 Android 操作，但不需要 LLM 看每一步。

这会让成本直接降一个数量级。

⸻

甚至比 UiAutomator2 更进一步：建立自己的 “Mobile DOM”

我其实非常推荐这个。

你之前已经有 browser evidence / exact DOM 的思路，那么 MEGA 可以抽象一个统一的：

Environment Adapter

浏览器：

Patchright
     ↓
DOM Adapter
     ↓
MEGA UI Schema

Android：

UiAutomator2
     ↓
Accessibility Adapter
     ↓
MEGA UI Schema

最终 Agent 根本不知道底下是 Browser 还是 Android。

例如：

{
  "screen": "restaurant_list",
  "elements": [
    {
      "ref": "e17",
      "role": "restaurant",
      "name": "麦当劳",
      "metadata": {
        "rating": 4.8,
        "eta": "30分钟",
        "delivery_fee": 3
      },
      "actions": ["open"]
    }
  ]
}

LLM：

{
  "action": "open",
  "ref": "e17"
}

Android executor 再把：

e17

映射回：

resourceId / xpath / bounds

这样你的 architecture 会非常干净。

⸻

那需要“一用户一个 Android”吗？

从账户隔离角度，我认为最终大概率是需要持久 Android identity 的。

但不是：

一个人永远占着一台 EC2。

应该是：

一个 user → 一个 persistent Android profile/device state

包含：

/data/data/...
cookies/session
device identifiers
app storage
login state
location/preferences

而 compute 可以动态 attach。

概念上：

User A
 └── Android Profile A
      ├── 美团 login
      ├── 淘宝 login
      ├── app state
      └── device identity
User B
 └── Android Profile B

用户需要 MEGA：

Profile A
   ↓
Android Worker
   ↓
boot/resume
   ↓
execute
   ↓
snapshot
   ↓
suspend

而不是 24/7 开着。

⸻

甚至可以直接运行 Android VM，而不是 Android Studio Emulator

生产环境我不会用 Android Studio 那套 UI emulator。

开发阶段：

Android Studio AVD

没问题。

生产我会看这种 abstraction：

Linux host
│
├── Android VM 001
├── Android VM 002
├── Android VM 003
└── Android VM ...

每个实例：

Android
ADB
UiAutomator2
MEGA executor

Android Appium 官方本身就支持 emulator 和真实 Android device。 

⸻

但是这里有一个很大的现实问题：App 风控

这是整个项目里我觉得比 Vision 成本更值得担心的事情。

美团 / 淘宝这类高价值消费平台很可能关注：

emulator
root
device fingerprint
IP
location
abnormal click patterns
login geography
mass account behavior

所以你如果：

100 用户 → 同一个 datacenter IP → 100 Android emulator

商业化以后很可能比 selector 稳定性更早遇到问题。

这也是为什么我建议早期：

不要设计成“爬虫农场”。

而是：

用户主动连接自己的 account
每个 session 明确由用户发起
下单前 Human approval
不大量扫描平台
尽可能利用官方授权/API

⸻

API 路线值得先调查

淘宝这边确实存在正式开放平台，并且当前文档中已经能看到淘宝闪购相关 API 类目，包括交易、营销、库存以及闪购联盟相关接口。 

但我现在看到的大量接口明显偏：

商户 / ISV / 联盟 / 库存 / 营销

而不是：

“以消费者身份搜索附近外卖 → 加购物车 → 使用用户优惠券 → 下订单”

例如闪购交易 API 的说明就是查询卖家交易数据；闪购库存 API 也是门店库存管理。 

所以暂时不能假设：

Alibaba Open API = Consumer ordering API

这是两个完全不同的问题。

但如果你以后真的把项目商业化，我会优先尝试与平台建立官方合作，因为 consumer ordering API 一旦开放给你，你可以直接砍掉整个 Android execution 层。

⸻

我建议 MEGA 的技术结构

我会设计成：

                      MEGA Web / Mobile
                             │
                             ▼
                       Agent Service
                             │
                   ┌─────────┴─────────┐
                   │                   │
              Intent Planner       Memory
                   │
                   ▼
                Food Agent
                   │
          ┌────────┴────────┐
          │                 │
       Search             Order
          │                 │
          ▼                 ▼
       Skills Layer / Tool Registry
          │
          ├── search_meituan()
          ├── search_taobao()
          ├── menu()
          ├── quote()
          ├── coupons()
          ├── cart()
          └── checkout()
                   │
                   ▼
             Platform Adapter
          ┌────────┼──────────┐
          │        │          │
       API     Android      Web
                 │
          ┌──────┴───────┐
          │              │
       UI Tree       Screenshot
          │              │
          │           CV/OCR
          │              │
          └──────┬───────┘
                 │ fallback
                 ▼
              Vision

重点是：

API > accessibility tree > CV > Vision

按这个优先级走。

⸻

比价还有一个非常重要的设计细节

不要比较商品列表页价格。

因为美团/淘宝闪购：

商品价
+ 配送费
+ 打包费
+ 服务费
- 店铺券
- 平台券
- 会员优惠
- 红包
- 满减

最终：

checkout price

才是真实价格。

所以你的 quote() tool 应该是一级核心 primitive：

{
  "platform": "meituan",
  "restaurant": "麦当劳",
  "items": [...],
  "subtotal": 32.0,
  "packaging": 2,
  "delivery": 3,
  "discount": -8,
  "coupon": -5,
  "final": 24.0,
  "eta": 31
}

MEGA 真正的 moat 很可能不是：

“AI 帮你搜饭。”

而是：

同一 purchase intent，自动构建各平台 basket，进入 checkout，算出 personalized final price，再比较。

因为优惠券是用户特定的。

这就是为什么你确实需要 user-authenticated execution environment。

⸻

我目前最推荐的 MVP

我不会一开始做“一用户一云电脑”。

第一版直接：

一台本地 Android 真机 / Emulator + 美团 + 淘宝 → ADB + UiAutomator2 → 自己定义 Mobile DOM → deterministic skills → LLM planner → checkout 前 HITL

先验证一个最关键的问题：

Accessibility tree 到底能覆盖美团和淘宝闪购多少核心 UI。

这件事一旦验证成功，你的技术风险会一下下降很多。Android 本身确实允许通过 AccessibilityNodeInfo 暴露 window content tree，因此这个方向理论基础是成立的；但具体 App 有多少自绘/WebView/隐藏 accessibility 信息，需要实际跑 App 才能知道。 

如果核心搜索、商品、购物车、结算页能达到 80%+ structured extraction，我就会坚持 Android UI-tree 路线，而不会选择纯 Vision Computer Use。

你这个项目完全可以继续沿用你之前 Patchright + exact evidence + HUMAN_ACTION_REQUIRED 的哲学，只是把底层从 Browser DOM 换成 Android Accessibility DOM。