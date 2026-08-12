# 仿COD UE FPS 射击游戏 Demo · 开发日志

> **项目名称**: 仿COD UE FPS 射击游戏 Demo
> **引擎版本**: Unreal Engine 5.8
> **当前版本**: v0.42

---

## v0.1 — 3C 开发（角色、相机、操控）

### 项目启动

- 创建 UE 5.8 纯蓝图 FPS 项目，项目路径 `D:\ueproject\FPS_Game`
- 导入 XL_FPSPack 资产包，获取玩家手臂模型、武器模型、动画、音效资源
- 创建 BP_FPSPlayer 玩家角色蓝图，配置 Camera + CharacterMesh + CharacterMovement 组件

### 玩家角色系统

- 实现第一人称视角控制（AddControllerYawInput + AddControllerPitchInput）
- 创建 E_PlayerState 枚举（Idle / Walk / Sprint / ADS）
- 实现移动速度切换：步行 265 / 瞄准 400 / 冲刺 600
- 实现 FOV 平滑过渡：FInterpTo 在默认 FOV 和瞄准 FOV 之间插值
- 配置鼠标灵敏度控制

---

## v0.2 — 射击系统开发

### 输入系统

- 创建 6 个 Enhanced Input Action（IA_Fire / IA_Aim / IA_Move / IA_Look / IA_Reload / IA_Run）
- 创建 IMC_FPSinput 输入映射上下文，绑定按键
- 在 BP_FPSPlayer BeginPlay 中挂载输入映射

### 武器系统

- 创建 BP_WeaponBase 武器基类，实现 BI_Fire 接口
- 定义核心变量：BaseDamage / CurrentAmmo / MagSize / TotalAmmo / FireRate
- 创建 BP_VIRTUS 突击步枪子类，装配 6 个配件
- 实现开火逻辑：LineTraceSingle → BreakHitResult → 物理材质判定 → VFX/SFX/弹孔
- 实现 Hitscan 即时命中检测，Visibility 通道
- 实现弹壳抛出系统（BP_Casing）：ProjectileMovement + RotatingMovement + 自动销毁
- 实现枪口闪光特效

### 动画系统

- 创建 ABP_FPSPlayer 玩家动画蓝图，5 状态 BlendSpace
- 创建 ABP_VIRTUS 武器动画蓝图，3 状态 BlendSpace
- 实现 AN_Reload 换弹 AnimNotify：计算装填量 → 更新弹药 → 刷新 HUD

### HUD 系统

- 创建 UI_MainUI 蓝图
- 实现弹药显示：当前弹药 / 总备弹
- 实现 updateweaponinfo 函数：GetOwningPlayer → GetPawn → Cast → Get CurrentWeapon → Cast → Ammo → FormatText → SetText

### 弹药系统修复

- 修复 IA_Reload 未保存到磁盘导致 R 键换弹失败
- 修复 UI 弹药不显示问题：updateweaponinfo 数据获取链路未接通
- 修复 UI 仅开枪后显示问题：BeginPlay 未 Spawn 武器导致 Weapon 变量为空

### 设计文档

- 编写 FPS 设计文档 v1.0
- 部署到 GitHub Pages 作品集

---

## v0.3 — 外围系统开发

### 靶场立牌系统（后弃用）

- 创建 BP_TargetDummy 立牌蓝图：Mesh + HeadBox + BodyBox 双碰撞体
- 实现头部一枪倒、身体四枪倒逻辑
- 实现 Timeline 倒下/扶正动画（TL_falldown / TL_reset）
- 遇到立牌被击中不倒下问题：Mesh 碰撞遮挡 BoxCollision
- 诊断：Onhit 函数为 Custom Event 而非 Function，跨蓝图调用失败
- 修复：将 Onhit 改为 Function 类型，添加 HitComp 输入参数
- **最终决定弃用立牌系统，改用更简单的小球靶场**

### Text Render 提示

- 在 BP_AmmoLoader 中添加 Text Render 组件显示 "Press F to Reload"
- 解决字体缺失黑块问题：设置 Font 为 Roboto
- 解决文字不可见问题：调整 World Size / Rotation / 位置

### 材质系统

- 创建 M_LimeWall 石灰墙材质
- 实现 Noise + Lerp 双色混合颗粒质感
- 参数：Scale=80, Levels=4, 亮色(0.62,0.6,0.56) + 暗色(0.48,0.46,0.42)

---

## v0.4 — 靶场初见雏形

### 靶场小球系统

- 创建 BP_TargetManager 管理器蓝图：MaxBalls=6, SpawnRange=500（Instance Editable）
- 创建 BP_TargetBall 小球蓝图：Sphere Mesh + BlockAll 碰撞 + Hit 函数
- 创建 BP_StartButton 启动按钮蓝图：射击触发 → 调用 Manager.StartGame()

#### 遇到的问题与修复

- **小球不生成**：StartButton 的 Hit 未正确调用 Manager 的 StartGame → 修复 Get All Actors → Get(0) → StartGame 连线
- **位置固定不跟随 Manager**：SpawnOneBall 用 SpawnCenter 变量而非 GetActorLocation → 改为 GetActorLocation + RandomUnitVector × SpawnRange
- **小球不消失**：BP_WeaponBase 缺少 Cast To BP_TargetBall → 在 Cast 链路最前面加 Cast To BP_TargetBall → Hit → DestroyActor
- **SpawnRange 不可编辑**：变量未勾选 Instance Editable → 勾选后编译保存

### 零延迟补球机制

- 原方案用定时器每 0.5 秒检查补球 → 改为球被打掉时立即调 Manager.SpawnOneBall() 补球
- Hit 函数流程：SpawnOneBall → AddScore → DestroyActor(Self)

### 计分计时系统（进行中）

- BP_TargetManager 添加 TimerText / ScoreText 两个 Text Render 组件
- 添加 Score / ElapsedTime / GameStarted 变量
- 实现 StartTimer 自定义事件 + Set Timer by Event 循环计时
- 实现 AddScore 函数：Score + 1 → FormatText → SetText
- 修改 StartGame 函数：清零 → StartTimer → 生成 6 球
- 修改 BP_TargetBall Hit 函数：补球后调 AddScore

### 设计文档重构

- 将设计文档拆分为设计文档 + 开发日志两个独立文档
- 设计文档改为结论先行结构：先写已实现系统总览，再写实现流程
- 更新文档版本至 v2.0

---

## v0.41 — 补弹功能

### 弹药补给系统

- 创建 BP_AmmoLoader 弹药补给箱蓝图
- 实现射击补给箱补满弹药至 180 发
- 遇到 Cast 失败问题：Object 引脚需连 BreakHitResult 的 Hit Actor
- 修复 Cast 链路：Is Valid → 音效 → Cast To BP_AmmoLoader → Hit → Set TotalAmmo(180)
- 修复击中音效丢失：将音效/特效移到 Cast 之前，所有命中都播放

### F 键补弹

- 创建 IA_AddAmmo Input Action，绑定 F 键
- 在 BP_FPSPlayer 中实现 F 键补弹逻辑
- 修复 AddAmmo 为 Custom Event 无法跨蓝图调用问题：改为 Function
- 修复 Target 引脚连 Weapon 而非 CurrentWeapon 问题

---

## v0.42 — 增加小球（当前版本）

### 小球增加机制

- 实现射击增加小球按钮：每次射击增加 1 个小球，无上限累加
- 打掉 1 个球立即补 1 个，始终保持当前数量
- 修改 BP_StartButton 逻辑：区分"启动靶场"和"增加小球"两种触发
- 修改 BP_TargetManager：支持动态调整球数量上限
