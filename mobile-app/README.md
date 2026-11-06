# Mobile App

手机端 Kigu Eye Controller 工作区。

核心模块：Device、Remote、2D Preview、Head Fit、Expressions、Animations、Tracking、Assets、Profiles、Diagnostics；后续追加 3D Head Preview 与 Camera View。

E-Ink 默认使用 `MANUAL_COMMIT`，OLED/LCD 保留 `REALTIME`。

## 2026-11-06 首版范围与输出边界

- V1：双眼 2D Preview、Head Fit、Set Origin、Eye Socket Mask、Expression/Gaze、Preview vs Displayed 差异、明确 Send、按眼事务状态、已缓存素材同步。
- 追踪结果在 E-Ink 模式仅影响预览，不直接修改物理屏幕；Beacon 丢失显示 LOST 并保持最后有效建议值。
- REALTIME 的协议/Capability 保留给未来 OLED/LCD；移动端不得据此向 E-Ink 自动发送。
- 自定义 GLB/glTF、3D Camera View、压缩视频与高帧率动画属后续迭代，不阻塞首个双屏操作 MVP。
- [ADR-0001](../docs/decisions/ADR-0001-risk-gated-v1.md) 为后续需求冲突时的优先决策依据。
