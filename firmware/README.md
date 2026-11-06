# Firmware

头壳端固件目录。

规划模块：BLE Service、Wi-Fi Service、UWB Service、IMU Service、Tracking Engine、State Engine、Output Manager、Asset Manager、Animation Engine、Storage Manager、Display Adapter、Left/Right Display Worker、Diagnostics。

当前阶段：P0 屏幕验证完成前不建立最终固件框架。

## 2026-11-06 复核后的最小实现范围

- E-Ink 首发只实现 **STATIC + 有界 SEQUENCE**；任何有界 Sequence 都必须由一次显式 Commit 授权，含最大循环次数/时长，不允许未经新 Send 无限自动改变实体眼片。
- Commit 必须包含不可变 Snapshot、Transaction ID、Asset ID/Hash、eye_scope、幂等处理、持久化 Journal、按眼状态/错误和断连恢复。
- MANUAL_COMMIT 模式下 TrackingState/PreviewState/Telemetry 不得进入 Display Worker 刷新队列。
- Busy 策略首版建议：仅一项 ACTIVE；新 Commit 明确返回 BUSY，手机保留未发送预览并提供重新发送，后续再扩展显式排队。
- RW612 保留 Control Plane；未来 OLED/LCD 帧渲染/视频解码不得默认由 RW612 单独承担。
- 固件实现前参考 [ADR-0001](../docs/decisions/ADR-0001-risk-gated-v1.md)；UWB 先独立验证再集成。
