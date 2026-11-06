# Kigurumi Digital Eye System

面向 Kigurumi 头壳的通用数字眼部控制平台。

第一代**目标**以左右各一块约 7 英寸真正柔性彩色 E-Ink 为主，**该最终屏的料号、驱动/波形授权与批量供货尚未验证**，同时从软件和协议层兼容未来 **OLED / LCD**。系统规划包含手机 App、左右独立数字眼片、Head Fit 校准、本地素材/动画、UWB 无光学追踪、2D/3D 预览和通用 Display Adapter。

> **2026-11-06 可行性复核状态：Conditional Go（有条件推进）。** RW612 / SR250 属于优先验证路线，不等于最终器件已锁定；常见 7.3 英寸 Spectra 6 是玻璃基板，仅可作为开发代理。首要 G0 是最终柔性彩色屏的书面 RFQ、驱动使用权与样片验证；SR250 真实头壳 AoA 台架与之并行。参见下方深度研究、[ADR-0001](docs/decisions/ADR-0001-risk-gated-v1.md) 和 P0 验证文档。

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

- Display：约 7 英寸**真正柔性**彩色 E-Ink × 2（**供应链/驱动权限未验证；玻璃 Spectra 6 仅代理**）
- Main MCU：NXP RW612（优先候选；用 FRDM-RW612 先验证）
- UWB AoA：NXP Trimension SR250（优先候选；真实头壳内 AoA 待验证）
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

正式进入 **G0/P0：柔性彩色 EPD 供应链和样屏验证**；**UWB SR250+Beacon 实头壳 PoC 并行开展**。G0 未通过不得将玻璃代理屏当作最终方案，也不应设计量产 Final PCB。

P0 的目标不是开发完整 App，而是先确认候选屏幕的真实型号、柔性基板、驱动资料、FPC/接口、刷新稳定性、BUSY/时序和实际机械特性。

## 文档

- [V1.0 详细技术方案](docs/Kigurumi_智能数字眼片系统_V1.0_详细技术方案.md)
- [V1.0 硬件推荐选型](docs/Kigurumi_智能数字眼片系统_V1.0_硬件推荐选型.md)

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
G0  真柔性彩色屏 RFQ/样片验证   +   UWB 台架并行风险验证
 ↓
P0  屏幕驱动验证（必要时使用玻璃代理）
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
P8  STATIC/SEQUENCE Animation Engine
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

做到 P6 可获得**功能性双眼操作 MVP**；若使用玻璃代理屏，只是技术开发原型，不能宣称满足“柔性彩色眼片”最终要求。UWB 追踪预览需另通过 G4；3D 与 OLED/LCD 视频属于后续版本。

---

V1.0 详细方案保留为需求/架构历史基线；**2026-11-06 深度研究与 ADR-0001 负责解释其中尚未验证的硬件/成本假设与 V1 范围收敛**。后续新增需求应优先检查是否破坏 `EyeState`、`Head Profile`、`Display Adapter` 以及 `Control Plane / Display Plane` 的分层。