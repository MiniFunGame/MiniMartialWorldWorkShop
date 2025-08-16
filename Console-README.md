# 控制台 README

## 打开与收起
- **Tab**：显示/隐藏输入框（`CommandController.ToggleInputFieldVisibility`）。
- **Enter**：提交命令。  

> 输入框已设置为**单行**（`InputField.LineType.SingleLine`），避免上下键被当作换行。

## 历史记录
- **↑**：上一条命令（首次按会备份当前正在编辑的内容）。
- **↓**：下一条命令；超过最后一条会回到“提交前正在编辑的内容”。
- 连续重复的相同命令不会重复入历史；历史仅在本次运行内有效。

## 命令总览（语法）
> 统一格式：`命令 参数…`（大小写不限，空格分隔）

### 1) `exp`
- 作用：获得 10000 阅历（`AttributeManager.ModifyExperience(10000)`）。

### 2) `buff`
- 作用：连发 100 次随机增益奖励展示（`DisplayControl.ShowStrategyAward(typeof(BuffAward), 3)`）。

### 3) `money <数值>`
- 作用：直接改铜钱（`AttributeManager.ModifyMoney(int)`）。  
- 示例：`money 500`（+500 铜钱）。

### 4) `skill <技能名>`
- 作用：添加技能（`SkillBuilder.AddSkill`）。  
- 示例：`skill 分光`。

### 5) `award <关键字> [顺序]`
- 作用：按名称**模糊匹配**奖励，默认取**最高品质**；提供顺序（从 1 开始）可精确指定匹配项。  
- 内部流程：先 `AwardManager.AddAward(typeof(SkillEffect))` 构建候选 → 名称 `Contains`（不区分大小写）→ 按 `Quality` **降序**。  
- 示例：  
  - `award 金剑`（匹配名称含“金剑”，取品质最高的那一个）；  
  - `award 金剑 2`（匹配结果里取第 2 个）。

### 6) `event <事件类名>`
- 作用：通过反射查找 `Event` 子类并实例化展示（大小写不敏感）。  
- 示例：`event GoldenTower_Teaching`。

### 7) `modify <属性> <数值> [notify]`
- 作用：反射调用 `AttributeManager` 的 `Modify{属性名}` 方法。  
- 匹配优先级：`(int,bool)` → `(float,bool)` → `(int)` → `(float)`。  
- `<属性>` 会经过**首字母大写**（`strength` → `Strength`）来拼方法名。  
- `notify` 省略则默认 `true`；小数用 `.`（如 `1.5`）。  
- 示例：  
  - `modify attack 50`（调用 `ModifyAttack(int)`）；  
  - `modify speed 1.25 false`（调用 `ModifySpeed(float,bool)`，且不弹提示）。

#### 常用属性名（对应方法后缀）
`Strength, Physique, Speed, Concentrate, Charm, Comprehension, Lucky, Attack, Defense, Agility, HitRate, Evasion, CriticalRate, CriticalMultiplier, MaxHealth, MaxMana, ...`

> 只要 `AttributeManager` 存在 `Modify{Name}`，就能被 `modify` 调到。  
> `money` 同样可用 `modify money 100 false`，因为有 `ModifyMoney(int,bool)`。

## 行为与限制
- **HardCore 难度**：`Start()` 中会直接禁用整个控制台（安全阀）。
- **事件查找**：遍历当前 AppDomain 的所有程序集中的 `Event` 子类，按**类名**精准匹配（不区分大小写）。
- **奖励池**：`AwardManager.AddAward` 当前写死使用 `typeof(SkillEffect)` 构建。如果要按不同基类构建，可调整入参来源或实现真正按传入的 `awardBaseType` 过滤。

## 使用示例
```text
exp
money 2000
skill 背水
modify strength 3.5
modify attack 120 
award 金剑
award 金剑 2
event GoldenTower_Teaching
```

## 常见问题（FAQ）
- **小数解析失败？** 仅支持 `.` 作为小数点（如 `1.25`），不要用中文逗号或中文句点。
- **找不到事件？** 请确认事件类名与代码中的**类型名**一致（例如类名 `GoldenTower_Teaching`，命令也应如此）。

