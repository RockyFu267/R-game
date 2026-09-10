好。这里我建议稍微认真一点，因为 **Scene Resolver 是我们传统版本第一个真正意义上的“核心引擎”**。

它解决的核心问题只有一句话：

> **玩家此时此刻来到这个地点，游戏应该呈现什么？**

## 一、先确定整体关系

我建议最终结构是：

```text
                    ┌── World State
                    │
                    ├── Story State
                    │
                    ├── Time State
                    │
Location ───────────┼── NPC State
                    │
                    ├── Player State
                    │
                    ├── Memory State
                    │
                    ├── Echo State
                    │
                    └── Anomaly State
                           │
                           ▼
                    Scene Resolver
                           │
                           ▼
                     Current Scene
```

所以 **Scene 本身不是提前写死的一篇文字**。

Scene 是系统在玩家进入地点时，根据当前状态**解析出来的结果**。

---

# 二、Location 只保存“相对永久”的东西

例如：

```yaml
location:
  id: guard_room

  name: 值班室

  description_base:
    - 一间狭窄的值班室。
    - 北侧连接走廊。
    - 东侧有一扇窗。

  objects:
    - radio
    - desk
    - locker
    - window

  exits:
    north: corridor
    south: dormitory
```

这里不要写：

> 老陈坐在桌边。

因为老陈今天在，明天可能不在。

也不要写：

> 无线电正在播放奇怪声音。

因为这是 Event / State。

Location只回答：

> **这里本质上是什么地方？**

---

# 三、Scene Resolver 每次进入都重新计算

玩家：

> 【前往值班室】

系统执行：

```text
resolve_scene(
    location = guard_room,
    current_state
)
```

然后按顺序解析。

我建议固定成 **8步**。

---

## ① 基础 Location

先读取：

```text
值班室

Objects:
radio
desk
locker
window

Exits:
corridor
dormitory
```

这是骨架。

---

## ② World Modifier

检查世界发生过什么。

例如：

```text
power_off = true
window_broken = true
radio_destroyed = false
```

那么：

```text
基础：
窗户

↓

当前：
破碎的窗户
```

或者：

```text
基础：
radio = usable

↓

当前：
radio = no_power
```

---

# ③ Time Modifier

时间影响场景。

比如：

```text
06:00–18:00
day

18:00–23:00
evening

23:00–06:00
night
```

甚至某地点可以：

```text
IF time >= 23:40
    enable observation_A
```

但这里有一个设计原则：

**时间本身最好少直接触发剧情。**

更推荐：

```text
23:40
↓
Event_017触发
↓
Event改变World State
↓
Scene Resolver读取变化
```

而不是到处写：

```text
if time > 23:40
```

否则以后会非常难维护。

---

# ④ NPC Resolver

然后检查：

> 谁现在应该在这里？

例如：

```text
NPC Chen

alive = true
location = guard_room
available = true
```

于是加入：

```text
Scene NPC:
陈
```

如果：

```text
alive = false
```

正常情况下自然不出现。

但是以后 Anomaly：

```text
anomaly >= 4
AND
special_condition
```

可能又出现。

注意：

**Scene Resolver不判断这是不是鬼。**

它只知道：

> 当前规则告诉我，这里需要呈现 NPC_Chen。

这个思想很重要。

---

# 五、Object Resolver

现在计算每个东西能不能：

**看 / 调查 / 操作。**

比如 Radio：

```yaml
radio:

  observe:
    always: true

  investigate:
    condition:
      radio_investigated == false

  actions:

    listen:
      condition:
        radio_destroyed == false

    repair:
      condition:
        has_item: fuse
```

于是玩家第一次来：

```text
【观察无线电】
【检查无线电】
【打开无线电】
```

后来无线电坏了：

```text
【观察无线电】
【检查损坏的位置】
```

如果玩家拿到了保险丝：

```text
【更换保险丝】
```

这就是状态驱动选项。

---

# 六、Clue Resolver

这一层我建议**不要直接显示线索**。

而是判断：

> 当前有哪些线索具备“可发现资格”。

例如：

```text
Clue_017

location:
guard_room

source:
radio

requirements:
  radio_investigated = true
  event_03_completed = true

exclude:
  clue_017_discovered = true
```

那么 Scene Resolver只告诉系统：

```text
clue_017:
AVAILABLE
```

玩家还必须执行：

> 【检查无线电】

才能真正获得。

这样：

**“线索存在”**

和

**“玩家发现线索”**

是两件不同的事情。

这点必须保留。

---

# 七、Memory / Echo / Anomaly Resolver

这里开始体现我们的特色。

正常选项可能：

```text
【打开无线电】
```

Memory State满足以后：

```text
【记忆】不要打开无线电
```

或者：

```text
【记忆】调到某个频率
```

Echo存在：

> 桌角似乎有什么划痕。

于是新增：

```text
【观察划痕】
```

Anomaly达到条件：

> 房间里似乎多了一把椅子。

于是：

```text
【观察多出来的椅子】
```

但是玩家UI永远不会看到：

```text
[Anomaly Level 3]
```

他只会觉得：

> 等等……

> **上次这里有这东西吗？**

这正是我们想要的。

---

# 八、Story / Event Resolver

最后才检查剧情层。

例如当前：

```text
Story Node = Chapter_2_A
```

允许：

```text
Event:
E17
E18
E23
```

禁止：

```text
E30+
```

然后检查有没有：

**进入场景立即触发的事件。**

例如：

```text
Event_18

trigger:
ENTER_LOCATION

location:
guard_room

requirements:
  story >= chapter_2
  npc_chen_alive = true
  event_18_triggered = false
```

如果满足：

玩家进入值班室。

先不出现普通 Scene。

而是：

```text
ENTER
 ↓
Event_18
 ↓
处理结果
 ↓
更新状态
 ↓
重新 Resolve Scene
```

注意最后这一点。

**事件发生以后必须重新解析场景。**

因为 Event 可能造成：

```text
NPC离开
灯熄灭
门锁上
出现新物品
时间推进
人物死亡
```

---

# 最终 Scene Resolver 输出什么？

我建议它输出一个统一的 **Scene View Model**。

例如：

```json
{
  "location": {
    "id": "guard_room",
    "name": "值班室"
  },

  "time": "23:17",

  "description_blocks": [
    "base_guard_room",
    "night_description",
    "window_broken_description"
  ],

  "npcs": [
    "npc_chen"
  ],

  "observations": [
    "radio",
    "window",
    "npc_chen"
  ],

  "investigations": [
    "radio"
  ],

  "actions": [
    "talk_chen",
    "turn_on_radio"
  ],

  "memory_actions": [
    "remember_radio_frequency"
  ],

  "exits": [
    "corridor",
    "dormitory"
  ]
}
```

UI完全不需要理解复杂剧情。

它只负责：

> Scene Resolver 给我什么，我显示什么。

这样以后你换 Web、Unity、Godot，甚至做手机端，剧情引擎都不用重写。

---

# 还有一个非常重要的问题：描述文字怎么处理？

传统游戏没有AI，所以我们不能：

```text
根据当前所有状态动态生成一段文字
```

但是也不能为每种组合人工写完整 Scene。

否则：

```text
夜晚 + 停电 + 陈在
夜晚 + 停电 + 陈不在
夜晚 + 有电 + 陈在
夜晚 + 有电 + 陈不在
白天 + ...
```

马上组合爆炸。

我的建议是：

## Block-Based Narrative

也就是**模块化叙事文本**。

不要写：

```text
scene_guard_room_version_37
```

而是：

```text
BASE
一间狭窄的值班室，墙边摆着一张掉漆的木桌。

+

TIME_NIGHT
窗外已经完全黑了。

+

POWER_OFF
头顶的灯没有亮。

+

NPC_CHEN_PRESENT
陈坐在桌边，一言不发。

+

ECHO_03
桌角似乎多了几道你不记得的划痕。
```

最终组合：

> 一间狭窄的值班室，墙边摆着一张掉漆的木桌。
>
> 窗外已经完全黑了。
>
> 头顶的灯没有亮。
>
> 陈坐在桌边，一言不发。
>
> 桌角似乎多了几道你不记得的划痕。

**完全不需要AI。**

但已经有一定动态场景能力。

将来接AI以后，甚至可以：

```text
这些 Block
     ↓
AI润色
     ↓
更自然的场景描述
```

所以我们现在做的东西未来完全不会浪费。

---

# 我建议再定一个“优先级覆盖规则”

因为以后肯定会出现冲突。

例如：

```text
BASE:
无线电在桌上。

WORLD:
无线电被玩家砸坏。

EVENT:
无线电正在播放声音。
```

到底显示哪个？

所以所有 Modifier 应该有优先级：

```text
基础 Location
      ↓
World State
      ↓
Story State
      ↓
Event State
      ↓
Loop / Echo
      ↓
Anomaly
      ↓
特殊强制覆盖
```

高优先级可以：

```text
ADD
REMOVE
REPLACE
DISABLE
ENABLE
```

例如：

```text
radio:

BASE:
usable = true

World Modifier:
radio_destroyed
→ usable = false

Event_17:
→ wants radio_playing = true

Resolver发现：
radio_destroyed = true

结果：
Event无法让已经损坏的radio正常播放
```

**除非：**

```text
Anomaly Modifier:
force_radio_playing = true
```

这时候：

> 一个已经被玩家砸坏的无线电，

> **开始说话。**

你看，连恐怖效果都可以从规则系统本身长出来。

---

## 到这里，Scene系统其实已经完整了

现在我们有：

```text
Location
   ↓
Scene Resolver
   ← World
   ← Time
   ← NPC
   ← Object
   ← Clue
   ← Story
   ← Event
   ← Memory
   ← Echo
   ← Anomaly
   ↓
Current Scene
   ↓
5+1 Actions
   ↓
玩家选择
   ↓
状态改变
   ↓
重新 Resolve
```

我认为这套应该直接作为 **《传统文字游戏需求设计》Scene System V1**。

而下一步我建议不要继续设计更大的系统了。

我们应该开始设计最重要的数据单位之一：

> **Action 到底长什么样。**

因为玩家看到的所有：

**【调查桌子】【问老陈】【前往走廊】【等待十分钟】【记忆：调到某个频率】**

底层最好全部是同一种 Action 数据结构。

如果我们把 Action 统一掉，后面 Event、Choice、时间、道具、NPC、Memory 基本就全部串起来了。
