# 教程 04：Flow Node 事件系统

本教程将教你如何使用 Flow Node 定义主动事件（按钮触发）和被动事件（概率触发）。

## 学习目标

- 理解 Flow Node 的作用
- 学会创建主动事件
- 学会创建被动概率事件
- 掌握事件链的触发机制

## Flow Node 简介

Flow Node 是 ERA 引擎的核心机制，用于定义游戏中的所有事件和动作。每个节点包含：

- **触发条件**（requires）：判断事件是否可以执行
- **效果**（effects）：执行状态变更
- **叙事钩子**（narrative_hook）：为 AI 提供叙事提示

## 文件组织

在内容包中创建 `flow_nodes/` 目录：

```
content_pack_root/
└── flow_nodes/
    ├── actions.yaml          # 主动事件
    └── passive_events.yaml   # 被动事件
```

## 实践步骤一：创建主动事件

主动事件由玩家点击按钮触发。

### 创建 flow_nodes/actions.yaml

```yaml
nodes:
  # 接受任务事件
  - node_id: "story.accept_mission"
    label:
      zh-cn: "接受讨伐任务"
      en-us: "Accept Subjugation Quest"
    trigger_type: active
    category: story
    available_scenes: ["exploration"]
    
    # 显示条件
    visible_if: "not has_flag($player.flags, 'mission_accepted')"
    
    # 执行条件
    requires:
      - "$location.id == 'dawn_village'"
      - "not has_flag($player.flags, 'mission_accepted')"
    
    # 效果
    effects:
      - target: "flags.mission_accepted"
        op: set
        value: 1
      - target: "player.stats.courage"
        op: add
        value: 2
    
    # 叙事提示
    narrative_hook:
      key: "accept_mission"
      hint:
        zh-cn: "村长把讨伐龙王的任务交给玩家。"
        en-us: "The village elder entrusts the player with the mission."
```

### 字段说明

| 字段 | 说明 |
|------|------|
| `node_id` | 唯一标识符 |
| `trigger_type` | 触发类型：`active` 或 `passive` |
| `available_scenes` | 可在哪些场景触发 |
| `visible_if` | 按钮显示条件 |
| `requires` | 执行条件 |
| `effects` | 状态变更效果 |
| `narrative_hook` | AI 叙事提示 |

## 实践步骤二：创建被动事件

被动事件不需要玩家点击，在特定条件下自动触发。

### 创建 flow_nodes/passive_events.yaml

```yaml
nodes:
  # 森林伏击事件
  - node_id: "passive.forest_ambush"
    trigger_type: passive
    trigger_hook: "enter_forest"  # 触发钩子
    
    # 触发概率
    probability: 0.3  # 30% 概率
    
    # 触发条件
    requires:
      - "$location.id == 'dark_forest'"
      - "$player.flags.forest_crossed != 1"
    
    # 效果
    effects:
      - target: "player.hp"
        op: sub
        value: 10
      - target: "flags.forest_ambush_triggered"
        op: set
        value: 1
    
    narrative_hook:
      key: "forest_ambush"
      hint:
        zh-cn: "玩家在森林中遭到伏击！"
        en-us: "The player is ambushed in the forest!"
```

### 被动事件字段说明

| 字段 | 说明 |
|------|------|
| `trigger_hook` | 触发钩子名称 |
| `probability` | 触发概率（0-1） |

## 实践步骤三：触发钩子

被动事件通过钩子触发。你可以在主动事件中发射钩子：

```yaml
# 在 actions.yaml 中
nodes:
  - node_id: "explore.forest"
    label:
      zh-cn: "进入森林"
      en-us: "Enter Forest"
    trigger_type: active
    available_scenes: ["exploration"]
    
    requires:
      - "$location.id == 'forest_entrance'"
    
    effects:
      - target: "location.id"
        op: set
        value: "dark_forest"
    
    # 发射钩子，触发被动事件
    hooks_emit:
      - "enter_forest"
    
    narrative_hook:
      key: "enter_forest"
      hint:
        zh-cn: "玩家踏入幽暗的森林。"
        en-us: "The player steps into the dark forest."
```

## 完整示例

### actions.yaml

```yaml
nodes:
  - node_id: "explore.village"
    label:
      zh-cn: "探索村庄"
      en-us: "Explore Village"
    trigger_type: active
    available_scenes: ["exploration"]
    effects:
      - target: "player.xp"
        op: add
        value: 5
    narrative_hook:
      key: "explore_village"
      hint:
        zh-cn: "玩家在村庄中四处查看。"
        en-us: "The player looks around the village."
  
  - node_id: "rest.at_inn"
    label:
      zh-cn: "在旅店休息"
      en-us: "Rest at Inn"
    trigger_type: active
    available_scenes: ["exploration"]
    requires:
      - "$location.id == 'village_inn'"
    effects:
      - target: "player.hp"
        op: set
        value: 100
      - target: "player.stamina"
        op: set
        value: 100
    narrative_hook:
      key: "rest_inn"
      hint:
        zh-cn: "玩家在旅店休息，恢复了体力。"
        en-us: "The player rests at the inn and recovers."
```

### passive_events.yaml

```yaml
nodes:
  - node_id: "passive.find_gold"
    trigger_type: passive
    trigger_hook: "explore_anywhere"
    probability: 0.1
    effects:
      - target: "player.gold"
        op: add
        value: 10
    narrative_hook:
      key: "find_gold"
      hint:
        zh-cn: "玩家意外发现了一些金币！"
        en-us: "The player accidentally finds some gold!"
```

## 验证方法

1. 启动游戏，测试主动事件按钮是否正常显示和执行
2. 检查 visible_if 条件是否正确控制按钮显示
3. 测试 requires 条件是否正确阻止不符合条件的事件
4. 多次触发带有钩子的事件，验证被动事件的概率触发

## 下一步

完成本教程后，你已经掌握了 ERA 内容包开发的核心机制。建议：

1. 尝试组合多个 Flow Node 创建复杂剧情
2. 探索更多条件表达式和效果类型
3. 参考 `docs/tutorials/` 中的高级教程

祝你的创作顺利！
