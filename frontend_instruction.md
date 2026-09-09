我觉得 MEGA — Make Eating Great Again 的前端不应该做成另一个 DoorDash。

核心应该是：

传统外卖 App：Restaurant → Menu → Cart → Checkout
MEGA：Intent → Agent works → Compare → Human approves → Done

也就是说，尽可能消灭 UI，而不是增加 UI。用户主要在和 Agent 交互，传统页面只在需要查看、比较、确认的时候出现。

1. 首页 /

首页应该极简，甚至第一屏只有一句：

What do you wanna eat?

下面一个很大的 Agent 输入框：

Get me something spicy and filling under $25.

支持自然语言，例如：

“想吃中餐，两个人，$40以内，不要太辣。”
“Find me the cheapest Chick-fil-A combo delivered within 30 mins.”
“I don’t know. Just get me something good.”
“昨天那个拉面再来一份，但是看看哪里更便宜。”

输入框附近只需要几个能力入口：

Location · Budget · People · Preferences

以及语音、上传图片之类。

下面可以有非常少量的快捷 Prompt：

I'm starving　Under $20　Healthy　Surprise me

不要一上来给用户几十家餐厅。

⸻

2. Agent 工作页面 /hunt

这是整个产品最重要的 UI。

用户发出：

“Get me a good burger under $20.”

MEGA 开始自己操作。

界面不是传统 loading，而应该让用户看到 Agent 在干什么：

MEGA
Finding your food...
✓ Understood what you want
✓ Found 38 nearby options
● Comparing prices across 4 sources
○ Checking promotions
○ Estimating final checkout price
○ Picking the best options

同时可以偶尔出现：

Uber Eats
$17.82

DoorDash
$21.40

Restaurant direct
$16.95

但不要把所有 Agent reasoning 暴露出来，只显示可验证的 action/evidence。

这会让产品有一种：

“有个东西正在替我跑腿”

的感觉。

⸻

3. Agent Result：不要给 50 个结果

这是 MEGA 和普通搜索最大的区别。

Agent 最终应该非常有主见：

I’d get this.

🍔 Double Cheeseburger Combo

$17.82 delivered · 24 min

Best overall. $4.13 cheaper than DoorDash and arrives 8 minutes sooner.

[ Get this ]

然后下面最多两个：

CHEAPEST

$14.60 · 37 min

FASTEST

$19.20 · 18 min

也就是：

MEGA Pick
Cheapest
Fastest

最多三个。

用户不是来 MEGA 看搜索结果的，而是来减少决策的。

⸻

4. Compare Sheet

但用户肯定会问：

为什么你说这个最便宜？

点击：

Why this?

弹出 Bottom Sheet：

	Uber Eats	DoorDash	Direct
Food	$13.99	$13.99	$12.99
Delivery	$0	$2.99	$0
Service	$2.84	$3.42	$1.20
Tax	$1.32	$1.41	$1.26
Promo	-$2.00	—	—
Total	$16.15	$21.81	$15.45

然后：

MEGA recommends Direct — save $6.36.

这张表非常重要，因为**“跨平台真实 checkout price”就是你的核心产品价值之一**。

⸻

5. Human-in-the-loop Approval

这是整个 Agent UX 最关键的一步。

Agent 绝对不能悄悄买。

准备下单的时候出现一个非常明确的 checkpoint：

Ready to order.

Din Tai Fung

Pork Xiao Long Bao × 1
Pork Chop Fried Rice × 1

Food　　　　　$28.40
Fees　　　　　$3.20
Tax　　　　　 $2.91
Tip　　　　　 $4.00

$38.51

Deliver to
Home · 123••••

Payment
Apple Pay / Visa •••• 4242

Estimated arrival
12:35–12:50 PM

然后两个最重要的操作：

[ Approve $38.51 ]

Change something

这就是 Human-in-the-loop 的边界。

⸻

6. “Change something” 不应该返回菜单

这点我觉得很重要。

用户点 Change 后，不应该突然把他扔进传统 checkout form。

而是继续跟 Agent 说：

What should I change?

用户：

“不要饮料。”

Agent：

Removed Coke.

New total: $35.21

Ready?

或者：

“tip改成$2。”

“换成牛肉。”

“这个配送费太贵了，找另一个。”

始终保持 Agent interaction。

⸻

7. Order Tracking

下单以后：

Food secured.

这个 wording 我觉得很符合 MEGA 😂

下面：

ORDERED → PREPARING → PICKED UP → HERE

预计：

12:42 PM

然后显示非常简洁的地图。

Agent 还可以主动处理：

Your order is running 12 minutes late.

Want me to check what’s going on?

[ Handle it ]

未来甚至：

Driver can’t find entrance.

MEGA：

I can send your saved delivery instructions.

[ Approve message ]

Human-in-the-loop 再次出现。

⸻

8. History 不应该只是 Order History

这里可以做成：

Your Taste

MEGA 慢慢学习：

You seem to like

Japanese Chinese Burgers High protein

You usually avoid

Raw food Very spicy Expensive delivery

Typical budget

Lunch $15–22
Dinner $20–35

Your behavior

You usually choose faster delivery when the difference is <$3.

这些应该允许用户编辑。

然后历史订单：

Yesterday
Ramen · $18.42

[ Do that again ]

这个按钮特别重要。

甚至首页直接：

Same as yesterday?

⸻

9. MEGA Memory

可以单独有：

MEGA knows

Food
✓ No raw food
✓ Steak well done
✓ Mild–medium spice
Ordering
✓ Prefer lowest total price
✓ Max delivery fee: $4
✓ Default tip: 15%
Delivery
✓ Home
✓ Leave at door
Platforms
✓ Uber Eats
✓ DoorDash
✓ Grubhub

用户可以自然语言修改：

“以后配送超过 $5 就别推荐了。”

MEGA：

Got it.
Delivery fee limit → $5

这样 Agent 才会有越用越懂你的感觉。

⸻

10. Connected Services

Settings 里面需要一个非常重要的页面：

MEGA can order from

Uber Eats　Connected ✓

DoorDash　Connected ✓

Grubhub　Connect

Restaurant websites　Enabled ✓

Google Maps　Enabled ✓

Payment：

Apple Pay
Google Pay
Credit Card

但这些应该藏在 Settings。

正常用户根本不应该关心 MEGA 后台用了哪个平台。

⸻

11. 一个很重要的功能：Agent Permission

既然你要 Human-in-the-loop，我建议直接把权限设计成产品 feature。

Agent Autonomy

Ask every time

MEGA can search and compare freely.
Always asks before spending money.

Auto-order under $20

MEGA may place orders ≤ $20 from previously ordered restaurants.

Always ask for new restaurants

这样以后产品可以逐渐从：

Copilot → Agent

而不是一开始就让用户把购买权全部交出去。

⸻

12. 首页其实可以更疯狂一点

MEGA 的品牌不是普通 SaaS，所以首页不需要：

Welcome back, John 👋
What would you like to order today?

太模板化了。

我反而会随机出现这种：

Hungry?

Fix that.

或者：

Feed me.

或者：

What’s the damage?

或者最符合品牌的：

Eating should be easier.

Ask MEGA

甚至第一次打开：

MEGA

Make Eating Great Again.

Tell me what you want.
I’ll figure out where to get it.

I want...

这样品牌记忆会非常强。

⸻

整个 Information Architecture 我会压得非常小

实际上甚至只需要 4 个一级入口：

                 MEGA
                   │
        ┌──────────┼──────────┐
        │          │          │
       Ask       Orders       You
        │
        ▼
     Intent
        │
        ▼
   Agent Hunt
        │
        ▼
  Recommendation
        │
        ├──── Why? → Comparison
        │
        ▼
     Approval   ← HUMAN
        │
        ▼
      Order
        │
        ▼
     Tracking

底部 Navigation：

MEGA　　Orders　　You

甚至连 Home 都不用叫 Home，MEGA 本身就是 Home。

⸻

我最想做的一个交互

首页：

MEGA
Make Eating Great Again.

What do you want?

用户：

idk just get me some fuckin food

Agent 工作十几秒。

然后整个页面突然收敛成一张卡：

Say less.

🍜 Tonkotsu Ramen

$18.72 · 26 min

I checked 31 options across 4 sources.
This is the one.

[ Get it — $18.72 ]

nah

用户点 nah：

Fair.

How about Korean?

这种略带态度但不妨碍任务完成的 Agent personality，会比再做一个漂亮的外卖 UI 有辨识度得多。

而且这套结构非常适合你后面用 Next.js + React + SSE/WebSocket agent event stream 实现：前端本质上是在渲染 Agent 的 searching → comparing → recommendation → approval_required → ordering → tracking 状态机。