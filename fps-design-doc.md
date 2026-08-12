# 仿COD UE FPS 射击游戏 Demo · 设计文档

> 本项目旨在打造一款**手感接近《使命召唤20：现代战争III》（Call of Duty: Modern Warfare III）**的第一人称射击游戏。以 UE 5.8 纯蓝图从零搭建，围绕"射击手感"这一核心命题，构建了涵盖武器操控、移动反馈、命中表现、弹药管理与靶场训练的完整 FPS 原型。实现了全自动 Hitscan 射击、Enhanced Input 全键位映射以及靶场小球即时补位系统，力求在开火响应、视觉冲击与操作流畅度上达到平均射击手感水平。

> **项目名称**: 仿COD UE FPS 射击游戏 Demo
> **引擎版本**: Unreal Engine 5.8
> **开发语言**: 纯蓝图（Blueprint）
> **武器原型**: SIG MCX-VIRTUS 突击步枪（300 BLK）
> **开发周期**: 个人项目
> **当前版本**: v0.42

---

## 版本规划

| 版本 | 主题 | 状态 |
|------|------|------|
| v0.00 | 基准 | — |
| v0.1 | 3C 开发（角色、相机、操控） | ✅ 已完成 |
| v0.2 | 射击系统开发 | ✅ 已完成 |
| v0.3 | 外围系统开发 | ✅ 已完成 |
| v0.4 | 靶场初见雏形 | ✅ 已完成 |
| v0.41 | 补弹功能 | ✅ 已完成 |
| v0.42 | 增加小球 | ✅ 当前版本 |
| v1.00 | 靶场功能结束（实现大部分 Aimlab 功能） | 🎯 下一目标 |
| v2.00 | 滑铲 + 蹲下 + COD 战役新手教程白盒 | 🎯 远期目标 |

---

## 一、已实现系统总览

本项目基于 UE 5.8 纯蓝图开发，从零搭建了一套 FPS 射击原型。目前已实现 **8 个核心系统**，覆盖输入、战斗、反馈、靶场交互全链路。

### 1.1 系统清单

| # | 系统 | 核心蓝图 | 一句话说明 |
|---|------|----------|-----------|
| 1 | 玩家角色系统 | BP_FPSPlayer | 第一人称角色操控，多种移动状态，FOV 平滑切换 |
| 2 | 武器系统 | BP_WeaponBase → BP_VIRTUS | Hitscan 即时命中，全自动开火，弹匣/备弹管理 |
| 3 | 输入系统 | IMC_FPSinput + 7 个 IA | Enhanced Input 全映射，WASD + 鼠标 + 功能键 |
| 4 | 命中反馈系统 | BP_WeaponBase (LineTrace) | 4 种物理材质差异化 VFX/SFX/弹孔贴花 |
| 5 | 靶场小球系统 | BP_TargetManager + BP_TargetBall + BP_StartButton | 射击按钮增加小球，打掉即时补位，无上限 |
| 6 | 弹药补给系统 | BP_AmmoLoader | 射击补给箱或近身按 F 键补满弹药至 180 发 |
| 7 | 动画系统 | ABP_FPSPlayer + ABP_VIRTUS | 双动画蓝图分层管理，BlendSpace 八向移动，AnimNotify 换弹同步 |
| 8 | 弹壳系统 | BP_Casing | 物理抛壳 + 碰撞音效 + 自动销毁 |

### 1.2 功能矩阵

| 系统 | 已实现功能 |
|------|-----------|
| 玩家角色 | 步行 / 冲刺 / 瞄准移动 · FOV 平滑过渡 · 鼠标灵敏度控制 · BeginPlay 自动生成武器 |
| 武器 | 全自动开火 · Hitscan 射线检测 · 弹匣 30 发 + 备弹 180 发 · R 键换弹 · 弹药耗尽禁射 · 枪口闪光 |
| 输入 | 开火 / 瞄准 / 移动 / 视角 / 换弹 / 冲刺 / 补弹（F 键）共 7 个 Input Action |
| 命中反馈 | 4 种表面材质判定 · 对应 VFX + SFX + 弹孔贴花 · Cast 链路分发（小球→按钮→弹药箱→环境） |
| 靶场小球 | 射击按钮增加 1 球 · 无上限累加 · 打掉 1 个立即补 1 个 · 管理器位置即生成中心 · SpawnRange 控制范围 |
| 弹药补给 | 射击补给箱补满 · F 键近距离补满 |
| 动画 | 玩家多状态 BlendSpace · 武器多状态 BlendSpace · 换弹 AnimNotify 同步弹药 |
| 弹壳 | 开火抛壳 · 抛物线运动 · 随机旋转 · 落地音效 · 定时销毁 |

---

## 二、核心玩法

**当前玩法**：射击靶场小球。射击"增加小球"按钮可向靶场区域添加小球（无上限），每次射击按钮增加 1 个；打掉一个立即补一个，始终保持当前数量。

**v1.00 目标**：实现大部分 Aimlab 的功能。

**v2.00 目标**：加入滑铲、蹲下，以及一个类似于 COD 战役模式开头新手教程的白盒关卡。

---

## 三、系统设计详情

### 3.1 玩家角色系统（BP_FPSPlayer）

**继承链**: `Character` → `BP_FPSPlayer`

#### 组件结构

| 组件 | 类型 | 说明 |
|------|------|------|
| Camera | CameraComponent | FPS 第一人称摄像机 |
| CharacterMesh0 | SkeletalMeshComponent | 角色手臂模型（SK_Base_Skin） |
| CharacterMovement | CharacterMovementComponent | 移动行为控制 |

#### 核心参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| Default FOV | 100° | 默认视场角 |
| Mouse Sensitivity | 1.0 | 鼠标灵敏度 |
| Walk Speed | 265 | 步行速度 |
| ADS Speed | 400 | 瞄准移动速度 |
| Sprint Speed | 600 | 冲刺速度 |

#### 状态机（E_PlayerState 枚举）

| 状态 | 触发条件 |
|------|----------|
| Idle | 无移动输入 |
| Walk | WASD 移动 |
| Sprint | 左 Shift + 移动 |
| ADS | 鼠标右键按下 |

#### 核心逻辑

- **FOV 平滑过渡**: 瞄准时 `FInterpTo` 在默认 FOV 和瞄准 FOV 之间插值
- **移动速度切换**: 根据 `E_PlayerState` 切换 `MaxWalkSpeed`
- **武器生成**: BeginPlay 时 Spawn `BP_VIRTUS` 实例并赋值给 `CurrentWeapon` 变量
- **F 键补弹**: IA_AddAmmo 触发 → Get CurrentWeapon → 调用 AddAmmo 函数

---

### 3.2 武器系统

#### 3.2.1 武器基类（BP_WeaponBase）

**继承链**: `Actor` → `BP_WeaponBase`
**实现接口**: `BI_Fire`

| 核心变量 | 类型 | 说明 |
|----------|------|------|
| BaseDamage | Float | 单发基础伤害 |
| CurrentAmmo | Int | 当前弹匣内弹药（默认 30） |
| MagSize | Int | 弹匣容量 |
| TotalAmmo | Int | 总备弹数（默认 180） |
| FireRate | Float | 射速（发/秒） |
| CurrentWeapon | Object Ref | 当前武器实例引用 |
| Barrel | SceneComponent | 枪管 Socket，枪口闪光/弹壳生成位置 |

#### 开火逻辑（LineTrace → Cast 链路）

```
LineTraceSingle (Visibility 通道)
  起点: 摄像机位置
  终点: 摄像机前向 × 10000
  → Is Valid (HitResult 有效?)
    → Is Valid: 击中音效/特效
      → Cast To BP_TargetBall
        ├─ Succeeded → Hit() → 销毁球 + 补球
        └─ Failed → Cast To BP_StartButton
          ├─ Succeeded → Hit() → 增加小球
          └─ Failed → Cast To BP_AmmoLoader
            ├─ Succeeded → Hit() → Set TotalAmmo(180) + 更新UI
            └─ Failed → 环境命中特效（VFX/SFX/弹孔）
```

#### 3.2.2 MCX-VIRTUS 突击步枪（BP_VIRTUS）

**继承链**: `BP_WeaponBase` → `BP_VIRTUS`

| 配件 | 说明 |
|------|------|
| 300-BACKOUT 弹匣 | 30 发 .300 BLK 弹匣 |
| DMR-X 枪管 | 长枪管，提升精度 |
| MCX-VIRTUS Close XSights | 机械瞄具 |
| Sig Sauer Romeo3 | 红点瞄准镜 |

---

### 3.3 输入系统（Enhanced Input）

| 输入动作 | 按键 | 触发方式 | 功能 |
|----------|------|----------|------|
| IA_Fire | 鼠标左键 | Started | 开火 |
| IA_Aim | 鼠标右键 | Triggered | 瞄准（ADS） |
| IA_Move | W/A/S/D | Continuous | 角色移动 |
| IA_Look | Mouse2D | Continuous | 视角旋转 |
| IA_Reload | R | Triggered | 换弹 |
| IA_Run | 左 Shift | Started | 冲刺 |
| IA_AddAmmo | F | Started | 近身补弹 |

**输入映射**: `IMC_FPSinput` 统一管理，BeginPlay 时挂载。

---

### 3.4 命中检测与反馈系统

#### 射线检测参数

| 参数 | 值 |
|------|---|
| 起点 | 摄像机世界位置 |
| 终点 | 起点 + 摄像机前向 × 10000 |
| 通道 | Visibility |
| 输出 | HitResult（命中点 / Actor / 组件 / 物理材质） |

#### 表面材质响应表（4 种）

| 表面材质 | 击中 VFX | 击中 SFX | 弹孔贴花 |
|----------|----------|----------|----------|
| 混凝土 | FX_SquibConcrete | SFX_Concrete_Cue | MI_BulletDecal_Concrete |
| 泥土 | FX_SquibDirt | SFX_Dirt_Cue | 默认弹孔 |
| 金属 | FX_SquibMetal | SFX_Metal_Cue | MI_BulletDecal_Metal |
| 木头 | FX_SquibWood | SFX_Wood_Cue | MI_BulletDecal_Wood |

---

### 3.5 靶场小球系统

#### 3.5.1 系统架构

| 蓝图 | 角色 | 说明 |
|------|------|------|
| BP_StartButton | 增加小球按钮 | 被射击时调用 Manager.SpawnOneBall()，每次增加 1 球 |
| BP_TargetManager | 管理器 | 负责生成小球，位置即生成中心 |
| BP_TargetBall | 靶标 | 被射击时销毁自身 + 通知 Manager 补球 |

#### 3.5.2 BP_TargetManager

| 变量 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| SpawnRange | Float | 500 | 生成范围半径（可编辑） |
| Score | Integer | 0 | 得分 |
| ElapsedTime | Float | 0 | 计时秒数 |

| 函数 | 说明 |
|------|------|
| SpawnOneBall | GetActorLocation + RandomUnitVector × SpawnRange → SpawnActor(BP_TargetBall) |
| AddScore | Score + 1 → 更新分数显示 |

#### 3.5.3 BP_TargetBall

| 组件 | 说明 |
|------|------|
| StaticMesh (Sphere) | 球体网格，Collision Presets = BlockAll |

| 函数 | 说明 |
|------|------|
| Hit | Get All Actors → Get(0) → SpawnOneBall() → AddScore() → DestroyActor(Self) |

#### 3.5.4 BP_StartButton

| 组件 | 说明 |
|------|------|
| StaticMesh | 按钮外观，Collision = BlockAll |

| 函数 | 说明 |
|------|------|
| Hit | Get All Actors → Get(0) → SpawnOneBall() → PrintString("增加小球") |

#### 执行流程

```
射击 BP_StartButton → Manager.SpawnOneBall()  ← 增加 1 个球
  （每次射击按钮增加 1 球，无上限累加）

射击 BP_TargetBall → Hit()
  → Manager.SpawnOneBall()  ← 立即补 1 个
  → Manager.AddScore()      ← 加 1 分
  → Destroy Actor (Self)    ← 销毁自己

例：射击按钮 3 次 → 场上 3 个球 → 打掉 1 个立即补 1 个 → 始终保持 3 个
```

---

### 3.6 弹药补给系统（BP_AmmoLoader）

> ⚠️ 外观组件（StaticMesh / Text Render）尚未接入，当前仅功能逻辑可用。

#### 补弹方式

| 方式 | 触发条件 | 逻辑 |
|------|----------|------|
| 射击补给箱 | LineTrace 命中 BP_AmmoLoader | Hit() → Set TotalAmmo(180) → 更新 UI |
| F 键补弹 | 玩家近身按 F | IA_AddAmmo → Get CurrentWeapon → AddAmmo() → Set CurrentAmmo(30) + TotalAmmo(180) → 更新 UI |

#### 关键实现

- **AddAmmo 必须是 Function 不是 Custom Event**：跨蓝图调用只能用 Function
- **Target 引脚连 CurrentWeapon 变量**，不是 Weapon 变量
- 补弹后调用 `updateweaponinfo` 刷新 HUD

---

### 3.7 动画系统

#### 3.7.1 玩家动画蓝图（ABP_FPSPlayer）

**骨骼**: `Skeleton_Base`

| 状态 | 动画/混合 | 说明 |
|------|-----------|------|
| Idle/Walk | BS_FPSPlayer + Player_Idle | 2D BlendSpace 驱动待机↔八向移动 |
| ADS Idle | Player_ADS_Idle | 瞄准待机 |
| ADS Walk | Player_ADS_Walk | 瞄准移动 |
| Sprint | Player_Sprint | 冲刺动画 |

**过渡条件**: `E_PlayerState` 枚举 + Velocity.Length > 0 + ADS Socket 位置判断

#### 3.7.2 武器动画蓝图（ABP_VIRTUS）

**骨骼**: `MCX-VIRTUS_Skeleton`

| 状态 | 动画/混合 | 说明 |
|------|-----------|------|
| Idle/Walk | BS_VIRTUS + Weapon_Idle | 武器 2D BlendSpace |
| ADS | Weapon_ADS_Idle + Weapon_ADS_Walk | 武器瞄准状态 |
| Sprint | Weapon_Sprint | 武器冲刺动画 |

#### 3.7.3 换弹动画通知（AN_Reload）

`AN_Reload` 在换弹蒙太奇关键帧触发：
1. 获取武器 BP_WeaponBase 引用
2. 计算装填量：`Min(MagSize - CurrentAmmo, TotalAmmo)`
3. 更新 `CurrentAmmo` 和 `TotalAmmo`
4. 调用 `UI_MainUI.updateweaponinfo` 刷新 HUD

---

### 3.8 弹壳系统（BP_Casing）

| 组件 | 说明 |
|------|------|
| Casing (StaticMesh) | 7.62×39mm 弹壳模型 |
| ProjectileMovement | 抛物线运动（InitialSpeed + Gravity） |
| RotatingMovement | 随机旋转速率 |

**生命周期**: 开火生成 → 飞行 → 碰撞地面播放音效 → InitialLifeSpan 自动销毁

---

### 3.9 蓝图接口（BI_Fire）

| 函数 | 实现方 | 调用方 |
|------|--------|--------|
| StartWeaponFire | BP_WeaponBase | BP_FPSPlayer（IA_Fire 回调） |

设计意图：解耦玩家输入和武器开火逻辑，新增武器只需实现 `BI_Fire` 接口。

---

## 四、技术架构

```
┌──────────────────────────────────────────────────────┐
│                     关卡层                            │
│                Untitled.umap                          │
├──────────────────────────────────────────────────────┤
│                    GameMode 层                        │
│       BP_FPSGameMode → BP_FPSPlayer (DefaultPawn)    │
├──────────────┬───────────────┬───────────────────────┤
│   输入层      │    武器层      │     反馈层             │
│  Enhanced    │ BP_WeaponBase │  ShotSystem            │
│  Input       │  → BP_VIRTUS  │  ├ VFX (4 种表面)     │
│  ├ IA_Fire   │  ├ LineTrace  │  ├ SFX (4 种表面)     │
│  ├ IA_Aim    │  ├ Cast 链路   │  ├ Decals             │
│  ├ IA_Move   │  ├ 弹壳抛出    │  └ Casing (物理)     │
│  ├ IA_Look   │  └ 枪口闪光    │                      │
│  ├ IA_Reload │               │  靶场系统             │
│  ├ IA_Run    │               │  ├ BP_TargetManager   │
│  └ IA_AddAmmo│               │  ├ BP_TargetBall      │
│              │               │  ├ BP_StartButton     │
│              │               │  └ BP_AmmoLoader      │
├──────────────┴───────────────┴───────────────────────┤
│                   动画与表现层                         │
│   ABP_FPSPlayer (玩家)  │  ABP_VIRTUS (武器)          │
│   BS_FPSPlayer          │  BS_VIRTUS                 │
│   AN_Reload (通知)      │  蒙太奇系统                 │
└──────────────────────────────────────────────────────┘
```

---

## 五、关键数据表

### 5.1 武器数值

| 参数 | 值 |
|------|---|
| 武器名称 | SIG MCX-VIRTUS |
| 口径 | .300 BLK |
| 弹匣容量 | 30 发 |
| 总备弹 | 180 发 |
| 有效射程 | 10000 UE 单位 |
| 射击模式 | 全自动 |
| 换弹时间 | 约 2.5s（空仓）/ 约 2.0s（战术） |

### 5.2 移动数值

| 状态 | 速度（UE单位/秒） | FOV |
|------|-------------------|-----|
| 步行 | 265 | 100° |
| 瞄准移动 | 400 | 待定 |
| 冲刺 | 600 | 100° |

---

## 六、已知问题与扩展方向

### 6.1 已知问题

| # | 问题 | 状态 |
|---|------|------|
| 1 | BP_AmmoLoader 缺少 StaticMesh 和 Text Render 外观组件 | 待补充 |
| 2 | 仅单一武器，无武器切换系统 | 后续扩展 |
| 3 | 靶场仅射击小球，玩法单一 | v1.00 目标 |

### 6.2 版本路线图

| 版本 | 目标 |
|------|------|
| v1.00 | 实现大部分 Aimlab 功能（多模式训练、数据统计、难度递进） |
| v2.00 | 滑铲 + 蹲下 + COD 战役模式新手教程白盒 |
| 后续 | 武器扩展（手枪 / 冲锋枪 / 霰弹枪）· 敌人 AI · 后处理（瞄准景深 / 伤害血丝 / 呼吸晃动） |
