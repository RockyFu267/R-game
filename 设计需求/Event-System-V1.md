好，Event System V1 我建议直接把它定义成：

> **Event = 世界在满足某些条件时，对当前游戏状态产生的一次结构化反应。**

它和 Action 正好是一对：

```text
Action = 玩家主动做什么
Event  = 世界因此或因时间/状态自行发生什么
```

而且 Event 不应该只是“播一段剧情”，它应该能真正改变世界。

### 1. Event 的最小结构

我建议第一版就用这一套：

```yaml
event:
  id: event_power_outage

  name: 停电

  trigger:
    type: time_reached

  conditions:
    - time >= 23:40
    - power_outage_triggered == false

  priority: 50

  repeat:
    mode: once_per_loop

  effects:
    - type: set_flag
      key: power_on
      value: false

    - type: set_flag
      key: power_outage_triggered
      value: true

  presentation:
    text_block: event_power_outage_01

  aftermath:
    resolve_scene: true
```

核心还是几个问题：

**什么时候触发、能不能触发、优先级多少、是否重复、改变什么、玩家看到什么、触发后怎么办。**

---

## 2. Trigger：Event 为什么发生

我建议 V1 支持 6 种触发来源就够了。

```text
ACTION
玩家执行某个 Action 后

TIME
游戏时间达到某个条件

ENTER_LOCATION
进入某地点

LEAVE_LOCATION
离开某地点

STATE_CHANGE
某个状态发生变化

STORY
进入某 Story Node / 阶段
```

再加一个：

```text
LOOP
循环开始 / 结束 / 第N轮
```

这个对我们很重要。

例如：

```yaml
trigger:
  type: action
  action_id: investigate_radio
```

或者：

```yaml
trigger:
  type: enter_location
  location: warehouse
```

---

# 3. Trigger 和 Conditions 必须分开

这一点很重要。

比如：

> “进入仓库时，老陈可能突然出现。”

Trigger 是：

```text
ENTER warehouse
```

但 Conditions 是：

```text
chen_alive == true
chen.location != warehouse
time >= 22:00
event_not_triggered
```

也就是说：

> Trigger 负责说“现在要不要检查一下”。

> Conditions 负责说“检查以后到底符不符合发生资格”。

这样系统会清晰很多。

---

# 4. Event 应该分三类

我建议从玩法角度分：

### Immediate Event

立即打断当前流程。

例如：

> 玩家刚打开门，里面突然有人冲出来。

流程：

```text
Action
↓
Event触发
↓
立刻展示
↓
处理后果
↓
重新解析场景
```

---

### Passive Event

不打断玩家，只悄悄改变状态。

例如：

```text
NPC suspicion +5
某扇门自动关闭
某角色移动到其他地点
```

玩家不一定立刻知道。

这是很适合恐怖游戏的。

---

### Scheduled Event

已经被安排，但未来才发生。

例如玩家打开某台设备后：

```text
schedule:
  +15 min
  trigger event_radio_reply
```

于是 15 分钟以后，世界产生反应。

这个非常重要，因为它让玩家行为有“延迟后果”。

---

# 5. Event Queue

我强烈建议做一个事件队列。

因为一次 Action 可能同时引发很多东西。

例如玩家：

> 【打开无线电】

结果同时满足：

```text
Event A：无线电启动
Event B：老陈注意到玩家
Event C：五分钟后收到异常信号
Event D：Story Node条件满足
```

不能乱序。

所以维护：

```text
Event Queue

priority 100 关键剧情
priority 80  强制危险事件
priority 50  普通场景事件
priority 30  NPC反应
priority 10  氛围事件
```

然后按优先级处理。

---

# 6. 但是不要允许无限连锁

这是事件系统特别容易炸掉的地方。

例如：

```text
Event A
→ 修改 Flag X

Flag X变化
→ Event B

Event B
→ 修改 Flag Y

Flag Y
→ Event C

Event C
→ 又触发 Event A
```

直接死循环。

所以每一轮事件处理都必须有：

```text
Event Resolution Cycle ID
Maximum Chain Depth
Already Processed Events
```

例如：

```text
本轮最多连续处理 20 个 Event
同一 once Event 禁止重复入队
发现循环依赖时终止并写 Error Log
```

开发阶段一定救命。

---

# 7. Repeat Policy

每个 Event 必须明确能发生几次。

我建议：

```text
ONCE_GAME
整个存档只发生一次

ONCE_LOOP
每轮最多一次

ONCE_SCENE
每次进入场景最多一次

REPEATABLE
可以重复

COOLDOWN
间隔一定游戏时间后可再次触发
```

例如普通环境声：

```text
REPEATABLE
cooldown: 30min
```

而关键人物死亡：

```text
ONCE_LOOP
```

---

# 8. Event Effect

和 Action 一样，Event 不应该自己写代码修改游戏状态。

全部使用统一 Effect。

所以：

```text
Action Effects
Event Effects
```

实际上共用同一个 **Effect Engine**。

例如：

```yaml
effects:
  - type: move_npc
    npc: chen
    location: warehouse

  - type: set_flag
    key: door_locked
    value: true

  - type: add_variable
    key: chen_suspicion
    value: 10

  - type: unlock_clue
    clue: clue_023

  - type: schedule_event
    event: event_024
    delay: 10
```

这样系统非常统一。

---

# 9. Event Presentation 和 Event Logic 分离

这点和 Scene 一样。

不要：

```text
Event_17 =
显示这一大段文字 + 修改这些东西
```

最好：

```text
Event Logic
↓
Presentation Block
```

例如：

```yaml
event:
  id: power_off

  effects:
    power_on = false

  presentation:
    block: blackout_01
```

以后同一个 Event 甚至可以根据条件用不同表达：

```text
如果玩家在宿舍
→ blackout_dormitory

如果玩家在走廊
→ blackout_corridor

如果玩家正在调查无线电
→ blackout_radio
```

**发生的是同一个 Event，叙事表现不同。**

这个设计很重要。

---

# 10. 隐藏 Event

我建议恐怖游戏里大量使用玩家完全不知道发生过的 Event。

例如：

```text
23:13

Hidden Event:
NPC Chen moves:
guard_room → warehouse
```

玩家什么都看不到。

但 23:20 到值班室：

> 老陈不在。

于是玩家自己产生：

> 他去哪了？

而不是系统弹：

> 【老陈已经移动到仓库】

这就是我们想要的信息差。

所以 Event 应该有：

```yaml
visibility:
  hidden: true
```

或者：

```text
VISIBLE
HIDDEN
CONDITIONAL
```

---

# 11. Event 可以产生“世界痕迹”

这和我们的 Echo 非常搭。

例如：

```text
Event:
door_broken

Effects:
door.state = broken

Scene Resolver下一次进入：
显示破损门
```

所以 Event 不需要负责后续每次显示“门坏了”。

它只负责：

> **世界发生了变化。**

Scene Resolver负责：

> **现在这个世界看起来是什么样。**

职责依然干净。

---

# 12. Event 和 Loop 的关系

这是我们这个项目比较特殊的部分。

每个 Event 最好明确一个：

```text
loop_policy
```

例如：

```yaml
loop_policy:
  reset: true
```

普通事件循环后全部重置。

但有些：

```yaml
loop_policy:
  create_memory: memory_017
```

于是世界重置，但玩家获得：

```text
Memory_017
```

还有极少数：

```yaml
loop_policy:
  create_echo: echo_003
```

上一轮事件给下一轮留下痕迹。

甚至：

```yaml
loop_policy:
  anomaly_delta: 1
```

推动世界失真。

于是：

```text
Event
↓
Current World Effect
+
Memory
+
Echo
+
Anomaly
```

自然和之前四层循环状态接起来了。

---

# 13. 一个完整的事件执行流程

最后我建议 Event Engine 固定这样跑：

```text
收到 Trigger
    ↓
寻找所有匹配 Event
    ↓
检查 Conditions
    ↓
检查 Repeat Policy
    ↓
加入 Event Queue
    ↓
按照 Priority 排序
    ↓
逐个执行
    ↓
应用 Effects
    ↓
需要时加入新的 Event
    ↓
输出 Presentation
    ↓
记录 Event Log
    ↓
重新 Scene Resolve
```

如果是 Hidden Event：

```text
应用 Effects
↓
不输出 Presentation
```

玩家之后通过场景变化自己发现。

---

# 14. Event Log 必须和 Action Log 对应

后台最好能看到：

```text
Loop 2
23:40

Trigger:
TIME

Event:
event_power_outage

Conditions:
PASS

Priority:
80

Effects:
power_on true → false

Presentation:
blackout_guardroom_01

Triggered Next:
event_radio_signal scheduled +5 min
```

以后排查剧情 Bug 会极其方便。

---

## Event System V1 可以最终压缩成这句话

> **Event 由 Trigger 唤起，经 Conditions 与 Repeat Policy 判断资格，按 Priority 进入事件队列，通过统一 Effect Engine 修改世界，并以可见、隐藏或延迟方式表现；事件可进一步产生新的事件、Memory、Echo 和 Anomaly。**

然后整个核心闭环现在已经非常漂亮了：

```text
Scene
  ↓
Action
  ↓
Effect
  ↓
Event
  ↓
Effect
  ↓
State
  ↓
Scene Resolver
  ↓
新的 Scene
```

再套上：

```text
Loop
├── Reset World
├── Keep Memory
├── Apply Echo
└── Increase / Change Anomaly
```

到这里，我认为我们的**传统文字游戏底层核心循环已经基本成立**。

下一步最值得设计的是 **Story Node / Story Graph V1**，因为现在我们有了“局部世界如何运行”，还缺一个东西控制：

> **整个故事怎么从序章，一路走到循环、真相和不同结局，而不变成一团 Event。**
