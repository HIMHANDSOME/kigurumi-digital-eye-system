# Hardware

本目录用于硬件设计、RFQ、PCB、原理图、BOM、接口、测试与证据。

> **当前状态：Conditional Go，尚无最终柔性彩色屏的 G0 PASS 记录。** 不能直接据旧选型文档采购最终器件或绘制量产 PCB。

## 相关文档

- [硬件推荐选型](../docs/Kigurumi_智能数字眼片系统_V1.0_硬件推荐选型.md)
- [深度研究：经济性与可行性复核](../docs/research/Kigurumi_V1.0_可行性与经济性复核_2026-11-06.md)
- [ADR-0001](../docs/decisions/ADR-0001-risk-gated-v1.md)
- [P0 柔性彩色 EPD RFQ + 验证计划](../docs/validation/P0-flexible-color-epd-gate.md)
- [UWB SR250+Beacon 并行台架](../docs/validation/UWB-aoa-parallel-poc.md)
- [成本模型（全部为估算假设）](../docs/cost-baseline-2026-11-06.md)

## 优先候选

- 目标屏：约 7″ 真柔性彩色 EPD ×2，**完整型号/驱动权利/量产报价待核实**；玻璃 Spectra 6 仅可作为 P0 电气代理。
- Control：NXP RW612 / FRDM-RW612（候选）。
- UWB：NXP SR250（候选），实头壳 AoA、区域射频和 Beacon 互操作需提前验证。
- Tag：SR040 / Murata Type 2DK 候选；不自动假定与 SR250 可互操作。
- IMU：BMI270 等六轴候选。
- Storage：microSD + 可恢复的 QSPI 关键资源。
- 双独立 EPD 驱动通道、BUSY/RESET 与电源域（实际电压按最终屏驱动确定）。

## 当前禁止提前冻结

- 最终屏型号/FPC/波形/物理曲率；
- 具体系统供电电压、功率、电池容量、DC/DC 型号；
- 自研三天线 UWB RF 最终布局；
- OLED/LCD 视频 Display Processor；
- 最终 PCB / 小批量工装。

用测试日志与 RFQ 数据逐项提升为 Verified。
