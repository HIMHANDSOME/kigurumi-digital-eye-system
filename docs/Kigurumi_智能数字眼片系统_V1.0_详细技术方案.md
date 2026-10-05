# Kigurumi 智能数字眼片系统 V1.0 详细技术方案

> 文档状态：V1.0 基线方案  
> 项目类型：Kigurumi 头壳数字眼片 / 可穿戴显示 / 无光学追踪系统  
> 当前主显示路线：左右各 1 块约 7 英寸柔性彩色电子纸  
> 软件兼容目标：E-Ink / OLED / LCD  
> 控制终端：手机 App  
> 追踪路线：UWB 射频定位/测向，不使用任何光学结构  
> 当前硬件主控优先路线：NXP RW612 + NXP Trimension SR250  
> 文档用途：作为后续硬件、固件、App、机械和验证工作的统一架构基线。

---

# 1. 项目背景与目标

传统 Kigurumi 头壳眼片通常采用固定印刷眼片或需要手工拆换的实体眼片，存在以下问题：

1. 更换表情需要人工拆装；
2. 视频拍摄过程中无法方便改变眼睛表情；
3. 难以快速改变视线方向；
4. 无法让角色稳定地“看向摄影机”；
5. 不同头壳的眼眶尺寸、屏幕安装位置、左右眼相对位置均不同，同一套数字素材无法直接套用；
6. 固定眼片无法实现循环动作、眨眼、特殊动画等动态效果；
7. 如果第一版软件被 E-Ink 的低刷新特性写死，未来改用 OLED/LCD 时会发生大规模重构。

本项目目标不是简单地“用屏幕替换眼片”，而是建立一套：

> **面向 Kigurumi 头壳的通用数字眼部控制、预览、素材、动画、校准、追踪与显示平台。**

第一代实体硬件以柔性彩色电子纸为主，同时从第一版软件架构起保留 OLED/LCD 的高帧率实时显示能力。

---

# 2. V1.0 产品边界

系统由三大部分组成。

## 2.1 头壳端

负责：

- 左眼显示；
- 右眼显示；
- 左右眼独立刷新与故障隔离；
- 本地素材存储；
- 本地静态素材读取；
- 本地动作序列与循环动画；
- UWB 追踪；
- 6 轴 IMU；
- BLE 控制与遥测；
- Wi-Fi 素材同步、OTA 和大日志；
- 设备状态机；
- EyeState 解析；
- Asset Manager；
- Animation Engine；
- Display Adapter；
- Device Capability 上报；
- 手机断开后的当前任务继续执行。

## 2.2 手机 App

负责：

- 设备连接；
- 实时 2D 眼片预览；
- 后续 3D 头壳预览；
- 表情选择；
- 视线调整；
- 左右眼独立微调；
- Head Fit 头壳适配；
- 眼睛大小、位置、X/Y 平移、缩放、旋转调整；
- 设置逻辑原点；
- UWB 追踪实时预览；
- E-Ink 的 Preview → Send/Commit；
- OLED/LCD 的 Realtime 模式；
- Character Pack / Asset / Animation 管理；
- Head Profile / Display Profile / Tracking Profile 管理；
- Camera Beacon 管理；
- OTA、诊断与日志查看。

## 2.3 Camera Beacon

安装于摄影设备附近，作为 UWB 射频目标。其职责仅包括：

- 提供可识别 Beacon ID；
- 被头壳端 UWB 接收器测向/测距；
- 可选 BLE 配置、配对和升级；
- 自带独立供电。

不包含：

- 摄像头；
- 红外；
- 光敏器件；
- 镜头；
- GPS；
- 视觉识别。

---

# 3. 已冻结的显示需求

## 3.1 当前 E-Ink 硬件目标

- 左右眼各 1 块显示屏；
- 当前目标约 7 英寸；
- 彩色；
- 柔性/塑料基板优先；
- 中国大陆可采购；
- 左右眼独立驱动；
- 左右眼独立刷新；
- 矩形屏可隐藏在眼眶之后；
- 屏幕自身不需要产生高光；
- 外层透明/胶层承担视觉高光与反射；
- 柔性屏尽量平装或轻微曲率安装，不强行球面弯折；
- 屏幕后必须有结构支撑，禁止 FPC/面板悬空受力。

## 3.2 刷新时间

刷新速度不设硬性上限。

当前标准是：

> **能够稳定刷新即可。**

因此屏幕实际刷新时间无论是数秒还是更长，都不直接淘汰。交互模式通过手机 Preview 与明确 Commit 解决。

## 3.3 Eye Module

左右眼建议分别构成可拆换的 Eye Module：

```text
Flexible Display
+ Driver Board
+ Support Frame
+ Connector
```

优点：

- 可维护；
- 可换屏；
- 可替换驱动板；
- 便于测试 E-Ink/OLED/LCD 不同路线；
- 一侧故障不影响另一侧结构维护。

---

# 4. 显示技术必须解耦

App 和协议不能写死为“电子纸遥控器”。

统一架构：

```text
EyeState
   │
   ▼
Output Manager
   │
   ▼
Display Adapter
 ┌─┼──────────┐
 ▼ ▼          ▼
E-Ink       OLED/LCD
```

核心逻辑只处理“眼睛是什么状态”，由 Display Adapter 决定如何在具体屏幕上实现。

---

# 5. 三种输出模式

## 5.1 PREVIEW_ONLY

所有操作只改变手机预览：

```text
User / Tracking
      ↓
PreviewState
      ↓
2D / 3D Preview
```

实体眼睛完全不变化。

适用于：

- 编辑；
- 头壳适配；
- 表情设计；
- 3D 检查；
- 调试。

## 5.2 MANUAL_COMMIT

```text
编辑
 ↓
PreviewState
 ↓
用户点击“发送”
 ↓
Committed Snapshot
 ↓
Head Display Transaction
```

这是 E-Ink 默认模式。

## 5.3 REALTIME

```text
State 变化
  ↓
Output Manager
  ↓
实时发送/本地渲染
  ↓
OLED / LCD
```

用于：

- OLED；
- LCD；
- 高刷新显示；
- 实时 UWB 追焦；
- 连续动画。

---

# 6. E-Ink 的核心交互原则

对于电子纸：

> **每一次实体眼片动作都必须由手机端用户明确点击“发送”后才执行。**

在点击 Send 之前，下列操作全部只能影响手机 Preview：

- 表情选择；
- 视线调整；
- UWB 追踪建议；
- Head Fit；
- X/Y 平移；
- 缩放；
- 旋转；
- 左右眼独立微调；
- 选择动画/动作；
- 选择静态素材。

实体屏不会因为 PreviewState 改变而刷新。

---

# 7. 通用 EyeState

核心状态建议采用连续参数，而不是直接把系统写死为 5×5 图片编号。

```text
EyeState
{
    expression_id

    gaze_x
    gaze_y

    pupil_scale

    left_eye
    {
        offset_x
        offset_y
        scale_x
        scale_y
        rotation
        eyelid
        enabled
    }

    right_eye
    {
        offset_x
        offset_y
        scale_x
        scale_y
        rotation
        eyelid
        enabled
    }

    tracking_enabled
    tracking_offset_x
    tracking_offset_y

    output_mode
}
```

建议逻辑视线坐标：

```text
gaze_x ∈ [-1.0, +1.0]
gaze_y ∈ [-1.0, +1.0]
```

这样：

- E-Ink Adapter 负责离散量化；
- OLED/LCD 直接使用连续值；
- 手机 2D/3D Preview 直接使用连续值；
- UWB Tracking 输出不受显示技术限制。

---

# 8. 四类状态必须严格分离

## 8.1 TrackingState

来自 UWB：

```text
target_id
azimuth
elevation
range
quality
gaze_x
gaze_y
```

实时变化。

## 8.2 PreviewState

手机当前正在编辑的状态。

例如：

```text
PreviewState = Happy + Gaze(+0.42,-0.18)
```

它不代表实体已经变化。

## 8.3 CommittedState

用户点击 Send 时冻结的不可变 Snapshot。

## 8.4 DisplayedState

头壳确认已经完成显示的真实实体状态。

App 在任何时候都不能用 PreviewState 假定实体状态。

---

# 9. Commit 事务模型

每次 Send 必须产生唯一 Transaction ID。

例如：

```text
COMMIT #000132
```

头壳返回：

```text
ACK #000132
DISPLAY_STARTED #000132

LEFT: BUSY → COMPLETE
RIGHT: BUSY → COMPLETE

DISPLAY_COMPLETED #000132
```

失败：

```text
DISPLAY_FAILED #000132
```

左右眼必须分别报告状态，例如：

```text
Transaction #132
Left: COMPLETE
Right: ERROR
```

App 可以提供“仅重试右眼”。

同一 Commit 的自动有限重试可以被视为同一用户确认动作，但不得生成新的 DisplayedState 语义。

---

# 10. 刷新期间继续编辑

示例：

1. 用户发送 #001 Happy；
2. 头壳正在刷新；
3. 用户在手机上继续编辑成 Shy；
4. Shy 只改变 PreviewState；
5. #001 不会被 Shy 静默覆盖；
6. 用户必须再次明确 Send，才产生 #002。

因此：

> **手机编辑与正在执行的实体事务完全解耦。**

V1 不应实现“Latest-State-Wins 自动覆盖正在执行的 E-Ink 事务”。

---

# 11. 本地素材优先

大体积媒体提前保存于头壳本地：

- 大尺寸眼片图片；
- 不同视线状态；
- 静态表情；
- 动作序列；
- 动画帧；
- 视频；
- Character Pack；
- Display Variant。

拍摄时手机尽量只发送小型控制指令：

```text
SET_EXPRESSION
SET_GAZE
PLAY_ANIMATION
STOP_ANIMATION
SET_TRACKING_TARGET
COMMIT_STATE
```

核心原则：

> **无线负责控制，本地存储负责媒体。**

这可以：

- 减少 Wi-Fi 常开；
- 降低无线带宽占用；
- 降低延迟不确定性；
- 手机断开后继续播放；
- 为未来 OLED/LCD 流畅循环动画提供基础。

---

# 12. 本地存储

V1 强烈建议保留 microSD。

用途：

- Character Pack；
- 静态图；
- 序列；
- 动画；
- 视频；
- 日志；
- OTA 临时包。

以后可根据性能需求升级为 eMMC/NAND，但不在 V1 初期复杂化。

---

# 13. 素材类型

统一定义五类。

## 13.1 STATIC

完整静态眼片。

## 13.2 LAYERED

分层素材：

```text
Base
Iris
Pupil
Highlight
Upper Eyelid
Lower Eyelid
Effect
```

主要服务未来 OLED/LCD 与实时追踪。

## 13.3 SEQUENCE

离散动作序列，例如：

```text
Open
→ Half
→ Closed
→ Half
→ Open
```

E-Ink 也可执行。

## 13.4 FRAME_ANIMATION

连续帧动画，主要用于 OLED/LCD。

## 13.5 VIDEO

完整视频资源，用于高复杂度特效。V1 协议预留，视频解码并非第一阶段必做。

---

# 14. Expression 与 Asset 解耦

一个 Expression 不是一张图片。

```text
Expression
{
    expression_id
    base_asset
    layer_set
    default_gaze
    default_scale
    tracking_policy
    animation_policy
}
```

例如 `Happy`：

- E-Ink：可映射为预渲染静态资源；
- OLED/LCD：可映射为 Base + Eyelid + Pupil + Highlight + Realtime Gaze。

同一语义状态可以被不同 Display Adapter 以不同方式实现。

---

# 15. 动画系统

```text
Animation
{
    animation_id
    type
    duration
    loop_mode
    speed
    tracking_policy
    interrupt_policy
    left_eye
    right_eye
}
```

## 15.1 Loop Mode

必须支持：

```text
ONCE
LOOP
PING_PONG
HOLD_LAST
```

## 15.2 Interrupt Policy

建议支持：

```text
IMMEDIATE
FINISH_CURRENT
FINISH_CYCLE
NON_INTERRUPTIBLE
```

## 15.3 左右眼独立

左右动画底层完全独立，但 App 可提供 `Link Eyes` 作为用户界面便利功能。

例如：

```text
Left = Wink
Right = Normal
```

---

# 16. 动画与 UWB 追踪

每个 Expression/Animation 指定 Tracking Policy：

## FOLLOW

动画过程中继续追踪。

适合：Normal、Happy、Blink。

## HOLD

动作开始时锁定当前视线。

## IGNORE

完全忽略 Tracking。

适合：Star Eyes、Loading、无瞳孔特效、固定视频。

建议默认优先级：

```text
Safety / System
>
Manual Override
>
Animation
>
Tracking
>
Expression Default
```

---

# 17. E-Ink 动作同样需要 Send

在 MANUAL_COMMIT 模式：

1. 用户在手机选择 Blink；
2. App 先本地预览；
3. 用户点击 Send/Execute；
4. 头壳才执行本地 Sequence。

不能因为用户只是选中了某动作就立即让 E-Ink 刷新。

OLED/LCD 的 REALTIME 模式则可根据 Output Policy 立即执行。

---

# 18. Head Fit：不同头壳的适配层

每一个实体头壳的眼眶、屏幕安装位置、可见区域都可能不同，因此必须引入：

> **Head Profile / Head Fit Profile**

Head Fit 不是某一个表情的属性，而是素材到实体屏幕之间的固定校准层。

数据链：

```text
Logical Eye Asset
      ↓
Expression / Animation
      ↓
Gaze / Tracking
      ↓
Head Fit Transform
      ↓
Display-space Eye
      ↓
Display Adapter
```

---

# 19. Head Profile 建议结构

```text
HeadProfile
{
    head_id
    name

    display_profile_left
    display_profile_right

    global
    {
        offset_x
        offset_y
        scale_x
        scale_y
        rotation
    }

    left_eye
    {
        origin_x
        origin_y
        offset_x
        offset_y
        scale_x
        scale_y
        rotation
    }

    right_eye
    {
        origin_x
        origin_y
        offset_x
        offset_y
        scale_x
        scale_y
        rotation
    }

    left_socket_mask
    right_socket_mask

    tracking_profile
}
```

“瞳距”不作为特殊产品参数冻结，它只是左右眼相对 Transform 的一种结果。

---

# 20. Head Fit App 功能

## 20.1 双眼整体

- Global X；
- Global Y；
- Global Scale；
- Global Rotation；
- 左右眼相对位置。

## 20.2 左眼独立

- X；
- Y；
- Scale X；
- Scale Y；
- Rotation。

## 20.3 右眼独立

同上。

## 20.4 Eye Socket Mask

2D Preview 必须支持：

```text
Display Area
+ Eye Socket Mask
+ Eye Texture
```

这样用户看到的是“装进眼眶后的可见效果”，而不仅是矩形屏幕。

---

# 21. 设置原点

适配流程：

```text
实际屏幕安装
↓
调整眼睛位置/大小/旋转
↓
左右眼精调
↓
确认眼眶边界
↓
Set Origin
```

设置后 App 可显示：

```text
X = 0
Y = 0
```

但内部继续保存真实 Calibration Transform。

运行时微调始终是：

```text
Calibration Origin
+
Runtime Offset
```

---

# 22. Calibration 与 Runtime Offset 必须分开

## Calibration Offset

属于具体头壳和屏幕安装，长期保存。

## Runtime Offset

属于当前拍摄、当前表情或用户临时微调。

最终显示位置：

```text
Head Calibration
+
Expression Transform
+
Tracking Offset
+
Manual Runtime Offset
=
Final Eye State
```

---

# 23. 2D Preview

V1 必须实现。

功能至少包括：

- 左右眼同时预览；
- 左右眼单独预览；
- Eye Socket Mask；
- Expression；
- Gaze；
- X/Y 平移；
- Scale；
- Rotation；
- Link/Unlink；
- Undo/Redo；
- Restore Displayed State；
- Restore Head Origin；
- Unsaved/Not Sent 提示；
- Send Both；
- Send Left；
- Send Right；
- Display transaction status。

---

# 24. 3D Head Preview

作为后续正式功能，架构从 V1 起必须预留。

```text
EyeState
+
Head Profile
+
3D Head Model
+
Eye Textures
↓
Phone GPU
↓
3D Preview
```

支持：

- Orbit；
- Zoom；
- Front；
- Left/Right View；
- Up/Down View；
- Camera View。

自定义模型建议采用：

```text
GLB / glTF
```

需要绑定：

- Left Eye Surface；
- Right Eye Surface；
- UV Mapping；
- Eye Transform；
- Camera Reference。

2D 和 3D 调整必须操作同一套 Head Profile 数据，而不是各自保存一套校准。

---

# 25. 3D Camera View

UWB 得到：

```text
Azimuth
Elevation
Range
```

未来 App 在 3D 场景中据此放置 Virtual Camera，从而预览：

> 当前真实摄影师从该方向看到的头壳和眼神效果。

这是 3D Preview 的重要实用目标。

---

# 26. 禁止光学追踪

系统明确不使用：

- 摄像头；
- 视觉 AI；
- 镜头识别；
- 红外 Beacon；
- 光敏阵列；
- 透镜；
- 任何光学结构。

追踪采用 RF / 惯性方案。

---

# 27. UWB 主路线

当前头壳优先：

> **NXP Trimension SR250 3D AoA**

目标数据：

```text
Azimuth
Elevation
Range
Quality / Validity
```

当前 Camera Beacon 优先验证：

> **NXP SR040 类 UWB Tag**

但 Beacon 最终 SoC 在真实互操作与 AoA 链路验证后冻结。

第一阶段优先采用开发板/参考硬件验证，避免过早进行自定义 UWB RF PCB。

---

# 28. Camera Beacon

每个 Beacon 具有唯一 `Beacon_ID`。

V1 只主动跟踪一个选中的目标：

```text
Active Target = CAM001
```

不做“自动跟踪最近 Beacon”。

安装方式应通用化，预留：

- 相机冷靴；
- 1/4"-20；
- 手机夹；
- MagSafe；
- GoPro/小三脚架。

Beacon 到镜头光轴存在空间偏移，协议预留 `Beacon-to-Lens Offset`。第一版可先提供水平/垂直修正，后续扩展 XYZ。

---

# 29. Tracking Engine

建议链路：

```text
Raw UWB
↓
Quality Filter
↓
Set Forward / Zero Calibration
↓
Range Check
↓
Azimuth/Elevation Mapping
↓
Dead Zone
↓
Hysteresis
↓
Stable Detection
↓
Manual Offset
↓
TrackingState
```

需要：

- Set Forward；
- Dead Zone；
- Hysteresis；
- Stable Time；
- 有效 Range；
- Tracking sensitivity；
- Beacon-to-Lens Offset；
- Target ID。

目标丢失时默认：

> 保持最后有效视线并显示 LOST，不自动跳回中心。

---

# 30. E-Ink 与 UWB

在 E-Ink / MANUAL_COMMIT：

```text
Beacon
↓
SR250
↓
Tracking Engine
↓
TrackingState
↓ BLE Telemetry
Phone Preview
↓
用户确认
↓ Send
E-Ink
```

UWB 可以实时运行，但**绝不直接触发实体电子纸刷新**。

---

# 31. OLED/LCD 与 UWB

在 REALTIME：

```text
UWB
↓
TrackingState
↓
Resolved EyeState
↓
Realtime Output
↓
OLED / LCD
```

可实现真正连续追焦。

---

# 32. E-Ink 离散视线

第一版建议默认 5×5，共 25 个离散 Gaze 状态：

```text
X,Y ∈ {-2,-1,0,+1,+2}
```

注意：

- 这只是 E-Ink Adapter 的默认量化策略；
- 核心 EyeState 仍使用连续值；
- 后续可改 3×3、7×7 等，不改变协议上层。

---

# 33. IMU

头壳建议配置 6 轴 IMU（加速度计 + 陀螺仪）。

V1 作用：

- 识别头部快速运动；
- 运动/稳定性遥测；
- 日志；
- 为后续预测补偿预留。

V1 不要求复杂 EKF，也不依赖磁力计。

---

# 34. 主控路线

当前优先：

> **NXP RW612 + SR250**

RW612 主要属于 Control Plane，负责：

- BLE；
- Wi-Fi；
- UWB Host；
- State Engine；
- Tracking；
- Asset Control；
- Animation Control；
- Storage；
- E-Ink 控制。

未来高分辨率 OLED/LCD 如果需要高帧率渲染/视频解码，可增加独立 Display Processor，而不是推翻 Control Plane。

---

# 35. Control Plane 与 Display Plane

## Control Plane

负责：

- BLE；
- Wi-Fi；
- UWB；
- IMU；
- Tracking；
- EyeState；
- Asset ID；
- Animation Commands；
- Device Capability；
- Profiles。

## Display Plane

负责：

- E-Ink 刷新；
- OLED/LCD Frame Output；
- Layer Composition；
- Frame Sequence；
- Video Decode；
- Frame Buffer。

---

# 36. Device Capability

设备连接 App 后先上报能力，而不是由 App 猜测。

```text
DeviceCapabilities
{
    display_type
    realtime_supported
    manual_commit_supported
    independent_eyes
    tracking_supported
    local_render_supported
    max_fps
    gaze_resolution_x
    gaze_resolution_y
}
```

E-Ink 示例：

```text
display_type = EINK
realtime_supported = false
manual_commit_supported = true
independent_eyes = true
gaze_resolution_x = 5
gaze_resolution_y = 5
```

OLED 示例：

```text
display_type = OLED
realtime_supported = true
manual_commit_supported = true
max_fps = 60
```

---

# 37. Display Profile

不同屏幕使用独立 Display Profile：

```text
DisplayProfile
{
    profile_id
    width
    height
    color_format
    palette
    rotation
    refresh_type
    driver_type
}
```

原始 Character/Expression 语义不能与某个具体屏幕分辨率硬绑定。

---

# 38. Display Adapter

至少定义：

- `EInkAdapter`；
- `OLEDAdapter`；
- `LCDAdapter`。

EInkAdapter：

- 连续 Gaze → 离散量化；
- Asset Variant 选择；
- Manual Commit；
- 左右 Worker；
- BUSY/刷新状态管理。

OLED/LCD Adapter：

- 连续 Gaze；
- Realtime；
- Layer Rendering；
- Animation Frame；
- 高帧率输出。

---

# 39. 手机与头壳通信

## 39.1 BLE

日常保持：

- Control；
- Telemetry；
- Commit；
- Device State；
- Tracking State；
- Animation Control。

## 39.2 Wi-Fi

仅按需开启：

- 大型 Asset；
- Character Pack；
- OTA；
- 大日志；
- 视频/帧序列同步。

正常拍摄可：

```text
Wi-Fi OFF
BLE ON
UWB ON
```

---

# 40. BLE 通道建议

## Control Channel

低频、可靠、需要 ACK：

```text
COMMIT
SET_MODE
SET_TARGET
PLAY_ANIMATION
STOP_ANIMATION
```

## Telemetry Channel

高频、允许偶发丢包：

```text
UWB azimuth/elevation/range/quality
IMU
Display Busy
Battery/Power status future
```

Tracking Telemetry 不得被解释为 Display Command。

---

# 41. 协议使用语义命令

不要发送 UI 事件：

```text
USER_TAPPED_HAPPY
SWIPE_LEFT
```

应发送：

```text
SET_EXPRESSION
SET_GAZE
SET_OUTPUT_MODE
SET_TRACKING_TARGET
PLAY_ANIMATION
STOP_ANIMATION
COMMIT_STATE
```

这样未来手机、PC 或专用控制器都可复用协议。

---

# 42. 核心命令建议

## Device

```text
GET_DEVICE_INFO
GET_CAPABILITIES
GET_STATUS
REBOOT
```

## Display

```text
GET_DISPLAY_STATE
GET_DISPLAY_BUSY
RESET_LEFT_DISPLAY
RESET_RIGHT_DISPLAY
```

## State

```text
SET_EXPRESSION
SET_GAZE
SET_REALTIME_STATE
SET_OUTPUT_MODE
COMMIT_STATE
```

## Tracking

```text
SET_TRACKING_ENABLED
SET_TRACKING_TARGET
SET_TRACKING_PROFILE
GET_TRACKING_STATE
```

## Animation

```text
PLAY_ANIMATION
STOP_ANIMATION
PAUSE_ANIMATION
RESUME_ANIMATION
SET_ANIMATION_SPEED
SET_LOOP_MODE
SET_TRACKING_POLICY
```

## Assets

```text
LIST_ASSETS
UPLOAD_ASSET
DELETE_ASSET
SET_ACTIVE_CHARACTER
```

## System

```text
GET_LOG
GET_STORAGE_INFO
OTA_UPDATE
```

---

# 43. Character Pack

建议目录：

```text
CharacterPack/
├── manifest.json
├── static/
├── layers/
├── sequences/
├── animations/
├── videos/
├── thumbnails/
└── profiles/
```

Manifest 至少记录：

- Character ID；
- Version；
- Expressions；
- Animations；
- Asset IDs；
- Dependencies；
- Display Variants；
- 默认 Profile。

---

# 44. Asset ID

运行时协议不使用文件路径作为业务标识。

例如：

```text
Expression 0x0102 = Happy
Animation 0x0201 = Blink
Asset 0x1007 = Happy Left Base
```

手机只发送 ID/参数，头壳通过 Manifest/索引解析实际文件。

---

# 45. 素材同步

```text
Phone Asset Library
↕ Wi-Fi
Head Asset Library
```

支持：

- Missing Check；
- Sync Missing；
- Version Compare；
- Remove；
- Device Variant Build；
- Content Hash/CRC 校验。

拍摄现场优先关闭 Wi-Fi，只保留本地媒体与 BLE 控制。

---

# 46. OLED/LCD 动态资源优先级

建议：

```text
参数化动画
>
帧序列
>
压缩视频
```

原因：完整视频会把瞳孔位置等信息烘焙死，不利于同时叠加 UWB Gaze。

未来 OLED/LCD 更推荐：

```text
Base Expression Animation
+
Realtime Pupil/Gaze
+
Eyelid
+
Highlight/Effect
=
Final Eye
```

---

# 47. 2D / 3D / 实体显示必须共享状态

```text
Resolved EyeState
├── 2D Renderer
├── 3D Renderer
└── Physical Display
```

不能维护三套彼此独立的表情/视线逻辑。

---

# 48. App 页面结构

建议：

```text
Kigu Eye Controller

├── Remote
├── Preview
├── Expressions
├── Animations
├── Head Fit
├── Tracking
├── Assets
├── Profiles
└── Device
```

## Remote

拍摄现场主页面：

- Head connection；
- Beacon status；
- Active Target；
- Output Mode；
- Preview；
- DisplayedState；
- Expression shortcuts；
- Gaze joystick / grid；
- Tracking；
- Manual Offset；
- Link Eyes；
- Send Both/Left/Right；
- Refresh progress。

## Preview

- 2D；
- 3D future；
- Camera View future。

## Head Fit

- Global；
- Left Eye；
- Right Eye；
- Socket Mask；
- Set Origin。

## Tracking

- Target Beacon；
- Set Forward；
- Dead Zone；
- Hysteresis；
- Stable Time；
- Offset；
- Quality；
- Lost state。

## Device

- Display state；
- Storage；
- Firmware；
- Diagnostics；
- Logs。

---

# 49. E-Ink App 状态显示

必须明确显示 Preview 与实体不同：

```text
Preview:
Happy / Right-Up

Head:
Normal / Center

● Not Sent

[ SEND ]
```

刷新中显示：

```text
Transaction #132
Left: Complete
Right: Busy
```

用户不能因为 App Preview 改变而误以为实体已改变。

---

# 50. Head Fit 校准向导

```text
Create Head Profile
↓
Select Display Profile
↓
Show Calibration Eye
↓
Adjust Global X/Y
↓
Adjust Size
↓
Adjust Relative Left/Right Position
↓
Adjust Left Eye
↓
Adjust Right Eye
↓
Rotation / Non-uniform Scale
↓
Define Socket Mask
↓
Set Origin
↓
Save Head Profile
```

---

# 51. UWB 校准向导

```text
Select Beacon
↓
Place Camera in Front
↓
Set Forward
↓
Check Left/Right
↓
Check Up/Down
↓
Configure Range
↓
Dead Zone
↓
Hysteresis
↓
Stable Time
↓
Save Tracking Profile
```

---

# 52. 头壳固件模块

建议：

```text
System Manager

├── BLE Service
├── Wi-Fi Service
├── UWB Service
├── IMU Service
├── Tracking Engine
├── State Engine
├── Output Manager
├── Expression Engine
├── Asset Manager
├── Animation Engine
├── Storage Manager
├── Display Adapter
│   ├── E-Ink
│   ├── OLED future
│   └── LCD future
├── Left Display Worker
├── Right Display Worker
└── Diagnostics
```

---

# 53. 左右显示 Worker

E-Ink 左右眼分别维护：

```text
LeftCommittedState
LeftDisplayedState

RightCommittedState
RightDisplayedState
```

只有 Commit Transaction 可以触发实体 Worker。

PreviewState 不能直接进入 E-Ink Worker。

---

# 54. 手机断开行为

## MANUAL_COMMIT

- 已经 Commit 且正在刷新的事务继续完成；
- 不因为手机掉线产生新的实体变化；
- DisplayedState 保持最后完成状态。

## REALTIME

根据设备能力决定：

- 若手机是实时状态源，断线后保持最后状态；
- 若头壳本地 UWB + Renderer 能自主运行，可继续跟踪。

---

# 55. UWB/主板机械布局

SR250 AoA 天线阵列优先放：

- 头顶；
- 前顶部；
- RF 净空良好的位置。

避免邻近：

- 大电池；
- DC/DC 电感；
- 大面积金属；
- 大面积铜；
- 高噪声显示驱动。

主控、存储与电源可优先布置在后脑/较远位置。

第一版禁止直接把未经验证的 SR250 天线阵列做进最终复杂 PCB，先使用官方/参考开发硬件验证。

---

# 56. 电源原则

当前明确不冻结：

- 总功率；
- 供电电压；
- USB-C PD 档位；
- 电池容量；
- 线径；
- DC/DC 型号。

因为最终显示屏、驱动和刷新功耗尚未确定。

只冻结电源分域思想：

```text
External / System Power
↓
Power Module
├── MCU/RF Domain
├── UWB Domain
├── Left Display Domain
└── Right Display Domain
```

左右显示供电建议独立可控。

最终 Power Budget 必须以实测/数据手册为依据。

---

# 57. 当前不做的 V1 功能

- Xbox 手柄主控；
- 云账号；
- 云素材商店；
- AI 自动生成表情；
- 摄像头追踪；
- 光学追踪；
- 视频时间轴编辑器；
- 房间级 UWB 世界坐标系统；
- 自动跟踪最近 Beacon；
- 复杂多传感器 EKF。

---

# 58. 开发原则：先验证高风险，再集成

禁止一开始就画最终 PCB。

第一阶段使用：

- 屏幕厂商/成熟驱动板；
- RW612 开发板；
- SR250 官方/参考开发硬件；
- UWB Tag 开发节点；
- microSD；
- 外接 IMU；
- 临时支架与线束。

在屏幕、UWB、功耗、接口、无线链路均验证后再进入最终板卡。

---

# 59. P0：柔性彩色屏验证

目标：证明候选约 7 英寸柔性彩色电子纸能够稳定驱动。

验证：

- 真实完整型号；
- 是否塑料/柔性基板；
- 可弯曲限制；
- 分辨率；
- 色彩格式；
- Driver IC；
- FPC/连接器；
- 初始化；
- 完整刷新；
- 连续多次刷新；
- 上电/断电时序；
- BUSY；
- 断电保持；
- 实际刷新时间；
- 开发资料；
- 大陆供应稳定性。

P0 不做 App、UWB、最终 PCB。

验收：

> 单块屏能够持续、可靠地显示指定图片。

---

# 60. P1：单眼本地显示

```text
microSD
↓
Asset Manager
↓
EInk Adapter
↓
Left Eye
```

至少支持：

- Normal；
- Happy；
- Closed。

目标是证明显示不依赖手机实时传大图。

---

# 61. P2：双眼独立显示

实现：

- 左右独立初始化；
- 左右分别刷新；
- 左右不同素材；
- 两侧同时工作；
- 一侧错误不拖死另一侧；
- 左右状态独立报告。

---

# 62. P3：EyeState / State Engine

正式建立：

- EyeState；
- TrackingState；
- PreviewState；
- CommittedState；
- DisplayedState；
- Output Mode；
- Display Capability。

摆脱临时的“show image 5”式控制。

---

# 63. P4：手机 App MVP

实现：

- BLE 连接；
- 2D Preview；
- Expression；
- Gaze；
- Preview/Displayed 状态分离；
- Send；
- Transaction ID；
- 左右刷新状态。

验收关键：

> 手机所有调整在点击 Send 前不得改变实体 E-Ink。

---

# 64. P5：Head Fit

实现：

- Global X/Y；
- Global Scale；
- Left X/Y；
- Right X/Y；
- Left/Right Scale；
- Relative Transform；
- Rotation；
- Set Origin；
- Head Profile。

使用两个不同安装位置的测试 Profile 验证同一套素材可以正确适配不同头壳。

---

# 65. P6：2D Editor

增加：

- Socket Mask；
- Undo/Redo；
- Link/Unlink；
- Runtime Offset；
- Restore DisplayedState；
- Restore Origin；
- 未发送提示。

**P6 作为第一阶段可用 MVP 分界点。**

---

# 66. P7：Asset System

实现：

- Asset ID；
- Character Pack；
- Manifest；
- STATIC；
- SEQUENCE；
- Phone ↔ Head Sync；
- Hash/Version。

---

# 67. P8：Animation Engine

实现：

- PLAY；
- STOP；
- PAUSE；
- RESUME；
- ONCE；
- LOOP；
- PING_PONG；
- HOLD_LAST；
- Tracking Policy；
- Interrupt Policy；
- 左右眼独立。

---

# 68. P9：UWB 基础链路

只验证：

```text
Camera Beacon
↓
SR250
↓
Range
Azimuth
Elevation
Quality
```

此阶段不控制眼片。

---

# 69. P10：Tracking Engine

实现：

- Set Forward；
- Range Filter；
- Dead Zone；
- Hysteresis；
- Stable Time；
- Manual Offset；
- Beacon-to-Lens Offset；
- Active Target。

---

# 70. P11：Tracking Preview

```text
UWB
↓
TrackingState
↓ BLE Telemetry
Phone 2D Preview
```

实体 E-Ink 保持不动，直到 Send。

验收目标：

> 手机上实时看到“如果现在发送，角色眼睛会看哪里”。

---

# 71. P12：Tracking + Animation

验证：

- FOLLOW；
- HOLD；
- IGNORE；
- 动画中手动 Override；
- 动画结束后的状态恢复。

---

# 72. P13：3D Head Preview

先采用内置通用头壳模型：

- Orbit；
- Zoom；
- Front；
- Side；
- Eye Texture Mapping；
- Camera View 基础。

---

# 73. P14：自定义 3D Head

支持：

- GLB；
- glTF；
- 左右眼 Surface 绑定；
- UV；
- Head Profile Binding；
- 2D/3D 共用校准。

---

# 74. P15：OLED/LCD 兼容验证

连接一块普通 OLED/LCD 开发屏，验证：

```text
Device Capability
↓
REALTIME
↓
EyeState
↓
Realtime Gaze
↓
OLED/LCD
```

若无需重构 App/协议，则证明显示技术解耦成功。

---

# 75. P16：最终 PCB

只有以下项目均有数据后才设计：

- 最终屏幕；
- EPD Driver；
- 实测功耗；
- RW612 接口；
- SR250 模块/天线要求；
- microSD；
- IMU；
- BLE/Wi-Fi；
- 左右显示接口；
- 线束；
- 电源域。

---

# 76. P17：机械集成

完成：

- Left Eye Module；
- Right Eye Module；
- UWB Module；
- Main Board；
- Screen Support；
- FPC Protection；
- Quick Release；
- Cable Management；
- Weight Distribution；
- 外部供电连接。

---

# 77. P18：整机验收

## 显示

- 左右独立；
- 静态表情；
- 长时间刷新可靠性；
- 单侧故障隔离；
- 本地资源读取。

## App

- Preview；
- DisplayedState；
- Send；
- Transaction；
- Head Fit；
- Set Origin；
- Profiles；
- 2D Editor。

## Animation

- Sequence；
- Loop；
- Stop；
- Interrupt；
- 左右独立。

## Tracking

- Beacon 配对；
- Target ID；
- Azimuth/Elevation/Range；
- Calibration；
- Tracking Preview；
- Lost Handling。

## 通信

- BLE 控制；
- Telemetry；
- Wi-Fi Asset Sync；
- 手机掉线恢复；
- OTA。

## 机械

- 屏幕固定；
- FPC 可靠性；
- Eye Module 可维护；
- UWB RF 净空；
- 实际佩戴；
- 拍摄效果。

---

# 78. MVP 定义

第一阶段真正可用产品：

> **两块柔性彩色 E-Ink + 手机 App + Head Fit + 2D Preview + 表情/视线调整 + Send/Commit + 本地素材播放。**

即 P6。

即使 UWB、3D 和 OLED/LCD 尚未完成，也已经可以用于实际 Kigurumi 拍摄。

---

# 79. 当前尚未冻结的项目

## 屏幕

- 最终 7 英寸柔性彩色屏型号；
- 分辨率；
- 色彩体系；
- Driver IC；
- 波形/LUT；
- BUSY；
- 局刷；
- 实际弯曲限制。

## 功耗

- 左右屏刷新功耗；
- 双屏峰值；
- RW612；
- SR250；
- Wi-Fi；
- Beacon；
- 最终输入电压；
- DC/DC；
- 电池。

## Camera Beacon

- 最终 UWB SoC；
- 电池；
- USB-C；
- BLE；
- 天线；
- 机械结构。

## OLED/LCD

- 分辨率；
- 接口；
- Display Processor；
- Frame Buffer；
- Codec；
- 是否需要 Linux SoC。

---

# 80. 第一轮采购优先级

1. 候选 7 英寸柔性彩色电子纸；
2. 对应成熟/官方驱动板；
3. FPC/连接件；
4. microSD；
5. RW612 开发板；
6. SR250 参考/开发硬件；
7. UWB Tag 开发节点；
8. 6 轴 IMU；
9. 第二块同型号屏；
10. 机械测试支架。

在 P0 之前不建议大量采购同型号屏幕。

---

# 81. 项目版本路线

```text
V0.1  单屏验证
V0.2  双屏本地显示
V0.3  App Preview + Send
V0.4  Head Fit + 2D Editor
V0.5  Asset + Sequence
V0.6  UWB Tracking Preview
V0.7  完整 Animation Engine
V0.8  3D Preview
V0.9  OLED/LCD Compatibility Test
V1.0  Final PCB + Mechanical Integration + Acceptance
```

---

# 82. 设计原则清单

后续开发必须遵守：

1. 手机负责编辑、预览和控制，不持续流式传输大媒体；
2. 大图片、动画、视频优先预存头壳；
3. E-Ink 所有实体变化必须明确 Send/Commit；
4. OLED/LCD 必须保留 Realtime；
5. UWB 始终可实时运行，但 E-Ink 不被 UWB 自动刷新；
6. 禁止任何光学追踪结构；
7. EyeState 与显示技术解耦；
8. Head Fit 与表情素材解耦；
9. Calibration 与 Runtime Offset 解耦；
10. 2D、3D、实体显示共享同一个 EyeState；
11. 左右眼从底层独立；
12. 素材使用 Asset ID，不依赖文件名；
13. Control Plane 与 Display Plane 分离；
14. Wi-Fi 只在大数据操作时开启；
15. 最终 PCB 必须晚于关键原型验证；
16. 电源参数必须来自真实硬件数据；
17. 屏幕适配统一通过 Display Profile；
18. 不同头壳统一通过 Head Profile；
19. 3D Preview 后续必须支持真实摄影机方向 Camera View；
20. 软件第一版就必须避免 E-Ink 专用死架构。

---

# 83. 当前下一步：P0

正式进入：

> **P0：约 7 英寸柔性彩色电子纸具体型号确认与驱动验证。**

屏幕候选评审时需要尽可能收集：

- 商品链接；
- 完整型号；
- 正反面照片；
- FPC 型号与针脚；
- 驱动板型号；
- 数据手册；
- 刷新演示；
- 柔性/最小弯曲说明；
- 价格；
- 中国大陆采购渠道。

确认屏幕后应产出：

1. P0 测试计划；
2. 驱动链路图；
3. 接口定义；
4. 测试固件结构；
5. 屏幕验收表；
6. 第一轮采购 BOM；
7. 初始功耗测试方案。

---

# 84. V1.0 基线结论

本项目最终定位不是“电子纸眼片”，而是：

> **Kigurumi Digital Eye Control Platform —— 面向不同头壳、不同显示技术、支持无光学追踪与本地动态素材的通用数字眼部系统。**

第一代 E-Ink 产品重点实现：

- 无需拆换实体眼片；
- 手机 2D Preview；
- 明确 Send/Commit；
- 表情切换；
- 视线切换；
- 左右眼独立；
- Head Fit；
- Set Origin；
- 本地素材；
- 本地动作序列；
- UWB Tracking Preview。

同一架构后续扩展到：

- OLED；
- LCD；
- 实时追焦；
- 实时眨眼；
- 参数化动画；
- 高帧率动画；
- 分层渲染；
- 循环特效；
- 3D Head Preview；
- UWB Camera View；
- 自定义 GLB/glTF 头壳；
- 更复杂的角色数字表情系统。

任何后续新增需求或架构修改，都应首先检查是否破坏以下四个核心边界：

- `EyeState`；
- `Head Profile`；
- `Display Adapter`；
- `Control Plane / Display Plane` 分层。
