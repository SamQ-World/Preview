# 教程 03：控制场景切换

本教程将教你如何在离开对话后控制场景切换，让游戏流程更符合剧情设计。

## 学习目标

- 理解场景切换的机制
- 学会配置对话离开后的场景跳转
- 掌握条件判断在场景切换中的应用

## 场景状态说明

游戏中的主要场景状态：

| 状态 | 说明 |
|------|------|
| `exploration` | 探索状态，自由行动 |
| `dialogue` | 对话状态，与 NPC 交谈 |
| `combat` | 战斗状态 |
| `management` | 管理状态，如整理背包 |
| `game_over` | 游戏结束 |

## 默认行为与自定义

**默认行为**：离开对话后总是回到探索状态。

**自定义需求**：根据剧情需要，离开后可能进入：
- 战斗状态（遭遇埋伏）
- 新的对话（被追上继续交谈）
- 探索状态（正常离开）

## 实践步骤

### 第一步：在 interactions.yaml 中添加配置

```yaml
# interactions.yaml
scene_actions:
  dialogue:
    - id: "leave"
      name:
        zh-cn: "离开"
        en-us: "Leave"
      type: "main"

# 场景切换配置
dialogue_exit:
  default_next_state: "exploration"
  default_message:
    zh-cn: "你结束了交谈，重新环顾四周。"
    en-us: "You end the conversation and look around again."
  
  rules:
    # 规则 1：在龙穴且已穿越森林，触发战斗
    - requires:
        - "$player.flags.forest_crossed == 1"
        - "$location.id == 'dragon_lair'"
      next_state: "combat"
      message:
        zh-cn: "你刚转身，龙王已逼近，战斗一触即发！"
        en-us: "As you turn away, the Dragon King closes in. Combat erupts!"
    
    # 规则 2：已接受任务但未穿越森林，继续对话
    - requires:
        - "$player.flags.mission_accepted == 1"
        - "$player.flags.forest_crossed != 1"
      next_state: "dialogue"
      message:
        zh-cn: "村长追上来叮嘱你：先穿越森林，再谈决战。"
        en-us: "The elder catches up: cross the forest before talking about the final duel."
```

### 配置字段说明

| 字段 | 说明 |
|------|------|
| `default_next_state` | 没有规则匹配时的默认跳转状态 |
| `default_message` | 默认跳转时显示的消息 |
| `rules` | 规则列表，按顺序匹配 |
| `requires` | 条件表达式，满足则执行该规则 |
| `next_state` | 满足条件后跳转到的状态 |
| `message` | 跳转时显示的消息 |

### 第二步：设计规则优先级

规则按列表顺序匹配，**越具体的规则越靠前**：

```yaml
rules:
  # 最具体的规则放前面
  - requires:
      - "$player.hp < 10"
      - "$location.id == 'danger_zone'"
    next_state: "combat"
    message:
      zh-cn: "你虚弱的状态被敌人察觉，遭到袭击！"
  
  # 较通用的规则放后面
  - requires:
      - "$player.hp < 10"
    next_state: "exploration"
    message:
      zh-cn: "你决定先找个安全的地方休息。"
```

## 完整示例

```yaml
# interactions.yaml
scene_actions:
  exploration:
    - id: "explore"
      name:
        zh-cn: "探索"
        en-us: "Explore"
      type: "main"
  
  dialogue:
    - id: "leave"
      name:
        zh-cn: "离开"
        en-us: "Leave"
      type: "main"

dialogue_exit:
  default_next_state: "exploration"
  default_message:
    zh-cn: "你结束了对话。"
    en-us: "You end the conversation."
  
  rules:
    - requires:
        - "$npc.attitude == 'hostile'"
      next_state: "combat"
      message:
        zh-cn: "对方不想让你离开！"
        en-us: "They won't let you leave!"
    
    - requires:
        - "$npc.has_more_to_say == 1"
      next_state: "dialogue"
      message:
        zh-cn: "等等，我还有话要说！"
        en-us: "Wait, I have more to say!"
```

## 验证方法

1. 进入对话状态
2. 点击"离开"按钮
3. 检查是否按预期跳转到正确的场景
4. 验证不同条件下是否触发不同的跳转

## 注意事项

1. 规则按顺序匹配，第一个满足条件的规则会生效
2. `next_state` 必须是合法的状态值
3. 建议始终设置 `default_next_state` 作为兜底
4. 消息文本使用双语格式，确保语言切换正常

## 下一步

完成本教程后，继续学习 [教程 04：Flow Node 事件系统](./04_flow_nodes_active_and_passive_events.md)。
