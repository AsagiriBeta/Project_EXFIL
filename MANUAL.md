# Project EXFIL 使用文档

Project EXFIL 是一个基于 Minecraft Paper (AdvancedSlimePaper) 的塔科夫/三角洲行动风格的 FPS 撤离类游戏插件。

## 1. 安装与依赖 (Installation)

### 服务端核心
*   **AdvancedSlimePaper (ASP)**: 本插件深度依赖 ASP 的 SlimeWorldManager API 来实现高性能的地图实例化和回滚。

### 依赖插件
*   **PlaceholderAPI**: 用于变量替换。
*   **FastAsyncWorldEdit (FAWE) / WorldEdit**: 用于管理员框选区域（撤离点、NPC区域、战利品区域等）。
*   **DecentHolograms**: 用于显示撤离点的倒计时全息图。
*   **ProtocolLib**: 用于协议级别的功能支持。
*   **QualityArmory**: 提供枪械系统支持。
*   **Citizens**: NPC系统支持。
*   **Sentinel**: NPC战斗AI支持。
*   **Parties**: 用于组队系统。
*   **Vault**: 用于经济系统显示。
*   **XConomy**: 经济插件，本插件已对其进行适配。
*   **TAB**: 用于计分板和 Tab 列表显示。
*   **LuckPerms**: 用于权限管理。
*   **GSit**: 玩家交互动作支持。
*   **ItemsAdder**: 提供自定义物品支持。

### 内置库
*   **Inventory Framework**: 用于构建 GUI 界面。

---

## 2. 管理员指南 (Admin Guide)

### 地图管理 (Map Management)

本插件采用"导入-转换-注册"的流程来管理地图。

1.  **准备地图文件**:
    *   支持 **普通世界文件夹** (Vanilla World Folder)。
    *   支持 **ZIP 压缩包** (推荐)。
    *   支持 **.slime 文件**。
2.  **上传**: 将文件放入 `plugins/Project_EXFIL/import_maps/` 文件夹。
3.  **导入命令**:
    *   `/exfil importmap <文件名> <显示名称>`
    *   *示例*: `/exfil importmap KritCity.zip "Krit City"`
    *   *注意*: 插件会自动解压、过滤垃圾文件、将普通地图转换为 Slime 格式，并存入 `slime_worlds` 库中。

### 区域设置 (Region Setup)

在导入地图后，您需要进入该地图的模板世界进行设置。

1.  **进入地图**: 使用 ASP 或 SWM 的命令进入模板世界（通常在 `slime_worlds` 中）。
2.  **设置撤离点 (Extraction Points)**:
    *   使用 WorldEdit 木斧 (`//wand`) 框选撤离区域。
    *   输入 `/exfil set extract <名称>` 保存。
    *   *提示*: 撤离点对所有以此地图为模板的游戏实例生效。
3.  **设置出生区域 (Spawn Region)**:
    *   站在您希望作为出生中心的位置。
    *   输入 `/exfil set spawn <半径>` (例如 200)。
    *   *机制*: 游戏开始时，系统会在该半径内随机寻找一个**安全、露天、且远离撤离点和其他玩家**的位置部署玩家。
4.  **设置 NPC 生成区域**:
    *   使用 WorldEdit 框选 NPC 生成区域。
    *   输入 `/exfil set npc <名称> <数量>` 保存。
    *   *示例*: `/exfil set npc scavs_area 5`
5.  **设置战利品生成区域**:
    *   使用 WorldEdit 框选战利品箱生成区域。
    *   输入 `/exfil set loot <名称> <数量>` 保存。
    *   *示例*: `/exfil set loot loot_boxes 10`

### 战利品系统配置

1.  **打开战利品编辑器**:
    *   输入 `/exfil loot` 打开战利品编辑器界面。
2.  **添加战利品**:
    *   从背包中拖拽物品到编辑器界面自动添加。
3.  **设置概率**:
    *   左键点击物品编辑生成概率 (0.0-1.0)。
4.  **移除物品**:
    *   右键点击物品从战利品表中移除。

### 安全箱系统

1.  **激活安全箱**:
    *   玩家右键使用"安全箱使用权"道具激活安全箱。
    *   不同等级的安全箱提供不同容量 (格子数)。
2.  **打开安全箱**:
    *   输入 `/exfil secure` 打开个人安全箱。
3.  **物品限制**:
    *   带有"不可存储至安全箱"标签的物品无法放入安全箱。

### 查看与删除

*   **查看列表**:
    *   `/exfil list maps`: 查看所有已注册的地图。
    *   `/exfil list extracts`: 查看所有撤离点。
    *   `/exfil list npc`: 查看所有 NPC 生成区域。
    *   `/exfil list loot`: 查看所有战利品生成区域。
*   **删除数据**:
    *   `/exfil delete map <ID>`: 删除地图配置及对应的 Slime 世界文件（保留 import_maps 中的源文件）。
    *   `/exfil delete extract <名称>`: 删除指定的撤离点。
    *   `/exfil delete npc <名称>`: 删除指定的 NPC 生成区域。
    *   `/exfil delete loot <名称>`: 删除指定的战利品生成区域。

---

## 3. 游戏机制 (Game Mechanics)

### 基础机制

*   **部署 (Deploy)**: 玩家通过 `/exfil` 菜单选择地图匹配。系统会创建独立的临时游戏实例。
*   **出生点 (Spawning)**: 智能算法确保玩家出生在固体方块上，且头顶无遮挡（室外），并自动避开危险区域（岩浆、水等）。
*   **撤离 (Extraction)**: 玩家到达撤离点后，会看到个人专属的倒计时全息图。倒计时结束即可携带战利品撤离。
*   **死亡与清理 (Death & Cleanup)**:
    *   玩家死亡或掉线会立即被移除出游戏实例。
    *   当实例内无玩家时，系统会在短暂延迟后自动卸载并删除该临时世界，释放服务器资源。

### 组队系统

*   **创建小队**: 最多 3 人组队进行游戏。
*   **邀请系统**: 队长可以邀请其他玩家加入小队。
*   **同步部署**: 小队成员会一起部署到同一游戏实例。

### NPC 系统

*   **AI 敌人**: 在指定的 NPC 区域内生成智能敌人 (Scavengers)。
*   **武器装备**: NPC 配备 QualityArmory 武器 (如 AK47、M4A1S) 或默认铁剑。
*   **战斗行为**: NPC 会主动攻击玩家，具有追击、射击等行为模式。

### 战利品系统

*   **随机生成**: 在指定的战利品区域内按概率生成战利品箱。
*   **概率控制**: 每个物品都有独立的生成概率。
*   **物品多样性**: 支持自定义物品、武器、装备、道具等。

### 安全箱系统

*   **持久存储**: 安全箱内的物品在死亡时不会丢失。
*   **容量等级**: 不同等级的安全箱提供不同存储容量。
*   **使用限制**: 特定物品无法存入安全箱。

### 脚步系统

*   **声音提示**: 玩家移动时会产生脚步声。
*   **状态区分**: 跑步和走路有不同的声音间隔。
*   **可配置**: 支持自定义脚步间隔时间。

### UI 界面系统

*   **主菜单**: 集成部署、组队、仓库等功能。
*   **地图选择**: 可视化地图选择和难度显示。
*   **小队管理**: 直观的队员管理和邀请界面。
*   **仓库系统**: 分页显示和物品整理功能。

### 边界系统

*   **作战区域**: 定义游戏的边界范围。
*   **离开警告**: 玩家离开作战区域时会收到警告。
*   **安全保护**: 防止玩家进入未授权区域。

### 全息图系统

*   **撤离点标记**: 在撤离点显示全息文字标识。
*   **倒计时显示**: 撤离过程中显示倒计时信息。

### BossBar 系统

*   **撤离点导航**: 显示最近的撤离点方向和距离。
*   **实时更新**: 动态更新撤离点信息和方向指引。

### 计分板系统

*   **游戏信息显示**: 显示当前地图、余额、军衔、行动时间等。
*   **个性化内容**: 支持自定义计分板内容和样式。

---

## 4. 命令速查 (Commands)

### 玩家命令
| 命令 | 描述 | 权限 |
| :--- | :--- | :--- |
| `/exfil` | 打开游戏主菜单 (部署/组队/仓库) | `exfil.use` |
| `/exfil stash` | 打开个人仓库 (Stash) | `exfil.use` |
| `/exfil secure` | 打开安全箱 | `exfil.use` |

### 管理员命令 (权限: `exfil.admin`)
| 命令 | 描述 |
| :--- | :--- |
| `/exfil importmap <文件> <名称>` | 导入地图 (支持 zip/文件夹) |
| `/exfil set extract <名称>` | 将 WE 选区设为撤离点 |
| `/exfil set spawn <半径>` | 设置随机出生范围 |
| `/exfil set npc <名称> <数量>` | 设置 NPC 生成区域 |
| `/exfil set loot <名称> <数量>` | 设置战利品生成区域 |
| `/exfil list maps` | 列出所有地图 |
| `/exfil list extracts` | 列出所有撤离点 |
| `/exfil list npc` | 列出所有 NPC 生成区域 |
| `/exfil list loot` | 列出所有战利品生成区域 |
| `/exfil delete map <ID>` | 删除地图 |
| `/exfil delete extract <名称>` | 删除撤离点 |
| `/exfil delete npc <名称>` | 删除 NPC 生成区域 |
| `/exfil delete loot <名称>` | 删除战利品生成区域 |
| `/exfil loot` | 打开战利品编辑器 |

---

## 5. 配置文件 (Configuration)

### config.yml
```yaml
# Update interval for chest UI
chest-ui-update-interval: 5

# Footstep update interval
footstep-interval: 4

# Sprint footstep make noise interval
footstep-interval-sprint: 4

# Walk footstep make noise interval
footstep-interval-walk: 8
```

### 语言配置
插件使用 JSON 格式的语言文件，位于 `resources/languages/zh_CN.json`，支持完整的 MiniMessage 格式和占位符系统。

---

## 6. 开发特性 (Development Features)

### API 集成
*   **ItemsAdder API**: 支持自定义物品集成。
*   **PlaceholderAPI**: 提供丰富的占位符支持。
*   **QualityArmory**: 完整的枪械系统集成。
*   **Citizens + Sentinel**: 强大的 NPC 系统。

### 性能优化
*   **SlimeWorldManager**: 高性能世界管理。
*   **异步保存**: 数据异步保存，避免卡顿。
*   **内存管理**: 智能的游戏实例清理机制。

### 扩展性
*   **模块化设计**: 游戏功能模块化，易于扩展。
*   **事件系统**: 完整的事件监听和处理机制。
*   **GUI 框架**: 灵活的 UI 构建框架。

---

## 7. 权限节点 (Permission Nodes)

| 权限节点 | 描述 | 默认值 |
| :--- | :--- | :--- |
| `exfil.use` | 基本使用权限 | true |
| `exfil.admin` | 管理员权限 | op |

