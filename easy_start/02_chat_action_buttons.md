# 教程 02：配置聊天框按钮

本教程将教你如何在聊天框中配置自定义按钮，包括主按钮和子按钮。

## 学习目标

- 理解按钮配置的覆盖关系
- 学会创建主按钮和子按钮
- 掌握按钮与 Flow Node 的绑定

## 按钮的优先级

聊天框按钮的优先级从高到低：

1. **内容包定义的动态按钮**（scene_actions）
2. **引擎默认按钮**（DEFAULT_ACTIONS）

## 实践步骤

### 第一步：创建 interactions.yaml

在内容包根目录创建 `interactions.yaml` 文件：

```yaml
# interactions.yaml
scene_actions:
  exploration:
    # 主按钮 - 直接执行
    - id: "explore"
      name:
        zh-cn: "探索"
        en-us: "Explore"
      type: "main"
      node_id: "explore.general"
    
    # 父按钮 - 点击展开子菜单
    - id: "social"
      name:
        zh-cn: "社交"
        en-us: "Social"
      type: "parent"
      children:
        - id: "chat"
          name:
            zh-cn: "闲聊"
            en-us: "Chat"
          node_id: "social.chat"
        
        - id: "trade"
          name:
            zh-cn: "交易"
            en-us: "Trade"
          node_id: "social.trade"
```

### 按钮类型说明

| 类型 | 说明 | 用途 |
|------|------|------|
| `main` | 直接点击执行 | 常用动作，如探索、攻击 |
| `parent` | 展开显示子按钮 | 动作分组，如社交、技能 |

### 第二步：配置按钮显示条件

你可以控制按钮在何时显示：

```yaml
scene_actions:
  exploration:
    - id: "secret_passage"
      name:
        zh-cn: "秘密通道"
        en-us: "Secret Passage"
      type: "main"
      node_id: "explore.secret"
      visible_if: "$player.flags.found_map == 1"  # 只有找到地图才显示
```

### 第三步：覆盖默认按钮（可选）

如果你想修改引擎默认的兜底按钮，在内容包根目录创建 `settings.yaml`：

```yaml
settings:
  state_machine:
    actions:
      exploration: ["探索", "观察环境", "搜寻线索"]
      dialogue: ["交谈", "询问", "说服"]
      combat: ["攻击", "防御", "技能", "撤退"]
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
      node_id: "explore.general"
    
    - id: "rest"
      name:
        zh-cn: "休息"
        en-us: "Rest"
      type: "main"
      node_id: "player.rest"
    
    - id: "interact"
      name:
        zh-cn: "互动"
        en-us: "Interact"
      type: "parent"
      children:
        - id: "talk"
          name:
            zh-cn: "对话"
            en-us: "Talk"
          node_id: "npc.talk"
        
        - id: "examine"
          name:
            zh-cn: "调查"
            en-us: "Examine"
          node_id: "object.examine"
```

## 验证方法

1. 启动游戏进入探索状态
2. 检查聊天框下方是否正确显示配置的按钮
3. 点击父按钮，确认子菜单正常展开
4. 测试按钮功能是否正常触发

## 下一步

完成本教程后，继续学习 [教程 03：场景切换控制](./03_scene_transition_and_validation.md)。
