# Kigurumi 智能数字眼片系统 V1.0 硬件推荐选型

> 文档状态：V1.0 硬件选型基线  
> 更新时间：2026-10-05  
> 适用仓库：`HIMHANDSOME/kigurumi-digital-eye-system`  
> 配套文档：`Kigurumi_智能数字眼片系统_V1.0_详细技术方案.md`  
> 目标：为 P0～P3 原型阶段以及后续 Final PCB 提供可执行的硬件选型建议。

---

# 1. 选型原则

本项目的硬件选型不能只围绕第一代 E-Ink 眼片，而必须同时满足以下长期架构：

- 第一代：左右各一块约 7 英寸柔性彩色 E-Ink；
- 手机 App 为主要控制端；
- 头壳端本地保存大图、动作序列、动画和后续视频资源；
- UWB 实时追踪必须保留；
- 不采用摄像头、红外、光敏器件或其他光学追踪结构；
- 软件与协议后续兼容 OLED / LCD；
- Control Plane 与 Display Plane 分离；
- 左右眼从电气、驱动、状态和供电上尽量独立；
- 最终 PCB 必须晚于屏幕、UWB、功耗和接口实测。

因此本文把器件分成四类：

| 标记 | 含义 |
|---|---|
| **首选** | 当前建议直接用于原型或作为最终方案重点发展 |
| **备选** | 首选受供应、尺寸、兼容性限制时切换 |
| **暂不锁定** | 需要真实屏幕/功耗/机械结果后再决定 |
| **仅验证代理** | 可用于软件和接口开发，但不代表最终机械/显示器件 |

---

# 2. 推荐硬件总览

| 子系统 | 当前推荐 | 状态 | 说明 |
|---|---|---|---|
| 左右眼显示 | 用户实际可采购的约 7 英寸**柔性彩色 E-Ink** | **首选，但型号待确认** | 最终屏必须验证柔性基板、FPC、驱动资料和大陆供应 |
| 显示开发代理 | 7.3" E Ink Spectra 6，800×480，例如 ED2208-GCA / GDEP073E01 类 | **仅验证代理** | 可先验证彩色转换、SPI、刷新状态机；常见版本并非柔性最终屏 |
| 主控开发板 | **NXP FRDM-RW612** | **首选** | 260 MHz Cortex-M33、1.2 MB SRAM、Wi-Fi 6、BLE、SPI/I²C/UART 丰富 |
| 最终主控 | **NXP RW612** 或 RW612 模组 | **首选** | 最终裸芯片/模组形式在 PCB 阶段决定 |
| 头壳 UWB | **NXP Trimension SR250** | **首选** | 支持 3D / 360° AoA、测距，并有 2026 官方开发板和设计资料 |
| UWB 原型 | **SR250 Development Board / SR250-ARD 类官方板** | **首选** | P9-P12 前不建议自画 SR250 RF |
| 最终 UWB | **SR250 集成 3D 天线模组**（如 NXP Partner Marketplace 中 Amotech 方案） | **优先考虑** | 比自研三天线相位链路风险低 |
| Camera Beacon | **SR040 类低功耗 UWB Tag** | **首选候选，需联调确认** | 必须实测与 SR250 AoA 会话兼容性 |
| Beacon 备选 | Qorvo DWM3001C 或成熟 FiRa UWB Tag | **备选** | 集成 UWB/BLE/天线/运动传感器，原型方便，但生态不同 |
| IMU | **Bosch BMI270** | **首选** | 可穿戴低功耗，6 轴，SPI/I²C，功耗适合头壳 |
| IMU 备选 | TDK ICM-42688-P / Bosch BMI323 / ST LSM6DSO32X | **备选** | 性能、供货、驱动生态不同 |
| 本地存储 | **microSD / microSDHC / microSDXC** | **首选** | 大图、Character Pack、动画、视频最方便 |
| 外部 Flash | QSPI NOR，容量按固件和缓存确定 | **建议** | 固件、配置、少量缓存；不替代 microSD 媒体库 |
| 左右眼驱动板 | 屏厂原装/成熟驱动板先行 | **首选原型路线** | 不在 P0-P2 直接自研高压 EPD 电源/波形链路 |
| 电源 | 独立 MCU/UWB/Left EPD/Right EPD 电源域 | **冻结原则** | 具体电压与功率不锁定 |
| 3.3 V 电源候选 | TI TPS63070 类宽输入 Buck-Boost | **候选** | 适合输入可能高于或低于 3.3 V 的情况 |
| 仅降压候选 | TI TPS62130A / 更新型 TPS62903 类 | **候选** | 仅在系统输入始终高于 3.3 V 时考虑 |

---

# 3. 显示屏选型

## 3.1 最终目标屏

最终屏幕必须满足：

1. 约 7 英寸；
2. 彩色；
3. **柔性基板或明确可弯曲结构**；
4. 左右各一块；
5. 中国大陆可以实际采购；
6. 有 datasheet；
7. 有 FPC pinout；
8. 有驱动时序/控制器资料；
9. 有开发板或成熟驱动板优先；
10. 可以可靠重复刷新；
11. 刷新时间不限，只要稳定即可。

### 重要风险

市面上常见“7.3 英寸彩色电子纸”并不等于“7.3 英寸柔性彩色电子纸”。

当前公开常见 7.3" Spectra 6 产品（例如 ED2208-GCA、GDEP073E01、Waveshare 7.3" E6）规格集中在：

- 800×480；
- 有效显示区约 160×96 mm；
- 外形约 170.2×111.2×0.9 mm；
- SPI；
- 六色 Spectra 6；
- 全刷通常十几秒级。

这些产品非常适合做 P0/P1 的软件、调色、传输和状态机验证，但**常见公开版本不能自动视为柔性最终眼片**。

2026 年行业已经出现柔性 Spectra 6 / 柔性色彩 EPD 展示与技术论文，但 7 英寸级可零售、可长期供货的柔性全彩面板仍属于必须逐型号确认的高风险环节。

## 3.2 P0 屏幕推荐策略

### A 路线：优先使用用户实际准备购买的柔性屏

这是最优方案。

采购前必须拿到：

- 完整型号；
- 面板厂商；
- TFT 背板类型；
- 是否 plastic TFT / flexible TFT；
- 最小弯曲半径；
- 是否允许长期定曲率安装；
- FPC 型号；
- FPC 引脚；
- Driver IC / TCON；
- SPI/并口类型；
- BUSY 定义；
- RST、CS、DC 等逻辑；
- 工作电压；
- 刷新峰值电流；
- 推荐驱动板；
- LUT / waveform 是否在 OTP；
- 温度补偿需求。

### B 路线：先用 7.3" Spectra 6 标准屏作“电气开发代理”

推荐作为代理的规格族：

- E Ink 官方 7.3" Spectra 6 ED2208-GCA；
- Good Display / OpenELAB GDEP073E01 类；
- Waveshare 7.3" Spectra 6 HAT/裸屏方案。

用途：

- 验证 800×480 图像处理；
- 验证六色色彩映射和抖动；
- 验证 SPI 驱动；
- 验证 BUSY 状态机；
- 验证 microSD → Frame → EPD；
- 验证双眼独立刷新；
- 验证 App Send/Commit；
- 为最终柔性屏驱动软件建立 Display Adapter。

**注意：代理屏通过，不代表机械柔性要求通过。**

## 3.3 不建议

第一版不建议：

- 直接根据商品标题“柔性”判断；
- 无 datasheet 的定制屏；
- 只有 Demo 视频、不给驱动文档的屏；
- 为了追求快速刷新而改变第一版产品定义；
- P0 就自己开发 EPD 高压波形和 PMIC。

---

# 4. 主控制器：NXP RW612

## 4.1 结论

**首选：NXP RW612。**

原型阶段直接使用：

> **FRDM-RW612**

NXP 当前将 RW612 标记为 Active。官方资料显示其主要能力包括：

- 260 MHz Arm Cortex-M33；
- 1.2 MB on-chip SRAM；
- Wi-Fi 6，2.4/5 GHz；
- Bluetooth LE 5.4；
- 802.15.4；
- Quad FlexSPI；
- PSRAM 扩展接口；
- 最多 5 组可配置 Flexcomm（SPI/I²C/I²S/UART）；
- LCD interface；
- 单 3.3 V 外部供电能力。

FRDM-RW612 开发板本身还提供：

- 512 Mbit Winbond QSPI Flash；
- 64 Mbit QSPI PSRAM；
- Pmod / mikroBUS / Arduino 扩展；
- MCU-Link 调试器；
- USB Type-C；
- SPI/I²C/UART 接口。

## 4.2 为什么适合本项目

RW612 很适合作为 **Control Plane MCU**，负责：

- BLE；
- Wi-Fi；
- 手机协议；
- Asset ID；
- State Engine；
- Animation Command；
- microSD；
- IMU；
- UWB Host；
- 左右显示 Worker；
- OTA；
- Diagnostics。

第一代 E-Ink 不需要高帧率渲染，RW612 性能足够。

## 4.3 最终 PCB：裸芯片还是模组

### 裸 RW612

优点：

- PCB 面积更可控；
- BOM 可优化；
- 天线位置可针对头壳优化。

缺点：

- Wi-Fi/BLE RF 设计和认证难度增加。

### RW612 模组

NXP 当前列出了多个 RW612 Partner Module，例如 AzureWave、KAGA FEI、CEL 等方案。

优点：

- 降低 2.4/5 GHz RF 设计风险；
- 更适合第一版定型。

缺点：

- 尺寸更大；
- 国内小批量采购情况需要确认。

### 推荐

P0-P15：**FRDM-RW612**。  
P16 Final PCB：优先评估“RW612 模组”，若体积不满足再转裸芯片。

---

# 5. 头壳端 UWB：NXP Trimension SR250

## 5.1 结论

**首选：SR250。**

NXP 当前标记 SR250 为 Active，官方资料明确支持：

- 3D AoA；
- 360° AoA；
- TDoA；
- ranging；
- on-chip radar processing；
- 直接 3.7 V 电池连接能力。

这与本项目“摄影机方向 + 距离 + 非光学追踪”的需求高度匹配。

## 5.2 原型阶段

优先：

> **NXP Trimension SR250 Development Board / SR250-ARD 官方开发方案**

NXP 在 2026 年已经发布：

- SR250 开发板；
- Getting Started；
- SR250 Hardware Design Guide；
- UCI Specification；
- Zephyr UWBIOT middleware；
- Application Code Hub 示例。

因此 P9-P12 不应该自己从裸芯片搭天线阵列。

## 5.3 Final PCB 强烈建议评估集成 3D 天线模组

NXP Partner Marketplace 当前列出 **SR250 Integrated 3D Antenna Module**，例如 Amotech 的集成方案：

- SR250；
- 时钟；
- 滤波；
- 外围器件；
- 3 个内置陶瓷天线。

对于本项目，这是非常重要的降风险路线。

因为 3D AoA 的难点不只是“有 3 根天线”，还包括：

- 天线几何；
- RF path 相位差；
- PCB 介质；
- 走线长度；
- 天线互耦；
- 头壳材料；
- 电池、铜、DC/DC 对方向图的影响；
- calibration。

### 推荐顺序

1. 官方 SR250 开发板跑通；
2. 验证 Camera Beacon + SR250 的真实 AoA；
3. 在真实头壳材料附近做遮挡和方向图测试；
4. 尝试 Partner 3D Antenna Module；
5. 最后才考虑自研裸 SR250 + 三天线阵列。

---

# 6. Camera Beacon 选型

## 6.1 首选候选：NXP SR040 类 Tag

SR040 当前仍为 Active，NXP 将其定位为低功耗 UWB IoT Tag，适合电池设备，并支持 IEEE 802.15.4z / FiRa 生态。

优点：

- 低功耗导向；
- Tag 定位明确；
- 与 NXP UWB 生态一致；
- 适合做小型相机 Beacon。

但这里必须保持一个工程约束：

> **SR040 不能在文档阶段就视为已验证可满足本项目 SR250 3D AoA 会话。**

P9 必须实际验证：

- channel；
- session 类型；
- ranging / AoA compatibility；
- 更新率；
- 多 Beacon ID；
- 连接丢失恢复；
- Beacon 电流。

## 6.2 备选：Qorvo DWM3001C

DWM3001C 是成熟的集成 UWB 模组，包含：

- UWB；
- BLE SoC；
- 板载 UWB 天线；
- motion sensor；
- 电源管理；
- crystal；
- FiRa 兼容。

优势是原型搭建快，RF 风险低。

缺点：

- 与 NXP SR250 生态跨厂商；
- 必须验证 FiRa / ranging 配置和 AoA 兼容；
- 尺寸相对大。

## 6.3 不建议作为新设计首选：SR150 路线

Murata Type 2BP / NXP SR150 方案技术上支持 2D/3D AoA、TWR、TDoA，而且 2026 年 Murata 仍有生产资料；但 NXP 自己的 Type2BP 页面已经出现 Archived 状态，生态正在向 SR250 迁移。

因此：

- 可作为实验备份；
- 不建议把新项目长期主线重新建立在 SR150 上。

---

# 7. IMU 选型

## 7.1 首选：Bosch BMI270

推荐理由：

- 面向 wearable；
- 6 轴；
- 16-bit gyro + accel；
- SPI / I²C；
- 体积 2.5×3.0×0.8 mm；
- 满 ODR 电流约 685 µA；
- 驱动生态成熟。

在本项目中不要求 IMU 负责找到摄影机，因此 BMI270 性能已经足够：

- 检测头部是否在快速运动；
- 运动状态门控；
- 日志；
- 后续补偿。

推荐接口：

> **SPI 优先**，若总线资源紧张再考虑 I²C。

## 7.2 高性能备选：TDK ICM-42688-P

适合后续需要更低噪声 gyro 或更强运动补偿时使用。

其官方资料显示：

- 6 轴；
- SPI 最高 24 MHz；
- 低噪声模式；
- gyro noise 2.8 mdps/√Hz；
- 6-axis low-noise current 约 0.88 mA。

## 7.3 通用备选

- Bosch BMI323：新一代通用 6 轴，支持 I3C/I²C/SPI；
- ST LSM6DSO32X：适合更高冲击范围、内置 FSM/MLC 的路线。

第一版没必要为了规格堆料而换掉 BMI270。

---

# 8. 本地存储

## 8.1 首选：microSD

V1 必须保留 microSD。

理由：

- 大量 PNG / raster；
- Sequence；
- Character Pack；
- OLED/LCD 后续帧动画和视频；
- 用户换卡方便；
- PC 直接准备素材方便；
- 容量和成本都优于把所有资源塞入 MCU Flash。

### 容量建议

原型：

- 32 GB 即足够。

后续 OLED/LCD 视频较多时：

- 64/128 GB。

软件必须根据文件系统实现，而不是把容量写死。

## 8.2 microSD 接口

P1-P8 建议：

- 先用现成 microSD 模块或 FRDM 扩展；
- 确定实际吞吐后再决定 SPI 还是 SDIO。

如果只是 E-Ink 静态图：SPI 足够。

如果未来直接本地播放高码率 OLED/LCD 动画：Final PCB 应考虑 SDIO 或更高速本地存储。

## 8.3 外部 QSPI Flash

建议保留用于：

- firmware image；
- OTA A/B；
- 配置；
- crash log；
- 常用 asset cache。

FRDM-RW612 已带 512 Mbit QSPI Flash，可用于原型评估。

---

# 9. 左右眼驱动架构

## 9.1 原型阶段

每只眼睛建议独立：

```text
RW612
├─ Left Display Worker  → Left Driver Board → Left EPD
└─ Right Display Worker → Right Driver Board → Right EPD
```

每路至少独立：

- CS；
- RST；
- BUSY；
- Power Enable；
- 状态机。

SPI 时钟/MOSI 可以在屏幕时序允许时共享。

## 9.2 为什么不建议 P0 自研屏驱动电源

彩色 EPD 往往涉及：

- panel controller；
- booster；
- VCOM；
- gate/source 驱动；
- 温度补偿；
- waveform。

第一版最大的风险是“屏到底能不能用”，不是“能不能少一块驱动板”。

因此 P0-P2：

> **优先使用屏厂原装/成熟驱动板。**

P16 再决定是否把 EPD Driver 集成到 Eye Module PCB。

---

# 10. 电源系统推荐

## 10.1 目前不冻结总功率

这是必须继续遵守的原则。

在实际测得：

- 双屏刷新峰值；
- RW612 Wi-Fi 峰值；
- SR250 峰值；
- microSD 峰值；
- Driver Board 峰值；

之前，不确定：

- 电池容量；
- USB-C PD 档位；
- 总线电压；
- DC/DC 额定功率。

## 10.2 推荐的电源域

```text
Main Input
  │
  ├─ 3.3V Control Rail
  │   ├─ RW612
  │   ├─ microSD / logic
  │   └─ IMU
  │
  ├─ UWB Rail
  │   └─ SR250 / UWB Module
  │
  ├─ Left Eye Rail
  │   └─ Left Driver
  │
  └─ Right Eye Rail
      └─ Right Driver
```

左右眼建议独立 Enable，便于：

- 单眼故障隔离；
- 单眼重启；
- 功耗测量；
- 维护。

## 10.3 3.3 V 候选

如果未来输入可能出现：

- 1S Li-ion 电池；
- USB 5 V；
- 其他上下跨过 3.3 V 的输入；

可评估：

> **TI TPS63070 类 2～16 V Buck-Boost**

官方规格支持宽输入、最高约 2 A 输出等级（具体工况需按 datasheet derating）。

如果最终输入始终显著高于 3.3 V，则 Buck-Boost 没必要，可改为高效率 Buck，例如：

- TPS62130A；
- 或更新一代 TPS62903 类。

### 结论

**现在只确定电源架构，不锁具体 DC/DC。**

---

# 11. 接口与连接器

## 11.1 Eye Module

建议每眼形成独立可替换模块：

```text
Display
+ Driver
+ Local Support Frame
+ Connector
```

主板到 Eye Module 传：

- 低压电源；
- SPI / control；
- BUSY；
- reset；
- power enable / status。

不要为了安装方便随意延长原始屏幕 FPC。

## 11.2 屏幕 FPC

屏幕 FPC 和 Driver Board 尽量保持原厂推荐长度和结构。

主控到 Driver Board 可以使用：

- JST-GH；
- JST-SH；
- Molex Pico-Lock / PicoBlade 类；
- 小型锁扣 FFC；

最终依线数、电流和插拔寿命选定。

## 11.3 调试接口

Final PCB 建议保留：

- SWD；
- UART Console；
- USB-C；
- Test Pads；
- Boot/ISP；
- Reset。

UWB 模块也应预留独立 reset / wake / interrupt 测点。

---

# 12. RF 与 PCB 布局建议

## 12.1 UWB

优先位置：

> 头顶前部 / 顶部中央

要求：

- 离大电池远；
- 离 EPD Driver 升压电路远；
- 离电感远；
- 避免金属件包围；
- 避免大面积铜直接位于天线前方；
- 保证天线阵列几何固定。

## 12.2 Wi-Fi / BLE

RW612 天线与 UWB 阵列需要分区设计。

如果 Final PCB 使用带天线 RW612 模组，应在机械设计阶段就给天线净空，不能 PCB 做完后再找位置。

## 12.3 DC/DC

开关电源集中放在：

- 远离 UWB；
- 远离 IMU；
- 远离天线净空区。

并尽量控制开关节点面积。

---

# 13. 机械选型与安装相关硬件

## 13.1 屏幕背板

柔性不代表可以悬空使用。

每眼建议：

- 薄 PET / PC 支撑；
- 轻量 3D 打印框；
- 泡棉缓冲；
- 避免点载荷压在 TFT/FPC 过渡区。

## 13.2 Eye Module 快拆

建议用：

- 2～4 个小螺钉；
- 定位柱；
- 卡扣只做辅助；
- 电连接器独立可拔。

这样坏一块屏不需要拆整套主控。

## 13.3 Camera Beacon

机械接口建议：

- 冷靴；
- 1/4"-20；
- 手机夹；
- MagSafe 转接；
- GoPro 转接。

Beacon 本体保持通用，转接件独立。

---

# 14. 开发板与采购优先级

## 第一批：P0 必买

1. 候选柔性彩色 E-Ink × 1；
2. 对应官方/商家成熟驱动板 × 1；
3. 备用 FPC / 转接板；
4. 简单 MCU/PC 驱动环境；
5. USB 电流表或功耗分析设备。

**先只买一块最终候选屏。**

## 第二批：P1-P4

1. 第二块同型号 E-Ink；
2. FRDM-RW612 × 1；
3. microSD 模块 + 32 GB 卡；
4. BMI270 开发小板；
5. 面包板/转接线/逻辑分析仪；
6. 左右显示独立供电开关测试模块。

## 第三批：P9 UWB

1. SR250 官方开发板；
2. Camera Beacon 候选开发板；
3. UWB 支架；
4. USB 延长/独立供电；
5. 用于测距和角度标定的机械转台/刻度夹具。

## 第四批：P16 Final PCB 前

1. SR250 集成 3D 天线模组样品；
2. RW612 module 样品；
3. 候选 DC/DC EVM；
4. 候选连接器；
5. 最终 microSD socket；
6. 电源与 EMI 测试器材。

---

# 15. P0～P3 原型 BOM 建议

## P0：单屏验证 BOM

| 项目 | 数量 | 推荐 |
|---|---:|---|
| 柔性彩色 E-Ink 候选屏 | 1 | 用户最终采购候选 |
| 原厂/成熟 EPD Driver Board | 1 | 与屏严格匹配 |
| FPC/Adapter | 1～2 | 原厂配套 |
| USB 电源/稳压源 | 1 | 可记录电流更好 |
| 逻辑分析仪 | 1 | 检查 SPI/BUSY/RST |
| 测试图 | 若干 | 六色块、渐变、眼片素材 |

## P1：单眼本地播放

新增：

| 项目 | 数量 | 推荐 |
|---|---:|---|
| FRDM-RW612 | 1 | 首选主控开发板 |
| microSD 模块 | 1 | 原型期模块即可 |
| 32 GB microSD | 1 | 可靠品牌 |
| BMI270 Board | 1 | 可先不上板运行 |

## P2：双眼

新增：

| 项目 | 数量 | 推荐 |
|---|---:|---|
| 第二块同型号屏 | 1 | 必须与左眼一致 |
| 第二路 Driver Board | 1 | 独立驱动 |
| 独立 Power Enable 测试 | 2 路 | 左右分开 |

## P3：正式 EyeState / State Engine

硬件通常无需新增，重点进入固件架构。

---

# 16. 关键验证清单

## 16.1 屏幕

- [ ] 真正柔性基板，而非“软排线 + 玻璃屏”
- [ ] 可接受的长期弯曲半径
- [ ] FPC 规格完整
- [ ] Driver IC 明确
- [ ] BUSY 行为明确
- [ ] 连续 100 次刷新无异常
- [ ] 断电保持正确
- [ ] 温度变化测试
- [ ] 左右屏色差测试
- [ ] 眼眶遮挡后有效显示区域足够

## 16.2 RW612

- [ ] BLE 与手机稳定连接
- [ ] Wi-Fi 素材上传
- [ ] microSD 长时间读写
- [ ] 双 SPI/共享 SPI 资源足够
- [ ] OTA 方案验证
- [ ] 断连恢复

## 16.3 UWB

- [ ] 前方 Zero Calibration
- [ ] ±水平角测试
- [ ] ±俯仰角测试
- [ ] 1～5 m 距离测试
- [ ] 头壳佩戴后测试
- [ ] 人体遮挡测试
- [ ] 左右转头测试
- [ ] Beacon 电池供电测试
- [ ] Beacon ID 多设备测试
- [ ] Camera Beacon 与镜头 Offset 标定

## 16.4 EMI

- [ ] EPD 刷新时 UWB 是否跳变
- [ ] Wi-Fi 上传时 UWB 是否受扰
- [ ] DC/DC 对 IMU 是否有影响
- [ ] 双屏同时刷新时主控是否复位

---

# 17. 当前不建议锁定的硬件

以下项目现在不要写死到 BOM：

1. 最终电池容量；
2. USB-C PD 电压；
3. 总输入电压；
4. EPD Driver PMIC；
5. 最终 Buck/Buck-Boost 型号；
6. 最终 Camera Beacon 电池；
7. SR250 自研天线结构；
8. OLED/LCD Display Processor；
9. OLED/LCD 分辨率和接口；
10. 视频 Codec。

这些必须等待对应阶段实测。

---

# 18. 最终推荐组合

## 当前 E-Ink V1 原型

```text
Phone App
   │ BLE / Wi-Fi
   ▼
FRDM-RW612
   ├─ microSD
   ├─ BMI270
   ├─ Left EPD Driver → Flexible Color E-Ink L
   └─ Right EPD Driver → Flexible Color E-Ink R

SR250 Development Board
   ▲
   │ UWB
Camera Beacon Prototype
```

## Final PCB 推荐发展方向

```text
RW612 / RW612 Module
        │
        ├─ microSD
        ├─ BMI270
        ├─ SR250 Integrated 3D Antenna Module
        ├─ Left Eye Interface + Power Switch
        └─ Right Eye Interface + Power Switch
```

这一路线的重点是：

> **Final PCB 不自行承担最困难的两个首版风险：柔性 EPD 面板物理未知和 SR250 3D AoA 天线阵列从零 RF 设计。**

先用成熟屏驱动和集成 UWB 模组把系统做成，再根据实测决定第二版是否进一步高度集成。

---

# 19. 推荐结论优先级

### 可以现在就定

- 主控生态：RW612；
- 主控开发板：FRDM-RW612；
- 头壳 UWB：SR250；
- UWB 原型：官方开发板；
- IMU：BMI270；
- 本地媒体存储：microSD；
- 左右眼独立 Driver/Power Domain；
- Final PCB 优先评估 SR250 集成 3D 天线模组。

### 需要 P0 后定

- 最终 7 英寸柔性彩色 E-Ink 型号；
- Display Driver；
- FPC/Connector；
- 每眼刷新峰值电流；
- Display Rail。

### 需要 P9 后定

- Camera Beacon 最终芯片；
- Beacon 更新率；
- Beacon 电池；
- UWB Session 参数。

### 需要 P15 后定

- OLED/LCD 主控还是独立 Display Processor；
- 视频/分层渲染处理器；
- Frame Buffer；
- 高速本地存储接口。

---

# 20. 主要资料来源

以下资料用于本次选型核验，建议后续工程阶段优先以官方最新版 datasheet / hardware design guide 为准。

## NXP

- RW612 Product Page  
  https://www.nxp.com/products/RW612
- FRDM-RW612 Development Board  
  https://www.nxp.com/design/design-center/development-boards-and-designs/FRDM-RW612
- Trimension SR250 Product Page  
  https://www.nxp.com/products/SR250
- Trimension SR250 Development Board  
  https://www.nxp.com/design/design-center/development-boards-and-designs/TRIMENSION-SR250
- Trimension SR040 Product Page  
  https://www.nxp.com/products/wireless-connectivity/trimension-uwb/trimension-sr040-reliable-uwb-solution-for-iot%3ASR040

## E Ink / E-Paper

- E Ink 7.3" Spectra 6 Display  
  https://shopkits.eink.com/en/product/detail/7.3%27%27Spectra6ePaperDisplay
- E Ink Kaleido Technology  
  https://www.eink.com/brand/detail/Kaleido
- Waveshare 7.3" Spectra 6  
  https://www.waveshare.com/7.3inch-e-paper-hat-e.htm
- Good Display / GDEP073E01 class reference  
  https://openelab.com/products/inch-e-paper-color-e

## IMU

- Bosch BMI270  
  https://www.bosch-sensortec.com/en/products/motion-sensors/imus/bmi270
- TDK ICM-42688-P  
  https://www.invensense.tdk.com/en-us/products/6-axis/icm-42688-p
- Bosch BMI323  
  https://www.bosch-sensortec.com/en/products/motion-sensors/imus/bmi323
- ST LSM6DSO32X  
  https://www.st.com/en/mems-and-sensors/lsm6dso32x.html

## UWB 备选

- Qorvo DWM3001C  
  https://cn.qorvo.com/products/p/DWM3001C
- Murata Type 2BP / NXP SR150  
  https://www.murata.com/products/connectivitymodule/ultra-wide-band/nxp/type2bp

## Power

- TI TPS63070  
  https://www.ti.com/product/TPS63070
- TI TPS62130A  
  https://www.ti.com/product/TPS62130A

---

# 21. 下一步

硬件选型文档完成后，项目应立即进入 P0，而不是继续纸面堆 BOM。

P0 的第一项实际动作应为：

> **确定并核验用户准备购买的那块约 7 英寸柔性彩色 E-Ink 的完整型号和商品资料。**

拿到具体型号后，下一份文档应是：

`P0_柔性彩色电子纸屏幕验证计划.md`

其中逐项给出：

- Datasheet 审核；
- FPC pinout；
- 驱动板确认；
- MCU 接线；
- Test Pattern；
- 刷新次数；
- 电流测量；
- 弯曲测试；
- 屏幕损伤判定；
- P0 Pass/Fail 标准。
