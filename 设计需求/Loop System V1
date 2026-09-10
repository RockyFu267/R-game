好。现在做 **Loop System V1**，实际上是在给前面所有系统规定一次“生命周期”。

我先定一个最重要的原则：

> **Loop 不是“重新开始游戏”，而是一次世界状态重建。**

玩家没有回到存档点。
**世界回去了，但玩家没有。**

这两者区别非常大。

---

# 一、Loop 在系统里到底是什么

我建议把一次循环定义成一个完整的运行实例：

```text id="x3w27p"
Loop Instance

Loop ID: 3
Start Time: 18:00
Current Time: 23:47

├── World State
├── NPC State
├── Current Story State
├── Event State
├── Item State
├── Scene State
└── Action / Event History
```

而循环之外还有一个：

```text id="dkt5b8"
Meta State

├── Memory
├── Knowledge
├── Claims
├── Previous Timelines
├── Echo
├── Anomaly
├── Meta Story Progress
└── Loop History
```

所以整个存档实际上是：

```text id="j0ocxr"
SAVE
│
├── Meta State          ← 跨循环
│
└── Current Loop        ← 当前这一轮
```

这非常干净。

---

# 二、循环开始时不是“恢复存档”，而是 World Rebuild

例如第一轮结束时：

```text id="3v02un"
老陈：死亡
仓库门：损坏
无线电：被玩家砸坏
钥匙：玩家持有
时间：23:57
```

Loop发生。

不要：

```text id="v87o92"
load save_001
```

而是：

```text id="98e8sz"
Loop Manager
      ↓
读取 World Baseline
      ↓
生成新一轮 World
      ↓
应用 Meta Story Progress
      ↓
应用 Echo
      ↓
应用 Anomaly
      ↓
应用特殊 Loop Rules
      ↓
生成 NPC 初始状态
      ↓
建立 Event Schedule
      ↓
Scene Resolver
```

最后新世界可能是：

```text id="wmzk4q"
老陈：活着
仓库门：恢复
无线电：恢复
钥匙：回到原处

但是：

桌上出现上一轮留下的划痕   ← Echo

名单多了一个名字           ← Anomaly

玩家知道无线电频率         ← Memory
```

这才是我们的循环。

---

# 三、哪些东西 Reset？

我们已经有四层状态，所以现在可以正式把 Reset Matrix 定下来。

| 状态                  | Loop后              | 说明               |
| ------------------- | ------------------ | ---------------- |
| 普通 World State      | Reset              | 门、灯、电源等恢复        |
| NPC 生死/位置           | Reset              | 默认恢复基线           |
| 普通 Relationship     | Reset              | NPC不记得上一轮        |
| 普通 Inventory        | Reset              | 物品回到世界           |
| 当前时间                | Reset              | 回到循环起点           |
| Event 状态            | Reset              | 按Repeat Policy重建 |
| 当前 Scene            | Reset              | 重新解析             |
| Memory              | **Keep**           | 玩家记得             |
| Knowledge           | **Keep**           | 玩家认知保留           |
| Claims              | **Keep**           | 保留历史判断           |
| Timeline History    | **Keep**           | 上轮时间线仍在          |
| Echo                | **Keep/Transform** | 根据规则进入下一轮        |
| Anomaly             | **Keep/Change**    | 累积或阶段变化          |
| Meta Story Progress | **Keep**           | 故事继续向前           |

但是这里必须留一个例外机制：

```text id="q4nrzp"
PERSIST_OVERRIDE
```

某些东西可以违反默认 Reset。

比如以后剧情需要：

> 玩家上一轮把钥匙埋进某处，下一轮钥匙居然还在那里。

不是把 Inventory 整套设为持久化。

而是：

```text id="b6p0s1"
Item:
key_03

loop_policy:
PERSIST_AS_ECHO
```

这才安全。

---

# 四、循环怎么结束？

我不希望只有：

> 玩家死亡 → Loop。

我们之前已经确定这一点。

所以我建议 V1 有五种：

```text id="9w5nxa"
DEATH
死亡

TIME_LIMIT
到达循环时间终点

CRITICAL_EVENT
关键事件导致循环

PLAYER_TRIGGER
玩家主动触发

STORY_TRIGGER
剧情阶段强制进入下一轮
```

但它们最终全部统一成：

```text id="3tr63h"
request_loop_transition(reason)
```

Loop Manager不关心玩家为什么循环。

它只负责：

> **有人请求结束这一轮。**

---

# 五、非常重要：Loop不能瞬间 Reset

循环发生应该有一个：

## Loop Transition

例如：

```text id="qjm17z"
正常游戏
↓
触发 Loop
↓
冻结普通 Action
↓
处理当前关键 Event
↓
生成本轮 Summary
↓
结算 Memory
↓
结算 Echo
↓
结算 Anomaly
↓
推进 Meta Story
↓
World Rebuild
↓
新循环 Opening
↓
恢复玩家操作
```

这样我们就获得了一个非常重要的机会：

> **在两轮之间做结算。**

---

# 六、Loop Settlement

这是整个系统真正的核心。

一轮结束以后，系统问几个问题。

### 这一轮玩家学到了什么？

例如：

```text id="nd2iqa"
Clue A
Clue B
Event C

↓

unlock:
memory_radio_frequency
```

---

### 这一轮玩家做了什么值得留下 Echo？

例如：

```text id="cxrggq"
玩家连续三轮都在墙上刻下记号
```

可能满足：

```text id="xndg0m"
Echo Condition
```

于是：

```text id="a81yvt"
create_echo:
wall_mark_03
```

---

### 这一轮让世界恶化了多少？

例如：

```text id="mvhdci"
Anomaly +1
```

但我不建议：

```text id="vt7coz"
每循环一次固定 +1
```

太机械。

更好：

```text id="59lq9k"
基础循环：
+0

接触异常：
+1

强行改变关键事件：
+1

触碰某禁忌：
+2

特殊剧情：
set anomaly_stage = X
```

所以玩家甚至可能：

> 循环很多次，但世界仍比较稳定。

或者：

> 两轮就把世界搞得非常糟。

---

# 七、Anomaly 不建议只做一个数字

这个我想稍微升级一下我们之前的设计。

后台可以有：

```text id="2ezf6f"
anomaly_pressure = 37
```

用于计算。

但是玩法最好使用：

```text id="sg8s0y"
Anomaly Stage

0 = Stable
1 = Subtle
2 = Distorted
3 = Unstable
4 = Collapse
```

这样剧情作者比较好控制。

例如：

### Stage 0 — Stable

世界几乎严格重复。

让玩家学习规律。

### Stage 1 — Subtle

出现非常小的偏差。

```text id="2pbxv6"
时间差1分钟
物品位置改变
一句话措辞不同
```

### Stage 2 — Distorted

玩家确定世界开始变化。

```text id="o0zox4"
NPC路线改变
事件提前
名单变化
Echo明显出现
```

### Stage 3 — Unstable

已经掌握的规律开始失效。

```text id="kh8j2u"
Confirmed Knowledge
→ Unstable
```

### Stage 4 — Collapse

世界规则严重失真。

用于后期/终局。

这样非常符合我们已经定下来的：

> 相信世界 → 发现循环 → 利用循环 → 规则背叛玩家。

---

# 八、Echo 到底怎么生成？

我特别不希望 Echo 变成：

> 上轮所有东西随机残留。

那会失去意义。

Echo必须是**有叙事意义的跨循环因果**。

我建议三种来源：

### Action Echo

玩家做过某件特殊行为：

```text id="n67gkz"
刻字
藏东西
破坏某物
留下某种标记
```

---

### Event Echo

重大事件留下：

```text id="ab0c7p"
某人死亡
火灾
枪击
特殊异常事件
```

---

### Emotional / Narrative Echo

这个可以很恐怖。

不是物理残留，而是：

> 世界似乎“记得”某件事。

例如上一轮某NPC死在这里。

下一轮他活着。

但进入这个房间时：

> 他突然停了一下。

> “……不知道为什么，我不太想进去。”

NPC并没有真正获得上一轮完整 Memory。

这是：

```text id="rmgzuv"
Echo Influence
```

我很喜欢这种。

它让：

> **玩家记得。**

慢慢发展成：

> **世界好像也开始记得。**

---

# 九、NPC跨循环记忆必须分等级

我们之前说过极少数NPC可以记得。

现在可以正式结构化。

```text id="ed76ze"
Memory Level 0
完全重置

Level 1
Deja Vu
模糊既视感

Level 2
Emotional Echo
保留情绪/恐惧/亲近感

Level 3
Fragment
记得片段

Level 4
Aware
明确知道循环
```

绝大多数：

```text id="fm8v4l"
Level 0
```

少量随着故事：

```text id="xam6l5"
0 → 1
```

极少数特殊角色：

```text id="k6fnd8"
2 / 3
```

而真正：

```text id="5vrkfr"
Level 4
```

应该是重大剧情事件。

这样我们不会滥用“NPC也记得”。

---

# 十、循环次数不能等于剧情进度

这个必须明确。

不要：

```text id="wp9vzf"
Loop 1 = Chapter 1
Loop 2 = Chapter 2
Loop 3 = Chapter 3
```

否则玩家其实还是在线性看剧情。

应该允许：

```text id="pl8kr0"
玩家A：
6轮进入真相阶段

玩家B：
9轮

玩家C：
4轮
```

取决于：

```text id="i9dz9e"
Knowledge
Narrative Requirements
行为
探索效率
关键选择
```

所以：

> **Loop Count 是历史，不是等级。**

---

# 十一、但也不能允许无限循环完全没有代价

否则玩家会：

> 那我每轮把所有选项试一遍。

直接暴力穷举。

这会毁掉推理。

所以我们需要一种**软性反穷举机制**。

Anomaly就是最好的答案。

玩家可以继续Loop。

但是：

```text id="xjcl4d"
循环越多
≠ 单纯越来越难
```

而应该是：

```text id="3hjfof"
重复干预世界
↓
Anomaly风险增加
↓
旧规律可靠性下降
↓
未来预测越来越困难
```

于是循环不是：

> 免费SL大法。

而是：

> **一种有风险的资源。**

但这里我不建议简单惩罚“循环次数”。

应该惩罚：

> **某些危险行为和对世界的过度干预。**

玩家正常探索失败，不应该被游戏恶意惩罚。

---

# 十二、我建议加入 Loop Signature

这个东西玩家看不到。

每一轮结束后生成一个摘要：

```text id="5knq8s"
Loop #3

Duration:
5h47m

Key Actions:
- 提前进入仓库
- 阻止Chen死亡
- 打开地下室

Knowledge Gained:
- radio_frequency
- chen_knows_blackout

Major Changes:
- npc_li_dead
- basement_opened

Echo Created:
- wall_mark_03

Anomaly Delta:
+2

Ending Reason:
TIME_LIMIT
```

这个叫：

```text id="0caj86"
Loop Signature
```

它以后有三个巨大用途：

**调试、存档历史、结局判定。**

甚至未来玩家通关以后可以给他看：

> 你经历了7次循环。

> 第3轮第一次救下某人。

> 第5轮第一次进入地下室。

这个会很有味道。

---

# 十三、玩家 Notebook 怎么表现多轮历史？

我建议不要每次循环清空笔记。

而是有：

```text id="fz2ct8"
当前循环
历史记录
```

例如时间线：

```text id="8pg3ob"
──── Loop 1 ────

23:40 停电
23:43 仓库出现枪声

──── Loop 2 ────

23:37 停电
23:43 仓库没有枪声

──── Loop 3 ────

23:40 未发生停电
```

这时候玩家不用游戏告诉他：

> 世界越来越异常。

**他自己看笔记就知道。**

这非常好。

---

# 十四、Loop开始时不要每次把序章重播一遍

这是体验上的大坑。

第一次：

```text id="bnhuej"
完整 Opening
```

第二次：

可以有：

```text id="l7t6im"
缩短版
+
Memory Reaction
```

第三次以后：

甚至：

```text id="1cpdt7"
【直接开始行动】
```

除非世界已经发生变化。

例如第四轮：

玩家本来准备快速跳过。

结果开场第一句话：

> **床边站着一个上一轮不存在的人。**

这时候游戏强制完整展示。

所以 Opening 也应该由 Scene/Event系统解析，而不是固定动画。

---

# 十五、Loop Transition 本身也可以被玩家逐渐理解

前期：

> 玩家不知道发生了什么。

第一次循环甚至不显示：

```text id="k23l11"
Loop 2
```

直接让玩家重新醒来。

这样才恐怖。

玩家：

> 等等？

中期确认循环后，Notebook才开始主动区分：

```text id="uvtl83"
第一次
第二次
第三次
```

后期玩家甚至可以：

```text id="g28gxk"
【主动触发循环】
```

这意味着一个很漂亮的玩法成长：

```text id="4ms9rv"
被循环支配
↓
理解循环
↓
利用循环
↓
主动操纵循环
↓
发现循环已经不再完全听话
```

这基本就是我们的整个游戏体验曲线。

---

# Loop System V1 最终结构

可以压缩成：

```text id="6fz1tc"
              Meta State
       ┌─────────────────────┐
       │ Memory              │
       │ Knowledge           │
       │ Claims              │
       │ Timeline History    │
       │ Echo                │
       │ Anomaly             │
       │ Meta Story Progress │
       └──────────┬──────────┘
                  │
                  ▼
             World Rebuild
                  │
                  ▼
          ┌── Current Loop ──┐
          │ World            │
          │ NPC              │
          │ Time             │
          │ Items            │
          │ Events           │
          │ Story State      │
          └────────┬─────────┘
                   │
             玩家不断行动
                   │
                   ▼
             Loop Trigger
                   │
                   ▼
             Loop Settlement
             /      |       \
        Memory     Echo    Anomaly
             \      |       /
                   ▼
           Meta Story Progress
                   │
                   ▼
             World Rebuild
                   │
                   ▼
                Next Loop
```

所以我会把 **Loop System V1** 的设计哲学最终定成三句话：

> **世界回到过去，玩家的认知没有。**

> **循环不是免费的读档，而是玩家逐渐学会利用、同时也可能污染世界的一种核心资源。**

> **前期玩家被循环困住，中期玩家利用循环，后期循环本身开始失去可靠性。**

到这里，我们的核心玩法已经不是一个概念了。**Scene → Action → Event → Knowledge → Story → Loop → World Rebuild** 已经形成完整闭环。

下一步我反而不建议继续加系统。现在最值得做的是把这些东西收束成一份 **Core Game Loop V1 + MVP边界**：明确玩家从“打开游戏”到“完成一次循环”的完整实际体验，以及第一版到底实现哪些东西、坚决不实现哪些东西。这样我们就能从“设计架构”开始进入“可以开发”的阶段。
