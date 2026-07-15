---
title: Godot 游戏开发流程【代码篇】
published: 2026-07-15
pinned: false
description: "此文档记录了游戏开发的代码流程"
image: "./image/Godot_code.avif"
tags: ["游戏开发","godot"]
category: 学习记录
slug: GDP_V2
---
# Godot 游戏开发标准流程文档

> 引擎版本：Godot 4.x  
> 脚本语言：GDScript  
> 适用范围：2D / 3D 独立游戏开发

---

## 目录

1. [前期规划阶段](#1-前期规划阶段)
2. [项目初始化](#2-项目初始化)
3. [原型开发阶段](#3-原型开发阶段)
4. [核心开发阶段](#4-核心开发阶段)
5. [美术与音效集成](#5-美术与音效集成)
6. [测试与优化阶段](#6-测试与优化阶段)
7. [发布与维护](#7-发布与维护)
8. [GDScript 编码规范](#8-gdscript-编码规范)
9. [常用 Godot 设计模式](#9-常用-godot-设计模式)
10. [项目目录结构规范](#10-项目目录结构规范)

---

## 1. 前期规划阶段

### 1.1 游戏设计文档 (GDD)
在开始编码前，必须完成以下文档：

| 文档项 | 说明 |
|--------|------|
| 核心玩法定义 | 一句话描述游戏核心循环 |
| 目标平台 | PC / Mobile / Web / Console |
| 技术选型确认 | Godot 版本、渲染器 (Forward+/Mobile/Compatibility) |
| 功能清单 (Feature List) | 必须实现 vs 可选实现的功能 |
| 关卡/场景规划 | 场景流程图、关卡设计草图 |
| UI/UX 流程图 | 菜单结构、HUD 布局 |

### 1.2 技术预研
- [ ] 确认 Godot 版本（推荐稳定版 Latest Stable）
- [ ] 测试目标平台性能基准
- [ ] 评估需要的第三方插件/工具
- [ ] 确定版本控制方案（Git + Git LFS 管理大型资源）

---

## 2. 项目初始化

### 2.1 创建项目

```bash
# 建议通过 Godot 项目管理器创建
# 或使用命令行（Godot 4.x）
godot --path ./my_game --editor
```

### 2.2 项目设置 (`project.godot`)

```ini
; 关键配置项
[application]
config/name="MyGame"
config/description="游戏描述"
run/main_scene="res://scenes/main_menu.tscn"
config/features=PackedStringArray("4.2", "Forward Plus")
config/icon="res://icon.svg"

[display]
window/size/viewport_width=1920
window/size/viewport_height=1080
window/stretch/mode="canvas_items"  ; 或 "viewport"
window/stretch/aspect="expand"

[rendering]
renderer/rendering_method="forward_plus"  ; 根据平台调整

[autoload]
GameManager="*res://autoload/game_manager.gd"
AudioManager="*res://autoload/audio_manager.gd"
SceneManager="*res://autoload/scene_manager.gd"
```

### 2.3 版本控制初始化

```bash
git init
git lfs install

# 创建 .gitignore
cat > .gitignore << 'EOF'
# Godot
.import/
export.cfg
export_presets.cfg

# IDE
.vscode/
.idea/
*.tmp

# OS
.DS_Store
Thumbs.db

# 构建输出
build/
EOF

git add .
git commit -m "init: 项目初始化"
```

---

## 3. 原型开发阶段

### 3.1 核心循环原型 (Core Loop)
用最简单的几何体验证玩法可行性：

```gdscript
# 玩家控制器原型 - proto_player.gd
extends CharacterBody2D

@export var speed: float = 400.0
@export var jump_velocity: float = -600.0
@export var gravity: float = 980.0

func _physics_process(delta: float) -> void:
    # 重力
    if not is_on_floor():
        velocity.y += gravity * delta

    # 输入处理
    var direction := Input.get_axis("ui_left", "ui_right")
    velocity.x = direction * speed

    if Input.is_action_just_pressed("ui_accept") and is_on_floor():
        velocity.y = jump_velocity

    move_and_slide()
```

### 3.2 原型检查清单
- [ ] 核心操作手感是否流畅？
- [ ] 游戏目标是否清晰？
- [ ] 单次游戏循环时长是否合理？
- [ ] 是否存在明显的性能瓶颈？

### 3.3 快速迭代原则
1. **灰盒测试**：使用 ColorRect / MeshInstance 代替最终美术资源
2. **参数外置**：将关键数值设为 `@export` 变量，便于实时调试
3. **场景独立**：每个原型场景可独立运行测试

---

## 4. 核心开发阶段

### 4.1 场景架构设计

```
Main (Node)
├── GameManager (AutoLoad)
├── SceneManager (AutoLoad)
├── AudioManager (AutoLoad)
├── CurrentScene (Node)  ← 动态加载的场景根节点
│   ├── Level
│   │   ├── TileMap
│   │   ├── Entities
│   │   │   ├── Player
│   │   │   └── Enemies
│   │   └── Collectibles
│   ├── UI
│   │   ├── HUD
│   │   └── PauseMenu
│   └── Camera2D
```

### 4.2 玩家系统开发

```gdscript
# player.gd - 完整的玩家控制器示例
class_name Player
extends CharacterBody2D

# === 导出变量 ===
@export_group("Movement")
@export var max_speed: float = 300.0
@export var acceleration: float = 2000.0
@export var friction: float = 1500.0
@export var air_resistance: float = 500.0

@export_group("Jump")
@export var jump_force: float = -450.0
@export var coyote_time: float = 0.08
@export var jump_buffer: float = 0.1

# === 内部状态 ===
var _coyote_timer: float = 0.0
var _jump_buffer_timer: float = 0.0
var _is_jumping: bool = false

# === 节点引用 ===
@onready var _sprite: AnimatedSprite2D = $AnimatedSprite2D
@onready var _state_machine: StateMachine = $StateMachine

func _ready() -> void:
    add_to_group("player")

func _physics_process(delta: float) -> void:
    _handle_coyote_time(delta)
    _handle_jump_buffer(delta)
    _handle_movement(delta)
    _handle_jump()
    _update_animation()

    move_and_slide()

func _handle_movement(delta: float) -> void:
    var input_dir := Input.get_axis("move_left", "move_right")

    if input_dir != 0:
        var accel := acceleration if is_on_floor() else air_resistance
        velocity.x = move_toward(velocity.x, input_dir * max_speed, accel * delta)
        _sprite.flip_h = input_dir < 0
    else:
        var decel := friction if is_on_floor() else air_resistance
        velocity.x = move_toward(velocity.x, 0.0, decel * delta)

func _handle_jump() -> void:
    if _jump_buffer_timer > 0 and (_coyote_timer > 0 or is_on_floor()):
        velocity.y = jump_force
        _is_jumping = true
        _jump_buffer_timer = 0.0
        _coyote_timer = 0.0

    if Input.is_action_just_released("jump") and velocity.y < 0:
        velocity.y *= 0.5  # 短按小跳

func _handle_coyote_time(delta: float) -> void:
    if is_on_floor():
        _coyote_timer = coyote_time
        _is_jumping = false
    else:
        _coyote_timer -= delta

func _handle_jump_buffer(delta: float) -> void:
    if Input.is_action_just_pressed("jump"):
        _jump_buffer_timer = jump_buffer
    elif _jump_buffer_timer > 0:
        _jump_buffer_timer -= delta

func _update_animation() -> void:
    if not is_on_floor():
        _sprite.play("jump")
    elif abs(velocity.x) > 10:
        _sprite.play("run")
    else:
        _sprite.play("idle")

func take_damage(amount: int) -> void:
    GameManager.player_health -= amount
    if GameManager.player_health <= 0:
        _die()

func _die() -> void:
    GameManager.emit_signal("player_died")
    queue_free()
```

### 4.3 状态机模式

```gdscript
# state_machine.gd
class_name StateMachine
extends Node

@export var initial_state: State

var current_state: State
var states: Dictionary = {}

func _ready() -> void:
    for child in get_children():
        if child is State:
            states[child.name.to_lower()] = child
            child.state_machine = self

    if initial_state:
        transition_to(initial_state.name)

func _process(delta: float) -> void:
    if current_state:
        current_state.update(delta)

func _physics_process(delta: float) -> void:
    if current_state:
        current_state.physics_update(delta)

func transition_to(state_name: String, msg: Dictionary = {}) -> void:
    if not states.has(state_name.to_lower()):
        push_warning("State '%s' not found" % state_name)
        return

    if current_state:
        current_state.exit()

    current_state = states[state_name.to_lower()]
    current_state.enter(msg)

# state.gd (基类)
class_name State
extends Node

var state_machine: StateMachine

func enter(_msg: Dictionary = {}) -> void:
    pass

func exit() -> void:
    pass

func update(_delta: float) -> void:
    pass

func physics_update(_delta: float) -> void:
    pass
```

### 4.4 信号系统与解耦通信

```gdscript
# game_manager.gd (AutoLoad)
extends Node

signal player_health_changed(new_health: int, max_health: int)
signal player_died
signal score_changed(new_score: int)
signal game_paused(is_paused: bool)

var player_health: int = 100:
    set(value):
        player_health = clamp(value, 0, max_health)
        player_health_changed.emit(player_health, max_health)
        if player_health <= 0:
            player_died.emit()

var max_health: int = 100
var score: int = 0:
    set(value):
        score = value
        score_changed.emit(score)

var is_paused: bool = false:
    set(value):
        is_paused = value
        get_tree().paused = is_paused
        game_paused.emit(is_paused)

func add_score(points: int) -> void:
    score += points

func pause_game() -> void:
    is_paused = true

func resume_game() -> void:
    is_paused = false
```

### 4.5 场景切换管理

```gdscript
# scene_manager.gd (AutoLoad)
extends Node

const FADE_DURATION := 0.3

@onready var _transition_rect: ColorRect = $CanvasLayer/TransitionRect

func change_scene(path: String, transition: bool = true) -> void:
    if transition:
        await _fade_out()

    get_tree().change_scene_to_file(path)

    if transition:
        await get_tree().process_frame
        _fade_in()

func _fade_out() -> void:
    var tween := create_tween()
    tween.tween_property(_transition_rect, "modulate:a", 1.0, FADE_DURATION)
    await tween.finished

func _fade_in() -> void:
    var tween := create_tween()
    tween.tween_property(_transition_rect, "modulate:a", 0.0, FADE_DURATION)
```

---

## 5. 美术与音效集成

### 5.1 资源导入规范

| 资源类型 | 推荐格式 | 导入设置 |
|---------|---------|---------|
| 2D 精灵图 | PNG (无损) | Filter: Nearest (像素风) / Linear (高清) |
| 3D 模型 | GLTF 2.0 / GLB | 嵌入纹理，启用压缩 |
| 音频 (音效) | WAV / OGG | 短音效用 WAV，长音频用 OGG |
| 音频 (音乐) | OGG Vorbis | 流式播放 (Stream) |
| 字体 | TTF / OTF | 动态字体，按需预渲染 |
| TileSet | PNG + JSON | 使用 Godot TileSet 编辑器 |

### 5.2 资源组织

```
res://
├── assets/
│   ├── sprites/          # 2D 精灵图
│   ├── textures/         # 通用纹理
│   ├── models/           # 3D 模型
│   ├── animations/       # 动画资源
│   ├── audio/
│   │   ├── sfx/          # 音效
│   │   └── music/        # 背景音乐
│   ├── fonts/            # 字体文件
│   └── shaders/          # 自定义着色器
```

### 5.3 音频管理器

```gdscript
# audio_manager.gd (AutoLoad)
extends Node

@onready var _music_player: AudioStreamPlayer = $MusicPlayer
@onready var _sfx_players: Node = $SFXPlayers

const MAX_SFX_PLAYERS := 8

func _ready() -> void:
    # 预创建 SFX 播放器池
    for i in MAX_SFX_PLAYERS:
        var player := AudioStreamPlayer.new()
        player.bus = "SFX"
        _sfx_players.add_child(player)

func play_music(stream: AudioStream, fade_duration: float = 1.0) -> void:
    if _music_player.playing:
        var tween := create_tween()
        tween.tween_property(_music_player, "volume_db", -80.0, fade_duration)
        await tween.finished

    _music_player.stream = stream
    _music_player.volume_db = -80.0
    _music_player.play()

    var tween := create_tween()
    tween.tween_property(_music_player, "volume_db", 0.0, fade_duration)

func play_sfx(stream: AudioStream, pitch_random: float = 0.0) -> void:
    var player := _get_available_sfx_player()
    if not player:
        return

    player.stream = stream
    if pitch_random > 0:
        player.pitch_scale = 1.0 + randf_range(-pitch_random, pitch_random)
    player.play()

func _get_available_sfx_player() -> AudioStreamPlayer:
    for player in _sfx_players.get_children():
        if not player.playing:
            return player
    return null
```

---

## 6. 测试与优化阶段

### 6.1 调试工具

```gdscript
# debug_overlay.gd - 运行时调试信息
extends CanvasLayer

@onready var _fps_label: Label = $FPSLabel
@onready var _memory_label: Label = $MemoryLabel
@onready var _object_count_label: Label = $ObjectCountLabel

func _process(_delta: float) -> void:
    _fps_label.text = "FPS: %d" % Engine.get_frames_per_second()
    _memory_label.text = "Memory: %.1f MB" % (OS.get_static_memory_usage() / 1048576.0)
    _object_count_label.text = "Objects: %d" % Performance.get_monitor(Performance.OBJECT_COUNT)
```

### 6.2 性能优化检查清单

- [ ] **Draw Calls**：2D 使用 Sprite2D 的 `z_index` 合并，3D 使用 LOD
- [ ] **碰撞优化**：使用 Layer/Mask 过滤不必要的碰撞检测
- [ ] **对象池**：频繁创建/销毁的对象（子弹、粒子）使用对象池
- [ ] **纹理尺寸**：最大纹理尺寸不超过目标平台限制
- [ ] **阴影**：移动端减少阴影质量或禁用实时阴影
- [ ] **物理**：减少每帧的物理计算，调整 Physics FPS
- [ ] **GDScript**：避免在 `_process` 中进行 heavy 计算

### 6.3 对象池实现

```gdscript
# object_pool.gd
class_name ObjectPool
extends Node

@export var scene: PackedScene
@export var pool_size: int = 20

var _available: Array[Node] = []
var _in_use: Array[Node] = []

func _ready() -> void:
    for i in pool_size:
        var obj := scene.instantiate()
        obj.set_process(false)
        obj.hide()
        _available.append(obj)
        add_child(obj)

func acquire() -> Node:
    if _available.is_empty():
        return null

    var obj := _available.pop_back()
    obj.set_process(true)
    obj.show()
    _in_use.append(obj)
    return obj

func release(obj: Node) -> void:
    if obj in _in_use:
        _in_use.erase(obj)
        obj.set_process(false)
        obj.hide()
        _available.append(obj)

func release_all() -> void:
    while not _in_use.is_empty():
        release(_in_use[0])
```

---

## 7. 发布与维护

### 7.1 导出配置

```
项目 → 导出 → 添加预设
├── Windows Desktop
│   ├── 架构: x86_64
│   ├── 可执行图标: res://assets/icon.ico
│   └── 自定义模板: (如需裁剪引擎)
├── macOS
│   ├── 应用标识符: com.yourcompany.game
│   └── 签名证书: (如需上架 App Store)
├── Android
│   ├── 包名: com.yourcompany.game
│   ├── 目标 SDK: 最新稳定版
│   ├── 密钥库: 创建/导入
│   └── 权限: 按需申请
└── Web
    ├── 导出类型: Regular
    └── 线程支持: (视浏览器兼容性)
```

### 7.2 发布前检查清单

- [ ] 所有场景可正常加载，无报错
- [ ] 输入映射完整测试（键盘、手柄、触屏）
- [ ] 分辨率适配测试（最小/推荐/最大）
- [ ] 存档系统验证
- [ ] 音效/音乐音量平衡
- [ ] 无调试代码残留（print、@tool 调试节点）
- [ ] 最终构建使用 Release 模板
- [ ] 版本号更新 (`project.godot` → `config/version`)

### 7.3 版本号规范

采用语义化版本：`MAJOR.MINOR.PATCH`

```gdscript
# version_manager.gd
const VERSION := "1.0.0"
const BUILD_NUMBER := 100  # CI/CD 自动递增
```

---

## 8. GDScript 编码规范

### 8.1 命名规范

| 类型 | 规范 | 示例 |
|------|------|------|
| 类名 (class_name) | PascalCase | `PlayerController`, `EnemyBase` |
| 节点名 | PascalCase | `Player`, `MainCamera` |
| 变量/属性 | snake_case | `max_speed`, `is_jumping` |
| 常量 | UPPER_SNAKE_CASE | `MAX_HEALTH`, `GRAVITY` |
| 信号 | snake_case | `health_changed`, `player_died` |
| 函数 | snake_case | `take_damage()`, `move_to()` |
| 私有函数/变量 | 下划线前缀 | `_calculate_damage()` |
| 枚举 | PascalCase + 大写成员 | `enum State { IDLE, RUN, JUMP }` |

### 8.2 代码格式

```gdscript
# 类型标注（Godot 4.x 推荐）
var health: int = 100
var speed: float = 300.0
var target: Node2D

# 函数签名
func take_damage(amount: int, source: Node = null) -> void:
    pass

# 信号定义
signal health_changed(new_health: int, max_health: int)

# 枚举
enum State {
    IDLE,
    RUN,
    ATTACK,
    DEAD
}

# 数组/字典类型
var enemies: Array[Enemy] = []
var stats: Dictionary[String, int] = {}

# 常量分组
const MAX_SPEED: float = 500.0
const JUMP_FORCE: float = -450.0
const COYOTE_TIME: float = 0.08
```

### 8.3 文档注释

```gdscript
## 玩家核心控制器类
## 处理移动、跳跃、受伤等基础逻辑
class_name Player
extends CharacterBody2D

## 最大移动速度 (像素/秒)
@export var max_speed: float = 300.0

## 对玩家造成伤害
## [param amount]: 伤害数值
## [param source]: 伤害来源节点（可选）
func take_damage(amount: int, source: Node = null) -> void:
    pass
```

### 8.4 代码组织顺序

```gdscript
extends Node

# 1. 类名与继承
class_name MyClass

# 2. 信号
signal my_signal

# 3. 枚举
enum MyEnum { A, B, C }

# 4. 常量
const MY_CONSTANT := 10

# 5. 导出变量 (按分组)
@export_group("Movement")
@export var speed: float

@export_group("Combat")
@export var damage: int

# 6. 普通变量
var _private_var: int

# 7. @onready 变量
@onready var _sprite: Sprite2D = $Sprite2D

# 8. 内置虚函数
func _ready() -> void:
    pass

func _process(delta: float) -> void:
    pass

func _physics_process(delta: float) -> void:
    pass

# 9. 公共函数
func public_function() -> void:
    pass

# 10. 私有函数
func _private_function() -> void:
    pass
```

---

## 9. 常用 Godot 设计模式

### 9.1 观察者模式（信号）

```gdscript
# 发布者
signal enemy_died(position: Vector2, score_value: int)

func _on_health_depleted() -> void:
    enemy_died.emit(global_position, score_value)
    queue_free()

# 订阅者
func _ready() -> void:
    for enemy in get_tree().get_nodes_in_group("enemies"):
        enemy.enemy_died.connect(_on_enemy_died)

func _on_enemy_died(pos: Vector2, score: int) -> void:
    GameManager.add_score(score)
    spawn_death_effect(pos)
```

### 9.2 单例模式（AutoLoad）

```gdscript
# game_data.gd (AutoLoad)
extends Node

var player_data: PlayerData
var settings: GameSettings

func save_game() -> void:
    var save_file := FileAccess.open("user://savegame.save", FileAccess.WRITE)
    var data := {
        "player": player_data.to_dict(),
        "settings": settings.to_dict()
    }
    save_file.store_var(data)

func load_game() -> bool:
    if not FileAccess.file_exists("user://savegame.save"):
        return false
    var save_file := FileAccess.open("user://savegame.save", FileAccess.READ)
    var data := save_file.get_var()
    player_data.from_dict(data["player"])
    settings.from_dict(data["settings"])
    return true
```

### 9.3 组件模式

```gdscript
# health_component.gd
class_name HealthComponent
extends Node

signal health_changed(new_health: int, max_health: int)
signal died

@export var max_health: int = 100

var current_health: int:
    set(value):
        current_health = clamp(value, 0, max_health)
        health_changed.emit(current_health, max_health)
        if current_health <= 0:
            died.emit()

func _ready() -> void:
    current_health = max_health

func take_damage(amount: int) -> void:
    current_health -= amount

func heal(amount: int) -> void:
    current_health += amount

# 使用方式：将 HealthComponent 作为子节点添加到任何需要血条的节点
```

### 9.4 资源模式（数据驱动）

```gdscript
# weapon_data.gd
class_name WeaponData
extends Resource

@export var name: String
@export var damage: int
@export var fire_rate: float
@export var projectile_scene: PackedScene
@export var icon: Texture2D

# 创建资源文件：右键 → 新建资源 → WeaponData
# 可在编辑器中直接配置武器数据，无需修改代码
```

---

## 10. 项目目录结构规范

```
res://
├── project.godot              # 项目配置文件
├── icon.svg                   # 项目图标
├── README.md                  # 项目说明
│
├── autoload/                  # 全局单例（AutoLoad）
│   ├── game_manager.gd
│   ├── audio_manager.gd
│   ├── scene_manager.gd
│   └── save_manager.gd
│
├── assets/                    # 原始资源文件
│   ├── sprites/
│   ├── textures/
│   ├── models/
│   ├── animations/
│   ├── audio/
│   │   ├── sfx/
│   │   └── music/
│   ├── fonts/
│   └── shaders/
│
├── resources/                 # .tres 数据资源
│   ├── weapons/
│   ├── items/
│   └── dialogues/
│
├── scenes/                    # 场景文件 (.tscn)
│   ├── levels/                # 关卡场景
│   ├── ui/                    # UI 场景
│   ├── characters/            # 角色场景
│   ├── enemies/               # 敌人场景
│   └── effects/               # 特效场景
│
├── scripts/                   # 脚本文件 (.gd)
│   ├── components/            # 可复用组件
│   ├── states/                # 状态机状态
│   ├── utils/                 # 工具类
│   └── interfaces/            # 接口/抽象类
│
├── tests/                     # 测试场景和脚本
│   ├── unit/
│   └── integration/
│
└── docs/                      # 项目文档
    ├── gdd.md                 # 游戏设计文档
    ├── api.md                 # 内部 API 文档
    └── changelog.md           # 更新日志
```

---

## 附录

### A. 推荐 Godot 插件

| 插件 | 用途 |
|------|------|
| **Godot Git Plugin** | 内置 Git 版本控制 |
| **Aseprite Wizard** | Aseprite 动画导入 |
| **Dialogue Manager** | 可视化对话系统 |
| **Phantom Camera** | 高级相机控制 |
| **SoundManager** | 高级音频管理 |

### B. 学习资源

- [Godot 官方文档](https://docs.godotengine.org/)
- [GDScript 风格指南](https://docs.godotengine.org/en/stable/tutorials/scripting/gdscript/gdscript_styleguide.html)
- [Godot 最佳实践](https://docs.godotengine.org/en/stable/tutorials/best_practices/index.html)

### C. 常用快捷键

| 快捷键 | 功能 |
|--------|------|
| `F5` | 运行项目 |
| `F6` | 运行当前场景 |
| `F7` | 逐语句调试 |
| `Ctrl + Shift + F` | 全局搜索 |
| `Ctrl + K` | 注释/取消注释 |
| `Ctrl + Shift + O` | 快速打开文件 |

---

*本文档遵循 Godot 4.x 最佳实践，根据项目实际需求可灵活调整。*
