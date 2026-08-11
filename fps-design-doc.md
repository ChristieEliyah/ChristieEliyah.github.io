# UE5 FPS 射击训练 Demo · 设计文档

> **项目名称**: FPS 射击训练靶场 Demo  
> **引擎版本**: Unreal Engine 5.8  
> **开发语言**: 纯蓝图（Blueprint）  
> **武器原型**: SIG MCX-VIRTUS 突击步枪（300 BLK）  
> **开发周期**: 个人项目  
> **文档版本**: v1.0

---

## 一、项目概述

### 1.1 项目定位

本项目是一个 **UE5 第一人称射击（FPS）训练靶场 Demo**，参考 COD 系列新手教程关卡的设计思路，构建了一套完整的 FPS 核心战斗系统——包括角色操控、武器射击、击中反馈、靶子交互、动画状态机和 HUD。

项目采用 **纯蓝图开发**，从第三方资产包（XL_FPSPack）的基础上进行系统化整合与定制，覆盖了从输入层、逻辑层到表现层的完整开发链路。

### 1.2 设计目标

| 目标 | 说明 |
|------|------|
| **射击手感** | 即时命中（Hitscan）+ 完整的击中反馈链路（VFX / SFX / 贴花 / 靶子受击反应） |
| **武器操作** | 开火 / 瞄准（ADS）/ 换弹 / 弹药管理，接近商业 FPS 的操作体验 |
| **多表面响应** | 13 种表面物理材质各自产生不同的击中特效、音效、弹孔贴花 |
| **动画融合** | 玩家动画与武器动画分层管理，BlendSpace 实现无缝移动过渡 |
| **可扩展架构** | 武器基类（BP_WeaponBase）+ 蓝图接口（BI_Fire），支持未来添加新武器 |

---

## 二、核心战斗循环

```
┌─────────────────────────────────────────────────────┐
│                    FPS 训练靶场循环                    │
│                                                       │
│    [输入]  ──→  [武器开火]  ──→  [射线命中检测]       │
│     ↑                              ↓                  │
│     │                    [命中结果拆解]                │
│     │                       ↓          ↓              │
│     │               [物理材质判定]   [命中Actor]       │
│     │                 ↓     ↓          ↓              │
│     │             [击中VFX] [SFX]   [ApplyDamage]     │
│     │                 ↓                  ↓            │
│     │             [弹孔贴花]      [靶子扣血/击倒]      │
│     │                 ↓                  ↓            │
│     └────────── [弹药UI更新]  ←  [靶子复位Timeline]   │
│                                                       │
│              [弹壳抛出 + 枪口闪光]                     │
└─────────────────────────────────────────────────────┘
```

---

## 三、系统设计

### 3.1 玩家角色系统（BP_FPSPlayer）

**继承链**: `Character` → `BP_FPSPlayer`

#### 3.1.1 角色组件

| 组件 | 类型 | 说明 |
|------|------|------|
| Camera | CameraComponent | FPS 第一人称摄像机 |
| CharacterMesh0 | SkeletalMeshComponent | 角色手臂模型（SK_Base_Skin） |
| CharacterMovement | CharacterMovementComponent | 移动行为控制 |

#### 3.1.2 核心参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| Default FOV | 90° | 默认视场角 |
| Mouse Sensitivity | 1.0 | 鼠标灵敏度 |
| Walk Speed | 265 | 步行速度 |
| ADS Speed | 400 | 瞄准移动速度 |
| Sprint Speed | 600 | 冲刺速度 |

#### 3.1.3 玩家状态（E_PlayerState 枚举）

| 状态 | 说明 | 触发条件 |
|------|------|----------|
| Idle | 待机 | 无移动输入 |
| Walk | 步行 | WASD 移动 |
| Sprint | 冲刺 | 左 Shift + 移动 |
| ADS | 瞄准 | 鼠标右键按下 |

#### 3.1.4 核心逻辑

- **FOV 平滑过渡**: 瞄准时使用 `FInterpTo` 在默认 FOV 和瞄准 FOV 之间插值
- **移动速度切换**: 根据 `E_PlayerState` 切换 `CharacterMovement.MaxWalkSpeed`
- **鼠标视角控制**: `AddControllerYawInput` + `AddControllerPitchInput`
- **武器生成**: BeginPlay 时创建 `BP_VIRTUS` 实例并附加到角色

---

### 3.2 武器系统

#### 3.2.1 武器基类（BP_WeaponBase）

**继承链**: `Actor` → `BP_WeaponBase`  
**实现接口**: `BI_Fire`

| 核心变量 | 类型 | 说明 |
|----------|------|------|
| BaseDamage | Float | 单发基础伤害 |
| CurrentAmmo | Int | 当前弹匣内弹药 |
| MagSize | Int | 弹匣容量 |
| TotalAmmo | Int | 总备弹数 |
| FireRate | Float | 射速（发/秒） |
| Barrel | SceneComponent | 枪管 Socket，射线起点 |
| BulletDecal | Class | 弹孔贴花蓝图类 |

#### 3.2.2 MCX-VIRTUS 突击步枪（BP_VIRTUS）

**继承链**: `BP_WeaponBase` → `BP_VIRTUS`

**武器配件**: 已装配至武器网格，使用 SceneComponent 定位

| 配件 | 说明 |
|------|------|
| 300-BACKOUT 弹匣 | 30 发 .300 BLK 弹匣 |
| DMR-X 枪管 | 长枪管，提升精度 |
| MCX-VIRTUS Close XSights | 机械瞄具 |
| Sig Sauer Romeo3 | 红点瞄准镜 |
| MCX Stock | 可调节枪托 |
| CQB 垂直握把 | 提升操控性 |

**动画**: 玩家侧 38 个动画 + 武器侧 37 个动画（Idle / Walk / ADS / Sprint / Fire / Reload / Draw / Holster / Slide）

**音效**: 21 个开火音频变体 + 11 个拟音音效（Draw / Holster / Reload / Sprint / ADS 过渡）

---

### 3.3 输入系统（Enhanced Input）

| 输入动作 | 按键 | 触发方式 | 功能 |
|----------|------|----------|------|
| IA_Fire | 鼠标左键 | Started | 开火 |
| IA_Aim | 鼠标右键 | Triggered | 瞄准（ADS） |
| IA_Move | W/A/S/D | Continuous | 角色移动 |
| IA_Look | Mouse2D | Continuous | 视角旋转（含轴交换 + 取反修饰器） |
| IA_Reload | R | Triggered | 换弹 |
| IA_Run | 左 Shift | Started | 冲刺 |

**输入映射**: `IMC_FPSinput` 统一管理，优先级最高，在 `BP_FPSPlayer.BeginPlay` 时挂载。

---

### 3.4 命中检测与反馈系统

#### 3.4.1 射线检测（LineTrace）

| 参数 | 值 |
|------|---|
| 起点 | 枪管 Barrel 组件的世界位置 |
| 终点 | 起点 + 摄像机前向 × 10000 |
| 通道 | Visibility / Camera |
| 输出 | HitResult（命中点 / 命中 Actor / 命中组件 / 物理材质） |

#### 3.4.2 表面材质响应表（13 种）

每发子弹命中后，根据被命中表面的物理材质（Physical Material）匹配对应的反馈：

| 表面材质 | 击中 VFX | 击中 SFX | 弹孔贴花 |
|----------|----------|----------|----------|
| 混凝土 | FX_SquibConcrete | SFX_Concrete_Cue | MI_BulletDecal_Concrete |
| 泥土 | FX_SquibDirt | SFX_Dirt_Cue | 默认弹孔 |
| 金属 | FX_SquibMetal | SFX_Metal_Cue | MI_BulletDecal_Metal |
| 木头 | FX_SquibWood | SFX_Wood_Cue | MI_BulletDecal_Wood |
| 岩石 | FX_SquibRock | SFX_Rock_Cue | MI_BulletDecal_Rock |
| 布料 | FX_SquibFabric | SFX_Fabric_Cue | MI_BulletDecal_Fabric |
| 玻璃 | FX_SquibGlass | SFX_Glass_Cue | MI_BulletDecal_Glass |
| 沙地 | FX_SquibSand | SFX_Sand_Cue | 默认弹孔 |
| 雪地 | FX_SquibSnow | SFX_Snow_Cue | 默认弹孔 |
| 水面 | FX_SquibWater | SFX_Water_Cue | — |
| 泥浆 | FX_SquibMud | SFX_Mud_Cue | 默认弹孔 |
| 树篱 | FX_SquibHedge | SFX_Hedge_Cue | 默认弹孔 |
| 血液 | Impact_Blood / FX_SquibBlood | — | — |

#### 3.4.3 伤害系统

| 参数 | 值 |
|------|---|
| 伤害类型 | ApplyDamage（GameplayStatics） |
| 伤害值 | BaseDamage（武器基类变量） |
| 伤害对象 | 命中 HitActor（如 BP_TargetDummy） |

---

### 3.5 靶子系统（BP_TargetDummy）

#### 3.5.1 组件结构

| 组件 | 说明 |
|------|------|
| Cube（身体） | 引擎基础 Cube，作为主体碰撞体 |
| head（头部网格） | 自定义头部模型 |
| headcube | 头部碰撞体 |

#### 3.5.2 核心逻辑

| 变量 | 类型 | 说明 |
|------|------|------|
| Health | Int | 当前生命值 |
| Bisdown | Bool | 是否已倒地 |
| resettime | Float | 击倒后自动复位时间 |

| 事件 | 说明 |
|------|------|
| Event AnyDamage | 接收武器 ApplyDamage，扣血 + 判断倒地 |
| ShootFallDown | Timeline 驱动倒下动画（RelativeRotation 0°→90°，随机方向） |
| TL_reset | 自动复位 Timeline（90°→0°） |

**击倒流程**: 中弹 → AnyDamage → Health ≤ 0 → TL_falldown（0.3s 倒下）→ 等待 resettime → TL_reset（0.3s 立起）

---

### 3.6 动画系统

#### 3.6.1 玩家动画蓝图（ABP_FPSPlayer）

**骨骼**: `Skeleton_Base`（XL_FPSPack 手臂骨骼）

**状态机**: 5 个动画状态，基于 `E_PlayerState` 和移动速度驱动过渡

| 状态 | 动画/混合 | 说明 |
|------|-----------|------|
| Idle/Walk | BS_FPSPlayer + Player_Idle | 2D BlendSpace 驱动待机↔八向移动 |
| ADS Idle | Player_ADS_Idle | 瞄准待机 |
| ADS Walk | Player_ADS_Walk | 瞄准移动 |
| Sprint | Player_Sprint | 冲刺动画 |

**过渡条件**: `E_PlayerState` 枚举 + Velocity.Length > 0 + ADS Socket 位置判断

#### 3.6.2 武器动画蓝图（ABP_VIRTUS）

**骨骼**: `MCX-VIRTUS_Skeleton`

| 状态 | 动画/混合 | 说明 |
|------|-----------|------|
| Idle/Walk | BS_VIRTUS + Weapon_Idle | 武器 2D BlendSpace |
| ADS | Weapon_ADS_Idle + Weapon_ADS_Walk | 武器瞄准状态 |
| Sprint | Weapon_Sprint | 武器冲刺动画 |

#### 3.6.3 换弹动画通知（AN_Reload）

`AN_Reload` 是一个自定义 AnimNotify，在换弹蒙太奇的关键帧触发，执行以下逻辑：
1. 获取武器 BP_WeaponBase 引用
2. 计算装填量：`Min(MagSize - CurrentAmmo, TotalAmmo)`
3. 更新 `CurrentAmmo` 和 `TotalAmmo`
4. 调用 `UI_MainUI.updateweaponinfo` 刷新 HUD

---

### 3.7 弹壳系统（BP_Casing）

| 组件 | 说明 |
|------|------|
| Casing (StaticMesh) | 7.62×39mm 弹壳模型（SM_7_62x39MM） |
| ProjectileMovement | 抛物线运动（InitialSpeed + Gravity） |
| RotatingMovement | 随机旋转速率 |

**生命周期**: 开火时生成 → 弹壳飞行 → 碰撞地面 → 播放 `Bullet_Impact` 音效 → `InitialLifeSpan` 到时后自动销毁

---

### 3.8 HUD 系统（UI_MainUI）

**弹药显示格式**: `当前弹药 / 总备弹`

| 函数 | 说明 |
|------|------|
| updateweaponinfo(current, total) | 武器弹药变化时由 AN_Reload / 开火逻辑调用 |

---

## 四、蓝图接口（BI_Fire）

| 函数 | 说明 | 实现方 | 调用方 |
|------|------|--------|--------|
| StartWeaponFire | 触发武器开火 | BP_WeaponBase | BP_FPSPlayer（IA_Fire 输入回调） |

设计意图：通过蓝图接口解耦玩家输入和武器开火逻辑，未来新增武器类型时只需实现 `BI_Fire` 接口即可无缝接入。

---

## 五、游戏模式（BP_FPSGameMode）

| 配置项 | 值 |
|--------|---|
| DefaultPawnClass | BP_FPSPlayer |
| HUD Class | 默认（UI 由 BP_FPSPlayer 自行创建） |

---

## 六、关卡设计

### 6.1 现有关卡

- **Untitled.umap**: 单一训练关卡，包含 141 个场景 Actor

### 6.2 靶场训练流程（规划中）

参考 COD 新手教程的 5 站训练流程（详见 `UE5射击Demo_靶场关卡设计文档.md`）：

| 站号 | 训练内容 | 关卡目标 |
|------|----------|----------|
| 第 1 站 | 基础射击 | 熟悉武器开火 + 后坐力 |
| 第 2 站 | 移动射击 | 边移动边射击移动靶 |
| 第 3 站 | 掩体切换 | 快速切掩体 + 预判射击 |
| 第 4 站 | 多目标压制 | 同时处理多个靶子 |
| 第 5 站 | 综合考核 | 时间限制 + 精度评级 |

---

## 七、技术架构总结

```
┌──────────────────────────────────────────────────┐
│                   关卡层                           │
│              Untitled.umap (141 Actors)            │
├──────────────────────────────────────────────────┤
│                   GameMode 层                      │
│     BP_FPSGameMode → BP_FPSPlayer (DefaultPawn)   │
├──────────────┬──────────────┬─────────────────────┤
│   输入层     │   武器层      │   反馈层             │
│  Enhanced    │ BP_WeaponBase│  ShotSystem (139)    │
│  Input       │  → BP_VIRTUS │  ├ VFX (13 种表面)   │
│  ├ IA_Fire   │  ├ LineTrace │  ├ SFX (13 种表面)   │
│  ├ IA_Aim    │  ├ 材质判定   │  ├ Decals (7 种)     │
│  ├ IA_Move   │  ├ 弹壳抛出   │  └ Casing (物理)    │
│  ├ IA_Look   │  └ 枪口闪光   │                     │
│  ├ IA_Reload │              │  BP_TargetDummy      │
│  └ IA_Run    │              │  ├ 生命值 + 倒地     │
│              │              │  ├ Timeline 动画     │
│              │              │  └ 自动复位          │
├──────────────┴──────────────┴─────────────────────┤
│                   动画与表现层                       │
│   ABP_FPSPlayer (玩家)  │  ABP_VIRTUS (武器)        │
│   BS_FPSPlayer          │  BS_VIRTUS               │
│   AN_Reload (通知)      │  蒙太奇系统               │
├───────────────────────────────────────────────────┤
│                    UI 层                            │
│   UI_MainUI (弹药 HUD)                             │
│   └ updateweaponinfo(current, total)               │
└───────────────────────────────────────────────────┘
```

---

## 八、关键数据表

### 8.1 武器数值

| 参数 | 值 |
|------|---|
| 武器名称 | SIG MCX-VIRTUS |
| 口径 | .300 BLK |
| 基础伤害 | BaseDamage（蓝图可调） |
| 弹匣容量 | 30 发 |
| 射速 | FireRate（蓝图可调） |
| 有效射程 | 10000 UE 单位 |
| 射击模式 | 全自动 |
| 换弹时间 | 约 2.5s（空仓）/ 约 2.0s（战术） |

### 8.2 移动数值

| 状态 | 速度（UE单位/秒） | FOV |
|------|-------------------|-----|
| 步行 | 265 | 90° |
| 瞄准移动 | 400 | 待定 |
| 冲刺 | 600 | 90° |

### 8.3 资产统计

| 类别 | 数量 |
|------|------|
| 自定义蓝图 | 24 |
| 动画文件 | 75+ |
| 音效文件 | 200+ |
| Niagara 特效 | 11 |
| 弹孔贴花材质实例 | 7 |
| 武器网格 | 12 |
| 关卡 | 1 |
| 总资产 | 933 |

---

## 九、已知问题与优化方向

### 9.1 已知问题

| # | 问题 | 状态 |
|---|------|------|
| 1 | 靶子 AnyDamage 事件待补完（射线检测有 ApplyDamage，靶子侧需加 Event AnyDamage） | 修复中 |
| 2 | 仅单一武器，无武器切换系统 | 后续扩展 |
| 3 | 无计分 / 关卡通关逻辑 | 后续扩展 |
| 4 | 靶场 5 站训练流程尚未搭建 | 规划中 |

### 9.2 后续扩展方向

| 方向 | 说明 |
|------|------|
| 武器扩展 | 添加手枪 / 冲锋枪 / 霰弹枪，利用 BP_WeaponBase 派生 |
| 敌人 AI | 添加行为树驱动的 AI 敌人（巡逻 / 发现 / 追击 / 攻击） |
| Level Sequence | 靶场开场 CG + NPC 教官语音引导 |
| 音效混合 | MetaSound 实现枪声远近感 / 室内混响 |
| 后处理 | 瞄准景深 / 伤害屏幕边缘血丝 / 呼吸晃动 |
