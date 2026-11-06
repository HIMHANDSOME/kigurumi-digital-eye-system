# ADR-0001：V1.0 范围收敛、供应链与风险 Gate 决策

- 日期：2026-11-06
- 状态：**Accepted for planning / 尚未完成硬件验证**
- 依据：[可行性与经济性复核](../research/Kigurumi_V1.0_可行性与经济性复核_2026-11-06.md)
- 适用：README、技术方案、硬件选型、里程碑、采购与验收

## 1. 变更背景

原 V1.0 定义了双约 7 英寸柔性彩色电子纸、UWB 摄影机追踪、手机 Head Fit、离散动作、未来 3D 与 OLED/LCD。深度复核确认：控制与软件框架总体可行，但最终 **真正柔性彩色约 7 英寸 EPD 供应链**和 **实际佩戴的 3D AoA**没有完成验证。继续把器件候选或公开产品特性写成“已确认”有采购、工程和商业风险。

## 2. 现在冻结的产品需求（不删除）

- 两眼各一块独立显示；首代首选约 7 英寸真正柔性彩色反射式 EPD。
- E-Ink：必须手机本地预览；每一次实体动作都经过用户明确 Send/Commit。
- 手机 Head Fit：左右分别 XY、比例、旋转、眼眶 Mask、每头壳设原点。
- 无光学追踪：使用 UWB 摄影机 Beacon；头壳无需摄像头、红外或光学器件。
- 头端存储素材，手机传 Asset ID/状态/控制；双眼独立状态与故障管理。
- 软件从最初版本保持 PREVIEW_ONLY / MANUAL_COMMIT / REALTIME 三种输出策略兼容。
- macOS Apple Silicon 作为主开发环境；可保留 Windows 给少数厂商工具备用。

## 3. 立即修改的技术假设

| 旧表达 | 新状态 |
|---|---|
| 7″ 柔性全彩屏已选定 | **Not Verified**；P0 供应商/驱动/弯曲/报价 Gate |
| 7.3″ Spectra 6 满足最终屏 | **否**；仅作为玻璃电气代理 |
| RW612 为最终芯片 | **优先候选**；FRDM-RW612 先验证接口/资源/功耗 |
| SR250 保证 360° 全场稳定追踪 | **仅原厂功能**；实际头壳覆盖/精度待验证 |
| SR040 与 SR250 已可协同 | **候选待互操作**；用 Dev Kit 联调 |
| E-Ink 播放流畅视频 | **V1 不承诺**；只 STATIC/SEQUENCE |
| 未来 OLED/LCD 视频解码由 RW612 完成 | **不成立为默认设计**；Display Processor 另选 |
| P9 才开始研究 UWB | **高风险台架提前并行**；仍保留 P9 正式集成编号 |
| 所有最终硬件可以直接按表采购 | **不允许**；价格/交期/许可证/供货均需书面证据 |
| 技术/生产成本可用单一 BOM 代表 | **不允许**；必须分 BOM、NRE、组测、税运、损耗、认证、人工 |

## 4. 版本范围

### V1 软件/实体最低可用目标

1. 左右独立彩色 EPD 静态显示（最终柔性屏必须通过 G0；玻璃代理仅算开发样机）。
2. Manifest/Asset ID/静态素材预加载。
3. 手机 2D Preview、Head Fit、Set Origin 与 Profile。
4. 每次用户点击 Send 创建不可变事务；断线可恢复结果，左右独立报告完成/失败。
5. E-Ink 显式授权后的 **有界离散 Sequence**；不做无限机械无意义刷新。
6. UWB TrackingState 预览作为目标；真实头壳 3D AoA Gate 通过后再标记完成。

### 兼容性保留，但不成为 E-Ink V1 阻塞项

- App 的 REALTIME Output Policy 和 Headless Tracking；
- OLED/LCD 显示处理器与高帧率分层渲染；
- 复杂 Frame/Video 动画解码；
- 通用 3D Head Preview、GLB/glTF 导入和摄影机视角；
- 多 Beacon 自动选择、多头壳云同步和线上素材商店。

## 5. 设计约束（不可回退）

### 5.1 显示的状态边界

PreviewState 只在手机；TrackingState 是遥测，不是 Commit；无显式 Send 时，MANUAL_COMMIT 模式下实体眼片绝不被编辑/追踪更新。

Commit 内容至少：transaction_id、设备/头壳档案版本、eye_scope、asset id/hash、完整左右眼目标状态快照、retry/idempotency key、执行限制。持久化 journal。最多一个 ACTIVE；Busy 时默认拒绝新的 Commit 并提示明确重试（可未来增加用户显式排队，禁止隐式覆盖和无限队列）。左眼成功、右眼失败时须分别报告并允许用户显式单眼重试。

### 5.2 E-Ink Sequence 的显式授权范围

用户按一次 Send 可以批准一段完整的有界动作序列；有限帧数、最大重复次数与总时长写在事务快照中。手机断开后可执行至已批准边界；不得把“本地视频循环不需要持续控制”错误推广为“未经新 Send 可无限改变 E-Ink”。

### 5.3 层间边界

EyeState / Head Profile / Asset Manifest / Display Adapter / Control-Display Plane 必须独立。Glass EPD Proxy 和未来 OLED/LCD 必须通过不同 Capability/DisplayProfile，不可写同一型号假值。

## 6. 风险 Gate 定义

- **G0（最高优先）**：最终柔性彩色 EPD 料号、驱动/波形权利、材料、弯曲规格、100/1000 量级有效报价与 20–50 次样片测试；不通过时只能继续玻璃代理验证，不准最终整机 PCB。
- **G1**：双眼各 100 次可靠刷新、一眼异常不影响另一眼；显示状态一致性通过。
- **G2**：不点击 Send 绝不改变实体；断线恢复/幂等/Head Fit 重建测试。
- **G3**：资产 Hash、Wi-Fi 关闭下本地 STATIC/SEQUENCE、存储故障恢复。
- **G4**：SR250+Beacon 真实头壳 30min+ 测试、覆盖/遮挡/多径和区域射频核查；失败则追踪模块延期，或改射频技术路线。
- **G5**：同一 EyeState 适配 2D/3D/未来 OLED，不改核心协议。
- **G6**：EVT/DVT 20–50 台、功率/温升/EMI/寿命和供应链复核。
- **G7**：PVT 100 台，建议 FPY≥95%、单台校准≤8min；如果 FPY<90% 或返修不可接受，暂停放量。

## 7. 工程启动顺序

**两个台架并行，而不是一条 19 阶段长队列：**

- Track A：屏供应商 RFQ → 原厂驱动/LUT → 标准玻璃 EPD 代理验证 → 真柔性样片 G0。
- Track B：FRDM-RW612 + SR250 / Beacon 开发板 → 实头壳角度/遮挡/区域频谱 G4。
- Track C：EyeState/Commit + 手机 Preview + Head Fit + Asset/STATIC/SEQUENCE。
- P16（Final PCB）在 G0、G1、G4 与功耗记录充分之前禁止立项。

## 8. 资金与停损

- 任何 BOM 报价必须注明单价对应数量、有效期、币种、税运、MOQ、良率/返修。
- 未报价的最终柔性 EPD 不得用同尺寸玻璃屏价格代替。
- 5/20/100 台报价要分开发板样机/工程小批量/可能量产三种成本模型，不作线性外推。
- 所有时间与人月仅为规划；不作为交付承诺。
- 变更必须附：实验记录、供应商回复、订单/票据（可脱敏）、测试报告或 ADR。

## 9. 执行检查清单

- [ ] 向 ≥3 家候选真正柔性彩色 EPD 供应商发 RFQ。
- [ ] 保存原厂料号、Datasheet、波形/驱动权利和弯曲数据。
- [ ] 用标准玻璃 EPD 只验证控制链，不标作目标柔性样机。
- [ ] 建立 UWB 台架并在真实头壳记录盲区与追踪质量。
- [ ] 将 Stage/Gate 状态映射到项目 Issue/PR。
- [ ] 按测试证据决定 Go、Conditional Go、Pivot 或 No-Go。
