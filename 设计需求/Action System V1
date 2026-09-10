好，Scene System V1 正式确定并加入快照。下一步我们就做 **Action System V1**。

我建议这里坚持一个原则：

> **玩家看到的是不同类型的行为，程序底层看到的全部都是 Action。**

也就是说这些：

```text
【观察桌子】
【调查无线电】
【询问老陈】
【拿走钥匙】
【前往仓库】
【等待10分钟】
【记忆：调到那个频率】
```

底层全部统一：

```text
Action
```

只是 `type` 不一样。

### Action 最小结构

第一版我建议不要搞得太复杂：

```yaml
action:
  id: investigate_radio

  type: investigate

  label: 检查无线电

  target: radio

  conditions:
    - radio_destroyed == false

  cost:
    time: 5

  effects:
    - set radio_investigated = true

  result:
    success_text: investigate_radio_01

  next:
    resolve_scene: true
```

它其实只回答几个问题：

> **我是谁？玩家什么时候能看到我？点我需要付出什么？点完发生什么？然后去哪？**

这就够了。

---

### 六种操作不需要六套代码

统一定义：

```text
Action
├── OBSERVE
├── INVESTIGATE
├── TALK
├── INTERACT
├── MOVE
├── WAIT
└── MEMORY
```

这里实际上是7个 type，因为之前“移动/等待”在玩家体验上属于一组，但程序内部我建议拆开。

比如：

```yaml
type: move
target: warehouse
cost:
  time: 8
```

和：

```yaml
type: wait
cost:
  time: 10
```

逻辑完全不同。

---

## Action最重要的是 Condition

比如：

```text
【检查床底】
```

默认存在。

但是：

```text
【用钥匙打开柜子】
```

要求：

```text
has_item(key_03)
AND
locker_locked
```

而：

```text
【记忆：直接调到 107.3】
```

要求：

```text
memory.radio_frequency_known
```

所以 Action 可以有三种状态：

```text
AVAILABLE
玩家可以看到，可以选择

HIDDEN
玩家完全看不到

DISABLED
玩家能看到，但现在不能做
```

我尤其建议谨慎使用 `DISABLED`。

例如：

> 🔒【打开地下室铁门】

这其实已经告诉玩家：

**这里存在一个“打开地下室”的玩法。**

有时候这本身就是剧透。

所以恐怖游戏默认应该：

> **条件不满足 → HIDDEN**

只有我们故意想让玩家知道“这里有事情以后可以做”，才显示 Disabled。

---

# Action执行顺序

玩家点击：

> 【调查无线电】

我建议严格执行：

```text
1. 再次检查 Conditions
        ↓
2. 扣除 Cost
        ↓
3. 推进游戏时间
        ↓
4. 应用 Effects
        ↓
5. 检查 Triggered Events
        ↓
6. 显示 Action Result
        ↓
7. Event（如果发生）
        ↓
8. Scene Resolver重新解析
```

为什么点击以后还要**再次检查条件**？

因为UI显示这个按钮的时候满足条件，不代表真正执行的时候状态一定还没变化。

这属于工程上的保险。

---

# Effect 也应该统一

以后不要在剧情代码里到处：

```text
chenTrust -= 10
radioOpen = true
...
```

而是统一 Effect：

```yaml
effects:

  - type: set_flag
    key: radio_investigated
    value: true

  - type: add_variable
    key: chen_trust
    value: -10

  - type: add_item
    item: key_03

  - type: discover_clue
    clue: clue_017

  - type: unlock_memory
    memory: memory_04

  - type: move_npc
    npc: chen
    location: corridor
```

这样以后我们做：

**存档、回档、调试、事件日志、循环Reset**

都会非常舒服。

---

## 还有一个我特别建议现在就加：Action Consequence

玩家选择之前，**不要告诉他所有后果**。

UI：

> 【继续追问老陈】

后台：

```text
time +5
chen_trust -8
chen_suspicion +12
flag: questioned_chen = true
```

玩家只看到：

> 老陈的脸色明显沉了下来。

而不是：

> `老陈好感 -8`

这和我们之前“隐藏数值”的原则一致。

**状态存在，但不把游戏玩成Excel。**

---

# Action可以触发Event，但不要自己写剧情树

比如：

```text
Action:
调查柜子

Effect:
cabinet_searched = true
```

然后 Event System：

```text
IF
cabinet_searched
AND
time > 23:00
AND
chen_alive
AND
event_32_not_triggered

THEN
Event_32
```

这样以后“调查柜子”本身不用知道：

> 我会触发什么恐怖剧情。

**Action负责行为。**

**Event负责世界反应。**

职责非常干净。

---

# Memory Action稍微特殊一点

它仍然是普通 Action：

```yaml
type: memory
```

但来源不是当前世界，而是：

```text
Memory State
```

比如：

```yaml
action:
  id: memory_radio_frequency

  type: memory

  label: "【记忆】调到107.3"

  conditions:
    - memory.radio_frequency_known

  cost:
    time: 1

  effects:
    - set radio_frequency = 107.3
```

所以从代码角度：

**【记忆】根本不是另一套系统。**

它只是：

> **由跨循环状态解锁的一种特殊 Action。**

这会让实现简单很多。

---

## 最后还有一个很有价值的东西：Action Log

每一次玩家行动都记录：

```text
Loop 2
DAY 1
23:17

Action:
investigate_radio

Location:
guard_room

Time Cost:
5 min

Effects:
radio_investigated = true

Triggered:
event_017

Result:
success
```

玩家看不到这个后台日志。

但**开发者必须有**。

以后我们遇到：

> “为什么我第二轮没有出现这个选项？”

直接看 Action Log + State 就能查。

否则这种大量 Flag、Event、Loop 的游戏调试起来会非常痛苦。

---

所以 **Action System V1** 可以浓缩成：

> **Action = Conditions + Cost + Effects + Result。**

然后：

```text
Scene Resolver
      ↓
生成当前可用 Actions
      ↓
玩家选择
      ↓
检查 Conditions
      ↓
支付 Cost
      ↓
执行 Effects
      ↓
Event System 检查世界反应
      ↓
记录 Action Log
      ↓
重新 Resolve Scene
```

这样我们目前几个核心系统已经真正接起来了：

**Location → Scene → Action → State → Event → Scene**

再叠加以后已经确认的：

**Memory / Echo / Anomaly / Loop**

传统版本的“引擎骨架”已经开始完整了。

我建议下一步设计 **Event System V1**。因为 Action 是“玩家对世界做什么”，Event 就是另外一半——**“世界对玩家做什么”**。这两个一旦都定下来，核心玩法闭环基本就完成了。
