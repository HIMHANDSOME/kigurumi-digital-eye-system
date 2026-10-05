# Kigurumi Digital Eye System

面向 Kigurumi 头壳的通用数字眼部控制平台。

第一代硬件以**左右各一块约 7 英寸柔性彩色 E-Ink**为主，同时从软件和协议层兼容未来 **OLED / LCD**。系统规划包含手机 App、左右独立数字眼片、Head Fit 校准、本地素材/动画、UWB 无光学追踪、2D/3D 预览和通用 Display Adapter。

## 项目核心原则

- **E-Ink：**所有实体变化必须经过手机端明确 `Send/Commit`。
- **OLED/LCD：**保留 `REALTIME` 实时输出和实时追踪能力。
- **UWB：**采用射频定位/3D AoA，不使用摄像头、红外或其他光学追踪结构。
- **本地媒体：**大图片、动作序列、动画、视频优先预存头壳端；日常无线链路只发送控制命令和状态。
- **Head Fit：**每个头壳保存独立 `Head Profile`，支持左右眼位置、缩放、旋转、相对关系、眼眶 Mask 和逻辑原点校准。
- **统一 EyeState：**2D Preview、后续 3D Preview 和实体显示共享同一状态模型。
- **左右眼独立：**底层驱动、状态和故障处理均独立。
- **显示解耦：**Control Plane 与 Display Plane 分离，通过 `Display Adapter` 适配 E-Ink / OLED / LCD。

## 当前硬件优先路线

- Display：约 7 英寸柔性彩色 E-Ink × 2
- Main MCU：NXP RW612（当前优先）
- UWB AoA：NXP Trimension SR250（当前优先）
- Camera Beacon：SR040 类 UWB Tag 先行验证，最终型号待原型测试
- Storage：microSD
- IMU：6 轴
- Control：手机 App

> 电源电压、总功率、电池容量、DC/DC 规格等暂不冻结，必须基于最终显示屏和整机实测功耗确定。

## 输出模式

```text
PREVIEW_ONLY
MANUAL_COMMIT   # E-Ink 默认
REALTIME        # OLED/LCD
```

## 当前开发阶段

正式进入 **P0：约 7 英寸柔性彩色电子纸具体型号确认与驱动验证**。

P0 的目标不是开发完整 App，而是先确认候选屏幕的真实型号、柔性基板、驱动资料、FPC/接口、刷新稳定性、BUSY/时序和实际机械特性。

## 文档

完整架构基线：

- [`docs/Kigurumi_智能数字眼片系统_V1.0_详细技术方案.md`](docs/Kigurumi_智能数字眼片系统_V1.0_详细技术方案.md)

## 仓库结构

```text
kigurumi-digital-eye-system/
├── README.md
├── docs/          # 总体方案、协议、测试与设计文档
├── firmware/      # 头壳端固件
├── mobile-app/    # 手机 App
├── hardware/      # PCB、原理图、BOM、硬件验证
├── mechanical/    # Eye Module、支架、头壳机械集成
├── assets/        # 测试素材与 Character Pack
└── .gitignore
```

## 开发主线

```text
P0  屏幕验证
 ↓
P1  单眼本地显示
 ↓
P2  双眼独立
 ↓
P3  EyeState / State Engine
 ↓
P4  App Preview + Send
 ↓
P5  Head Fit
 ↓
P6  2D Editor            ← 第一阶段 MVP
 ↓
P7  Asset System
 ↓
P8  Animation Engine
 ↓
P9  UWB 基础链路
 ↓
P10 Tracking Engine
 ↓
P11 Tracking Preview
 ↓
P12 Tracking + Animation
 ↓
P13 3D Head Preview
 ↓
P14 Custom Head Model
 ↓
P15 OLED/LCD 验证
 ↓
P16 Final PCB
 ↓
P17 Mechanical Integration
 ↓
P18 System Acceptance
```

## MVP

第一阶段可用产品定义为：

> **两块柔性彩色 E-Ink + 手机 App + Head Fit + 2D Preview + 表情/视线调整 + Send/Commit + 本地素材播放。**

做到 P6 后，即使 UWB、3D 和 OLED/LCD 尚未实现，也应已经能够用于实际 Kigurumi 拍摄。

---

V1.0 详细方案是当前项目的统一架构基线。后续新增需求应优先检查是否破坏 `EyeState`、`Head Profile`、`Display Adapter` 以及 `Control Plane / Display Plane` 的分层。