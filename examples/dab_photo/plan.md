# DAB 染色切片拍照实验 · 实验室自动化规划 (S-LAB)

样本：8 张 DAB 染色石蜡切片（带红色描边 + QR 码，含 N10↑、F11↑、N11↑ 等标记）
目标：通过自动化显微成像流水线获取可定量 DAB 信号的高分辨率切片照片，并归档。

---

## 1. 实验模块化分解

| 模块 | 责任 | 输入 | 输出 |
|------|------|------|------|
| M1 上样 | 人工将切片按序送入载物台 | DAB 切片 | 已定位切片 |
| M2 初筛 QC | 4× 扫描 + QR 识别 + 组织判定 | 已定位切片 | 带 ID 的有效切片 |
| M3 低倍预扫 | 4× 组织全景扫描，定位 ROI | 有效切片 | ROI 坐标 |
| M4 XY 定位 | 电动载物台移动到 ROI | ROI 坐标 | 100× 视野 |
| M5 高倍拍照 | 100× 物镜自动对焦 + 多通道采集 | 100× 视野 | 原始高分辨率图 |
| M6 拼接与增强 | 图像拼接、去噪、校正白平衡 | 原始图 | 整合大图 |
| M7 DAB 定量 | DAB 通道强度/阳性面积分析 | 整合大图 | 定量数据 |
| M8 归档 | 切片入库 + 图像数据入库 | 切片 + 数据 | 归档记录 |

---

## 2. 资源映射表 (machines.tsv)

| 机器类型 | 资源名称 | 承担操作 |
|----------|----------|----------|
| 1 | Manual_Slide_Loader | 切片排队 / 上样 |
| 2 | Scanner_4x | QR 识别、组织 QC、低倍预扫 |
| 3 | Motorized_XY_Stage | 电动载物台 XY 定位 |
| 4 | Microscope_100x_HighMag | 100× 高倍拍照 |
| 5 | Slide_Storage_Rack | 切片归档存放 |
| 6 | Image_Processing_WS | 图像拼接增强、DAB 定量 |

**资源复用**：Scanner_4x 承担 2 个操作（O2 与 O3），Image_Processing_WS 承担 2 个操作（O6 与 O7），需通过调度避免冲突。

---

## 3. 操作节点表 (operations.tsv)

| ID | 操作名 | 兼容资源 | 加工耗时(s) | 输入状态 | 输出状态 | 阻塞/非阻塞 | QC 触发 |
|----|--------|----------|------------|----------|----------|-------------|---------|
| 1 | Queue_And_Load_Slide | Manual_Slide_Loader | 30 | 待上样切片 | 已定位切片 | 阻塞 | — |
| 2 | Identify_Tissue_QC | Scanner_4x | 20 | 已定位切片 | 带 ID 有效切片 | 阻塞 | QC1 QR 可识别+组织覆盖率≥70% |
| 3 | LowMag_Preview_Scan | Scanner_4x | 60 | 有效切片 | ROI 坐标 | 阻塞 | QC2 DAB 信号可见 |
| 4 | XY_Tissue_Positioning | Motorized_XY_Stage | 15 | ROI 坐标 | 100× 视野 | 阻塞 | QC3 定位误差<5μm |
| 5 | HighMag_Photo_Capture | Microscope_100x_HighMag | 180 | 100× 视野 | 原始高分辨率图 | 阻塞 | QC4 对焦评分>阈值 |
| 6 | Stitch_And_Enhance | Image_Processing_WS | 300 | 原始高分辨率图 | 整合大图 | 非阻塞 | QC5 接缝无明显断层 |
| 7 | DAB_Intensity_Quant | Image_Processing_WS | 60 | 整合大图 | 定量数据 | 非阻塞 | QC6 阳性面积比值输出 |
| 8 | Archive_Slide_Store | Slide_Storage_Rack | 20 | 已成像切片+数据 | 归档记录 | 阻塞 | QC7 位置码写入成功 |

---

## 4. TCMB 约束矩阵 (tcmb.tsv)

Time-Constraints by Mutual Boundaries（相邻节点的端到端时间窗，单位 s）：

| Op1 | 边界点1 | Op2 | 边界点2 | 最大允许等待 | 约束说明 |
|-----|---------|-----|---------|--------------|----------|
| 1 | end | 2 | start | 30 | 上样完成后必须 30s 内启动 QR 识别（防组织风干） |
| 2 | end | 3 | start | 60 | QR 识别到低倍扫描窗口 |
| 3 | end | 4 | start | 30 | 低倍 ROI 坐标 → 电动载物台移动 |
| 4 | end | 5 | start | 30 | 定位完成 → 100× 拍照启动（样品不得移位） |
| 5 | end | 6 | start | 30 | 拍照完成 → 数据工作站获取原始图 |
| 6 | end | 7 | start | 60 | 拼接完成 → 定量分析启动 |
| 7 | end | 8 | start | 30 | 定量完成 → 归档写入 |

---

## 5. 依赖关系表 (dependency.tsv)

严格先后依赖（强序）：

```
O1 → O2 → O3 → O4 → O5 → O6 → O7 → O8
```

形成线性 DAG，无分叉、无合并；每张切片为一个独立 job，可并行在不同资源车道上流动。

---

## 6. 操作 DAG（概念图）

```
 ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
 │ O1 Queue │──▶│ O2 QR ID │──▶│O3 LowMag │──▶│O4 XY Pos │
 │  & Load  │    │  + QC    │    │ Preview  │    │  Stage   │
 └──────────┘    └──────────┘    └──────────┘    └──────────┘
                                                       │
                                                       ▼
 ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
 │ O8 Store │◀──│ O7 DAB   │◀──│ O6 Stitch│◀──│O5 HighMag│
 │ Archive  │    │ Quantify │    │ Enhance  │    │ Capture  │
 └──────────┘    └──────────┘    └──────────┘    └──────────┘
```

---

## 7. 仪器泳道操作图（Gantt-style lanes）

| 资源泳道 \ 时间 → | 0–30s | 30–90s | 90–150s | 150–330s | 330–630s | 630–690s | 690–710s |
|--------------------|-------|--------|---------|----------|----------|----------|----------|
| Manual_Slide_Loader (1) | O1 | O1… | | | | | |
| Scanner_4x (2) | | O2+O3 | | | | | |
| Motorized_XY_Stage (3) | | | O4 | | | | |
| Microscope_100x_HighMag (4) | | | | O5 | | | |
| Image_Processing_WS (6) | | | | | O6 | O7 | |
| Slide_Storage_Rack (5) | | | | | | | O8 |

单个切片的流水线窗口约 **710 s**，对应理论节拍（理想串行）。

---

## 8. 自动化调度逻辑

1. **Job 粒度**：`N_job = 8`，每张 DAB 切片一个 job。
2. **调度模式**：`Sequential = 0`（允许跨资源并行，由 S-LAB 求解器自动排程）。
3. **批量策略**：
   - Loader（O1）连续送入切片 → 形成批队列。
   - Scanner_4x（O2、O3）串行处理每张切片的 QR+预扫，存在资源瓶颈。
   - Image_Processing_WS（O6、O7）在高倍拍照后可与下一张切片的 O1–O3 流水线并行，形成流水线重叠（pipeline overlap）。
4. **同步屏障**：O1→O2、O3→O4、O5→O6、O7→O8 构成硬同步屏障，由 TCMB 窗口保证。
5. **QC 门控**：任一 QC 点（QC1~QC7）失败则触发该 job 的 rework 分支（重上样 / 重扫描 / 重拍照 / 人工介入），由 MES 子系统挂起并记录事件。
6. **数据边**：切片物理流（load→store）与数据流（image→quant→DB）解耦，通过作业 ID 关联。

---

## 9. 关键路径分析（Critical Path）

单 job 关键路径总时长：

\[
T_{\text{CP}} = 30 + 20 + 60 + 15 + 180 + 300 + 60 + 20 = \mathbf{685\ s}
\]

（TCMB 缓冲上限未计入 CP，仅作为调度可行域约束。）

| 阶段 | 时长(s) | 是否可压缩 |
|------|---------|-----------|
| O1 上样 | 30 | 可通过并行 loader 压缩 |
| O2+O3 扫描 | 80 | 需升级多通道扫片机 |
| O4 XY 定位 | 15 | 已接近下限 |
| **O5 高倍拍照** | **180** | 视物镜/相机而定 |
| **O6 拼接增强** | **300** | GPU 加速可压缩 |
| O7 定量 | 60 | 可通过批量分析压缩 |
| O8 归档 | 20 | 可通过自动归档压缩 |

**关键瓶颈 O6（300s）与 O5（180s）合计占 CP 的 ~70%。**

---

## 10. 瓶颈分析（Bottleneck）

| 瓶颈节点 | 所在资源 | 原因 | 缓解策略 |
|----------|----------|------|----------|
| O3 LowMag_Preview_Scan | Scanner_4x | 承担 O2、O3 双操作，串行 | 引入第二台 4× 扫描通道 |
| O5 HighMag_Photo_Capture | Microscope_100x | 单 100× 镜资源，每切片 180 s | 配置第二台高倍镜并行 |
| O6 Stitch_And_Enhance | Image_Processing_WS | 图像计算密集 | GPU 集群 / 预计算流水线 |
| Loader/O1 → 批队列积压 | Manual_Slide_Loader | 人工节拍限制 | 引入自动上样机械臂 |

**资源竞争**：
- Scanner_4x 同时承载 O2、O3，需保证 QR 识别与低倍扫描在同一张切片上顺序执行，不可并行。
- Image_Processing_WS 同时承载 O6、O7，O6 完成前 O7 不可启动（依赖约束），O6 与下一张切片的 O5 可并行。

---

## 11. 配置文件

- 目录：`examples/dab_photo/`
- 文件：
  - [machines.tsv](file:///workspace/examples/dab_photo/machines.tsv)
  - [operations.tsv](file:///workspace/examples/dab_photo/operations.tsv)
  - [dependency.tsv](file:///workspace/examples/dab_photo/dependency.tsv)
  - [tcmb.tsv](file:///workspace/examples/dab_photo/tcmb.tsv)
  - [config.tsv](file:///workspace/examples/dab_photo/config.tsv)

运行：

```bash
julia --project=. bin/run.jl examples/dab_photo
```

`Plot_range = 1200 s` 覆盖 8 张切片的流水线并行窗口。

---

## 12. 术语对照（Discrete Event System / Manufacturing Scheduling）

- Job = 单张 DAB 切片
- Operation = 流程单元（带阻塞/非阻塞属性）
- Machine = 资源车道（可被多个 operation 复用）
- TCMB = 相邻操作的端到端时间窗，对应制造调度中的"紧耦合时间约束"
- QC checkpoint = 质量门（与 MES 中的 in-process inspection 对齐）
- Critical Path = 单 job 端到端最短可行完成路径
- Bottleneck = 节拍最小的资源节点，决定系统吞吐量上限
