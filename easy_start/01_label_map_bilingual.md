# 教程 01：使用 label_map 实现双语支持

本教程将教你如何使用 `label_map` 机制为内容包添加双语支持。

## 学习目标

- 理解 `label_map` 的作用和使用场景
- 学会创建双语映射文件
- 掌握 UI 标签的翻译方法

## 什么是 label_map

`label_map` 是一个专门用于管理 UI 文本映射的文件。它将英文语义键（如 `inventory`、`quest`）映射到不同语言的显示文本。

### label_map 的适用范围

**适合放入 label_map 的内容：**
- 在多个 UI 配置文件中重复出现的词语
- 导航和入口类的语义名词（如面板名称、按钮标签）
- 需要全局统一修改的文本

**不适合放入 label_map 的内容：**
- 剧情描述、地点描述
- 任务目标文本
- 上下文触发器的提示片段

## 双语格式规范

所有可显示给玩家的文本字段，统一使用以下格式：

```yaml
field_name:
  zh-cn: "中文文本"
  en-us: "English Text"
```

### 回退优先级

当请求的语言不存在时，按以下顺序回退：
1. 请求的语言（如 `zh-cn` 或 `en-us`）
2. 语言简码（如 `zh` 或 `en`）
3. 默认语言
4. 原始键名

## 实践步骤

### 第一步：创建 label_map.yaml

在内容包中创建文件：`core_system/UI/label_map.yaml`

```yaml
label_map:
  # 面板标签
  inventory:
    zh-cn: "背包"
    en-us: "Inventory"
  
  quest:
    zh-cn: "任务"
    en-us: "Quest"
  
  map:
    zh-cn: "地图"
    en-us: "Map"
  
  attributes:
    zh-cn: "属性"
    en-us: "Attributes"
  
  # 按钮标签
  explore:
    zh-cn: "探索"
    en-us: "Explore"
  
  chat:
    zh-cn: "交谈"
    en-us: "Chat"
```

### 第二步：在 panel_definitions.yaml 中使用

```yaml
panels:
  - id: "inventory_panel"
    label_key: "inventory"  # 使用 label_map 中的键
    # 其他配置...
```

### 第三步：验证效果

启动游戏后，切换语言设置，观察面板标签是否正确显示对应语言的文本。

## 最佳实践

1. **保持键名一致**：使用英文小写和下划线命名语义键
2. **集中管理**：相同的 UI 文本只在 label_map 中定义一次
3. **避免冗余**：不要在 label_map 中存放长文本或剧情内容
4. **及时更新**：新增 UI 元素时，同步更新 label_map

## 下一步

完成本教程后，继续学习 [教程 02：配置聊天框按钮](./02_chat_action_buttons.md)。
