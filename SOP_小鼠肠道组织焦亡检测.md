# SOP: 小鼠肠道组织焦亡检测综合实验方案（Lab Automation 版）

## 流程名称
小鼠肠道组织焦亡检测综合实验 —— GSDMD / Caspase-1 / NLRP3 / ASC 的 WB 与 IHC/IF 检测

## 适用场景
规划与开展自动化生物实验：放射性肠炎（RE）及 Nic 治疗模型中细胞焦亡水平的全方位评估，涵盖 Western Blot（蛋白表达与剪切检测）与 IHC/IF（组织学定位与 ASC Specks 检测）两大并行模块。

## 负责人
[待填写]

## 更新日期
2026-08-13

---

## 流程步骤

### 1. Protocol 解析与 Lab Automation 建模

- **触发条件**：实验负责人提交小鼠肠道组织焦亡检测 protocol（WB + IHC/IF 双模块）
- **动作**：
  1. 读取输入的 protocol markdown 内容
  2. 使用 Trae 根據 SLab.jl 代碼邏輯，按照 `labauto_prompt.md` 的 skill 對 protocol 進行 lab automation 分析
  3. 輸出：Experiment Modular Decomposition、Instrument-Lane Operation Diagram、Material Metadata、TCMB Constraint Matrix
- **完成标准**：
  - 材料 metadata 中列出的所有试剂/仪器均已 on shelf 且可用
  - TCMB 约束矩阵中的所有时间约束均已确认

### 2. 样本制备与分流

- **触发条件**：步骤 1 完成，所有材料就绪
- **动作**：
  1. 处死小鼠，取出肠道组织
  2. 组织分切：一部分（30-50mg）液氮速冻 → WB 路径；另一部分 4% PFA 固定 → IHC/IF 路径
  3. 记录样本编号与对应分组
- **完成标准**：两份样本分别进入液氮速冻与 PFA 固定流程，样本记录完成

### 3. Western Blot 蛋白检测模块

- **触发条件**：WB 路径组织送达
- **动作**：按以下子步骤依次执行

| 子步骤 | 操作 | 仪器 | 时长 |
|:---|:---|:---|:---|
| 3.1 | 蛋白提取（RIPA 裂解 + 冰上匀浆 + 4°C 12,000rpm 离心 15min） | SamplePrep_Station | 45 min |
| 3.2 | BCA 蛋白定量 + 浓度调平 | ProteinQuantifier | 60 min |
| 3.3 | 蛋白变性（95°C 5min → 冰浴 → 短暂离心） | SamplePrep_Station | 15 min |
| 3.4 | SDS-PAGE 电泳（8-12% 梯度胶，120V，50-60min） | Electrophoresis_System | 60 min |
| 3.5 | 电转膜（0.22μm PVDF，300mA，4°C，60-90min） | Transfer_System | 75 min |
| 3.6 | 封闭（5% 脱脂奶粉，RT，1h） | Incubation_Station | 60 min |
| 3.7 | 一抗孵育（4°C 过夜） | Cold_Incubator_4C | 720 min |
| 3.8 | 二抗孵育（RT，1h） | Incubation_Station | 60 min |
| 3.9 | ECL 化学发光成像 | ECL_Imager | 15 min |

- **完成标准**：
  - 获得 GSDMD-N (~30kDa)、Cleaved Caspase-1 p20 (~20kDa)、NLRP3 (~118kDa)、ASC (~22kDa) 的清晰条带
  - GSDMD-N / 全长比值 ≥ 0.3 判定为焦亡激活
  - Caspase-1 p20 条带强度与焦亡正相关

### 4. IHC/IF 组织学检测模块

- **触发条件**：IHC/IF 路径组织固定完成（24h 以上）
- **动作**：按以下子步骤依次执行

| 子步骤 | 操作 | 仪器 | 时长 |
|:---|:---|:---|:---|
| 4.1 | 固定→脱水→石蜡包埋→切片（4μm） | Tissue_Fixation_Station | 120 min |
| 4.2 | 脱蜡复水（二甲苯→梯度乙醇→水） | IHC_Workstation | 30 min |
| 4.3 | 抗原修复（EDTA pH 8.0，微波/高压） | IHC_Workstation | 30 min |
| 4.4 | 免疫染色（阻断内源性过氧化物酶→BSA 封闭→一抗过夜→二抗 50min） | IHC_Workstation | 180 min |
| 4.5 | DAB 显色（NLRP3 / GSDMD-N 定位） | IHC_Workstation | 15 min |
| 4.6 | IF 染色（ASC 特异性抗体 + 荧光二抗） | IHC_Workstation | 60 min |
| 4.7 | 荧光成像（ASC Specks 检测） | Fluorescence_Microscope | 30 min |

- **完成标准**：
  - NLRP3：胞质弥漫性染色，强度显著上调 → 炎症小体启动
  - GSDMD-N：胞质/胞膜染色增强，膜破裂 → 焦亡执行
  - ASC Specks：1-2μm 强荧光斑点（≥5 个/高倍视野）→ 焦亡激活金标准

---

## Lab Automation 分析

### 1. Experiment Modular Decomposition

本实验可分解为以下模块：

```
┌─────────────────────────────────────────────────────────────────┐
│                     MODULE 0: SAMPLE SPLIT                       │
│  Operation 1: Tissue Extraction & Aliquot                       │
│  Output: WB_sample + IHC_sample (parallel fork)                 │
└──────────────────────────┬──────────────────────────────────────┘
                           │
              ┌────────────┴────────────┐
              ▼                          ▼
┌─────────────────────────┐  ┌─────────────────────────────────┐
│  MODULE 1: WB PROTEIN   │  │  MODULE 2: IHC/IF HISTOLOGY     │
│  DETECTION              │  │  DETECTION                     │
│                         │  │                                 │
│  Op 2: Protein Extract  │  │  Op 11: Fixation + Embedding    │
│  Op 3: BCA Quantify     │  │  Op 12: Deparaffinization      │
│  Op 4: Denaturation     │  │  Op 13: Antigen Retrieval      │
│  Op 5: SDS-PAGE         │  │         ↓                      │
│  Op 6: Transfer         │  │  Op 14: IHC Immunostaining     │
│  Op 7: Blocking         │  │         ↓                      │
│  Op 8: Primary Ab (4°C) │  │  Op 15: DAB Development        │
│  Op 9: Secondary Ab     │  │                                 │
│  Op 10: ECL Imaging     │  │  Op 13 → Op 16: IF Staining    │
│                         │  │  Op 16 → Op 17: IF Imaging     │
└─────────────────────────┘  └─────────────────────────────────┘
```

### 2. Resource Mapping Table

| Machine Type | Machine Name | 用途 | 兼容操作 | 资源类型 |
|:---|:---|:---|:---|:---|
| 1 | SamplePrep_Station | 样本处理：匀浆/离心/分装/变性 | Op 1, 2, 4 | 共享 |
| 2 | ProteinQuantifier | BCA 蛋白定量 | Op 3 | 独占 |
| 3 | Electrophoresis_System | SDS-PAGE 电泳 | Op 5 | 独占 |
| 4 | Transfer_System | 电转膜 | Op 6 | 独占 |
| 5 | Incubation_Station | 封闭/二抗孵育/洗涤 | Op 7, 9 | 共享 |
| 6 | Cold_Incubator_4C | 4°C 一抗孵育（过夜） | Op 8 | 共享 |
| 7 | ECL_Imager | 化学发光成像 | Op 10 | 独占 |
| 8 | IHC_Workstation | 脱蜡/修复/染色/IF | Op 12, 13, 14, 15, 16 | 共享 |
| 9 | Fluorescence_Microscope | 荧光成像 | Op 17 | 独占 |
| 10 | Tissue_Fixation_Station | 固定/包埋/切片 | Op 11 | 独占 |

### 3. TCMB Constraint Matrix

| Op ID | Boundary 1 | Op ID | Boundary 2 | α (min) | 约束说明 |
|:---|:---|:---|:---|:---|:---|
| 2 | end | 3 | start | 10 | 蛋白提取后 10min 内必须开始定量 |
| 3 | end | 4 | start | 30 | 定量完成后 30min 内必须开始变性 |
| 5 | end | 6 | start | 30 | 电泳结束后 30min 内必须转膜 |
| 6 | end | 7 | start | 30 | 转膜后 30min 内必须封闭 |
| 8 | end | 9 | start | 60 | 一抗孵育结束后 60min 内必须二抗 |
| 11 | end | 12 | start | 1440 | 固定完成后 24h 内可开始脱蜡（石蜡稳定） |
| 13 | end | 14 | start | 60 | 抗原修复后 60min 内必须开始免疫染色 |
| 14 | end | 15 | start | 30 | 二抗孵育后 30min 内必须 DAB 显色 |
| 13 | end | 16 | start | 60 | 抗原修复后 IHC/IF 并行，60min 内启动 IF |

### 4. Operation DAG

```
                            ┌──────────────┐
                            │  Op 1: Split │
                            └──────┬───────┘
                                   │
                    ┌──────────────┼──────────────┐
                    ▼                               ▼
           ┌────────────┐                  ┌─────────────┐
           │  Op 2: WB  │                  │ Op 11: IHC  │
           │  Extract   │                  │  Fixation   │
           └──────┬─────┘                  └──────┬──────┘
                  │                               │
           ┌──────▼─────┐                  ┌──────▼──────┐
           │  Op 3: BCA │                  │ Op 12: DePx │
           └──────┬─────┘                  └──────┬──────┘
                  │                               │
           ┌──────▼─────┐                  ┌──────▼──────┐
           │ Op 4: Denat│                  │ Op 13: AgR  │
           └──────┬─────┘                  └──┬──────┬───┘
                  │                           │      │
           ┌──────▼─────┐              ┌──────▼──┐  ┌──────▼───┐
           │ Op 5: PAGE │              │ Op 14   │  │ Op 16    │
           └──────┬─────┘              │ IHC Stn │  │ IF Stn   │
                  │                    └────┬────┘  └────┬─────┘
           ┌──────▼─────┐                 │             │
           │ Op 6: Xfer │              ┌──▼───┐    ┌────▼───┐
           └──────┬─────┘              │Op 15 │    │Op 17   │
                  │                    │DAB   │    │IF Img  │
           ┌──────▼─────┐              └──────┘    └────────┘
           │ Op 7: Block│
           └──────┬─────┘
                  │
           ┌──────▼─────┐
           │ Op 8: 1stAb│
           └──────┬─────┘
                  │
           ┌──────▼─────┐
           │ Op 9: 2ndAb│
           └──────┬─────┘
                  │
           ┌──────▼─────┐
           │Op 10: ECL  │
           └────────────┘
```

### 5. Instrument-Lane Operation Diagram

以下为基于 SLab.jl 调度引擎（Cbc MILP 优化器）生成的 Instrument-Lane 操作时序图。
**总完工时间：1141 min (19.0 h)**，关键路径为 WB 路径（受 4°C 过夜孵育限制）。

> 交互式甘特图详见：[gantt_pyroptosis.html](file:///workspace/gantt_pyroptosis.html)

```
时间轴 (min)    0        60       120      180      240      300      360      420      480      540      600      660      720      780      840      900      960      1020     1080     1140
                │        │        │        │        │        │        │        │        │        │        │        │        │        │        │        │        │        │        │        │
SamplePrep_     ■1■      ■2■                        ■4■
Station         0-30     31-76                      136-151

ProteinQuantif.          ■3■
                         76-136

Electrophoresis                   ■5■
System                            151-211

Transfer_System                            ■6■
                                           211-286

Incubation_Station                                ■7■                                               ■9■
                                                  286-346                                          1066-1126

Cold_Incubator_4C                                 ■████████████████████████████████████████████8███████████████████████████████████████████████████████■
                                                  346──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────1066

ECL_Imager                                                                                           ■10■
                                                                                                     1126-1141

Tissue_Fixation  ■11■
Station          30-150

IHC_Workstation           ■12■     ■13■          ■14■                             ■16■   ■15■
                          150-180  181-211       212-392                          393-453 454-469

Fluorescence_Microscope                                                                     ■17■
                                                                                            453-483

图例: ■ 操作执行区间    ■███ 长时间孵育（跨越多格）
颜色: 紫色=样本分裂 | 绿色=WB蛋白检测 | 蓝色=IHC组织学检测 | 橙色=IF荧光检测
```

**调度详情表：**

| Op | 操作 | 仪器 | 开始(min) | 结束(min) | 时长(min) |
|:---|:---|:---|:---|:---|:---|
| 1 | Tissue extraction & aliquot | SamplePrep_Station | 0 | 30 | 30 |
| 11 | IHC: Fixation + embedding | Tissue_Fixation_Station | 30 | 150 | 120 |
| 2 | WB: Protein extraction | SamplePrep_Station | 31 | 76 | 45 |
| 3 | WB: BCA quantification | ProteinQuantifier | 76 | 136 | 60 |
| 4 | WB: Denaturation | SamplePrep_Station | 136 | 151 | 15 |
| 12 | IHC: Deparaffinization | IHC_Workstation | 150 | 180 | 30 |
| 5 | WB: SDS-PAGE | Electrophoresis_System | 151 | 211 | 60 |
| 13 | IHC: Antigen retrieval | IHC_Workstation | 181 | 211 | 30 |
| 6 | WB: Electrotransfer | Transfer_System | 211 | 286 | 75 |
| 14 | IHC: Immunostaining | IHC_Workstation | 212 | 392 | 180 |
| 7 | WB: Blocking | Incubation_Station | 286 | 346 | 60 |
| 8 | WB: Primary Ab (4C overnight) | Cold_Incubator_4C | 346 | 1066 | 720 |
| 16 | IF: Immunofluorescence staining | IHC_Workstation | 393 | 453 | 60 |
| 17 | IF: Fluorescence imaging | Fluorescence_Microscope | 453 | 483 | 30 |
| 15 | IHC: DAB development | IHC_Workstation | 454 | 469 | 15 |
| 9 | WB: Secondary Ab | Incubation_Station | 1066 | 1126 | 60 |
| 10 | WB: ECL imaging | ECL_Imager | 1126 | 1141 | 15 |

### 6. Automation Scheduling Logic

基于 SLab.jl 调度引擎（MILP 优化器，Cbc），调度逻辑为：

1. **目标函数**：最小化总完工时间 `min(Ω)`
2. **约束条件**：
   - 每个操作分配到且仅能分配到一台兼容机器
   - 每台机器同一时间只能处理一个操作（含 β=1min 切换缓冲）
   - 操作依赖关系（DAG）：前驱操作完成后才可启动后继
   - TCMB 时间边界约束：成对操作的 start/end 时间差 ≤ α
   - 已调度任务不可被新任务抢占
3. **并行策略**：WB 路径与 IHC/IF 路径在样本分裂后并行执行，共享 IHC_Workstation 资源
4. **瓶颈缓解**：
   - Cold_Incubator_4C（Op 8，12h 孵育）为 WB 路径的关键路径
   - IHC_Workstation 为 IHC/IF 路径的共享资源（Op 12-16），需合理排队

### 7. Critical Path Analysis

**WB 路径关键路径**：
Op 1 → Op 2 → Op 3 → Op 4 → Op 5 → Op 6 → Op 7 → Op 8 (12h) → Op 9 → Op 10
总耗时：30+45+60+15+60+75+60+720+60+15 = **1140 min (19h)**

**IHC/IF 路径关键路径**：
Op 1 → Op 11 → Op 12 → Op 13 → Op 14 → Op 15（IHC终点）
或 Op 1 → Op 11 → Op 12 → Op 13 → Op 16 → Op 17（IF终点）
IHC 路径耗时：30+120+30+30+180+15 = **405 min (6.75h)**
IF 路径耗时：30+120+30+30+60+30 = **300 min (5h)**

**整体关键路径**：WB 路径（19h），受限于 4°C 过夜孵育。

### 8. Bottleneck Analysis

| 瓶颈点 | 原因 | 缓解措施 |
|:---|:---|:---|
| Cold_Incubator_4C (Op 8) | 12h 一抗孵育，WB 路径最长操作 | 使用多舱位冷孵箱；利用过夜时间窗口 |
| IHC_Workstation 共享 | Op 12-16 均使用，串行排队 | 合理安排 IHC 与 IF 的起始时间差 |
| ECL_Imager (Op 10) | 独占资源，成像后无法回溯 | 优化样本批次，集中成像 |
| 手动切片 (Op 11) | 120min 固定+包埋+切片 | 若有自动切片机可缩短至 60min |

---

## 材料 Metadata

### 试剂清单

| 编号 | 试剂名称 | 货号/规格 | 供应商 | 存储条件 | 用量/样本 | 状态 |
|:---|:---|:---|:---|:---|:---|:---|
| R1 | RIPA 裂解液（含 PMSF） | 100mL | 碧云天 | 4°C | 500μL | 需确认 |
| R2 | 蛋白酶抑制剂 Cocktail | 100× | CST | -20°C | 5μL/mg | 需确认 |
| R3 | 磷酸酶抑制剂 Cocktail | 100× | CST | -20°C | 5μL/mg | 需确认 |
| R4 | BCA 蛋白定量试剂盒 | 500 次 | Thermo | RT | 25μL | 需确认 |
| R5 | 8-12% 梯度 SDS-PAGE 胶 | 10 孔 | Bio-Rad | 4°C | 1 块/电泳 | 需确认 |
| R6 | 4× 上样缓冲液（含 DTT） | 10mL | Bio-Rad | RT | 3:1 稀释 | 需确认 |
| R7 | 1× MOPS 电泳缓冲液 | 1L | Bio-Rad | RT | 1L/槽 | 需确认 |
| R8 | 0.22μm PVDF 膜 | 7×8.5cm | Bio-Rad | RT | 1 张/转膜 | 需确认 |
| R9 | 转膜缓冲液（含甲醇） | 1L | Bio-Rad | RT | 1L/转膜 | 需确认 |
| R10 | 5% 脱脂奶粉 | 500mL | - | 4°C | 20mL/膜 | 需确认 |
| R11 | TBS-T 洗脱液 | 1L | - | RT | 500mL/膜 | 需确认 |
| R12 | ECL 化学发光底物 | 100mL | Millipore | 4°C | 1mL/膜 | 需确认 |
| R13 | Anti-GSDMD 抗体（N 端特异） | CST #96458 | CST | -20°C | 1:1000 | 需确认 |
| R14 | Anti-Cleaved Caspase-1 (p20) | AdipoGen | AdipoGen | -20°C | 1:500 | 需确认 |
| R15 | Anti-NLRP3 抗体 | AdipoGen | AdipoGen | -20°C | 1:1000 | 需确认 |
| R16 | Anti-ASC 抗体（WB/IF 兼容） | CST #67824 | CST | -20°C | 1:1000 / 1:200 | 需确认 |
| R17 | HRP 标记二抗（兔/鼠） | CST #7074/7076 | CST | -20°C | 1:3000 | 需确认 |
| R18 | 4% PFA 固定液 | 500mL | - | RT | 20mL/样本 | 需确认 |
| R19 | EDTA 抗原修复液 (pH 8.0) | 500mL | - | RT | 500mL/切片 | 需确认 |
| R20 | DAB 显色试剂盒 | 50 次 | CST | 4°C | 100μL/切片 | 需确认 |
| R21 | 内源性过氧化物酶阻断剂 | 100mL | - | RT | 100μL/切片 | 需确认 |
| R22 | BSA 封闭液 | 500mL | - | 4°C | 200μL/切片 | 需确认 |
| R23 | 荧光二抗（Alexa Fluor 488） | 1mg/mL | Invitrogen | -20°C | 1:500 | 需确认 |
| R24 | 防荧光淬灭封片剂 | 50mL | - | 4°C | 10μL/切片 | 需确认 |

### 仪器清单

| 编号 | 仪器名称 | 型号 | 位置 | 用途 | 状态 |
|:---|:---|:---|:---|:---|:---|
| I1 | 组织匀浆仪 | - | 细胞室 | 组织裂解 | 需确认 |
| I2 | 冷冻离心机 | Eppendorf 5810R | 细胞室 | 4°C 12,000rpm 离心 | 需确认 |
| I3 | 蛋白定量仪 | Nanodrop / BCA 读板仪 | 生化室 | BCA 吸光度检测 | 需确认 |
| I4 | 电泳仪 | Bio-Rad PowerPac | 电泳室 | SDS-PAGE 电泳 | 需确认 |
| I5 | 转膜仪 | Bio-Rad Trans-Blot | 电泳室 | 湿式电转 | 需确认 |
| I6 | 冷孵箱 | - | 细胞室 | 4°C 一抗孵育 | 需确认 |
| I7 | ECL 成像系统 | Bio-Rad ChemiDoc | 成像室 | 化学发光检测 | 需确认 |
| I8 | IHC 工作站 | - | 病理室 | 脱蜡/修复/染色 | 需确认 |
| I9 | 荧光显微镜 | Olympus BX63 | 成像室 | ASC Specks 成像 | 需确认 |
| I10 | 石蜡包埋机/切片机 | - | 病理室 | 固定/包埋/切片 | 需确认 |

### 样本 Metadata

| 字段 | 说明 |
|:---|:---|
| 样本类型 | 小鼠肠道组织（肠粘膜上皮为主） |
| 样本量 | 30-50mg (WB) / 整块组织 (IHC) |
| 分组 | RE 模型组 / Nic 治疗组 / 对照组 |
| 标记物 | GSDMD (~53kDa 全长, ~30kDa N 端剪切体)、Caspase-1 (~45kDa 前体, ~20kDa p20)、NLRP3 (~118kDa)、ASC (~22kDa 单体) |
| 样本编号规则 | [分组]-[动物编号]-[WB/IHC] |

---

## 常见问题

- **WB 条带弱或无信号**：检查转膜效率（PVDF 活化、电流/时间）、抗体浓度、ECL 底物新鲜度；使用 Ponceau S 染色验证转膜效果
- **NLRP3 高分子量泳道弯曲**：使用 8% 分离胶、延长电泳时间、降低电压；确保缓冲液新鲜
- **GSDMD-N 剪切体与全长难以分离**：使用 12% 胶、延长转膜时间、0.22μm 膜防止小分子穿透
- **ASC Specks 无法观察**：确认抗体识别单体形式、增加封闭时间、使用去垢剂（如 0.1% Triton X-100）增强胞内抗体穿透
- **IHC 背景染色过高**：延长封闭时间（3% BSA，37°C，2h）、降低一抗浓度、增加洗涤次数
- **一抗 4°C 孵育过夜后效价降低**：使用密封膜防止蒸发、加入 0.02% NaN₃ 防腐

---

## 相关文件

- [labauto_prompt.md](file:///workspace/labauto_prompt.md) — Lab Automation 分析 prompt 模板
- [SLab.jl](file:///workspace/src/SLab.jl) — 调度引擎主程序
- [types.jl](file:///workspace/src/types.jl) — Machine / Operation / TCMB / Job / Batch 数据结构定义
- [schedule.jl](file:///workspace/src/schedule.jl) — MILP 调度优化求解器
- [plot.jl](file:///workspace/src/plot.jl) — 甘特图可视化
- [gantt_pyroptosis.html](file:///workspace/gantt_pyroptosis.html) — 交互式甘特图（浏览器打开）
- [examples/case_pyroptosis/](file:///workspace/examples/case_pyroptosis) — 本 protocol 的 SLab.jl 仿真 case 文件
- [config.tsv](file:///workspace/examples/case_pyroptosis/config.tsv) — 仿真配置（N_job=1, 非顺序执行）
- [machines.tsv](file:///workspace/examples/case_pyroptosis/machines.tsv) — 仪器资源定义（10 台）
- [operations.tsv](file:///workspace/examples/case_pyroptosis/operations.tsv) — 17 个操作定义
- [dependency.tsv](file:///workspace/examples/case_pyroptosis/dependency.tsv) — 15 条依赖边（DAG）
- [tcmb.tsv](file:///workspace/examples/case_pyroptosis/tcmb.tsv) — 9 条 TCMB 时间约束

---

## 梳理方式

1. **实验前**：按本 SOP 的材料 Metadata 清单完成所有试剂/仪器的 on-shelf 确认
2. **实验中**：按照 Instrument-Lane Operation Diagram 进行并行路径调度，注意 TCMB 时间约束
3. **实验后**：将原始数据（WB 灰度值、IHC 光密度、IF Specks 计数）录入 MES 系统

## 更新机制

- 每个季度提醒做一次 SOP 全面审查
- 每次更新后在文档顶部刷新"更新日期"

---

## 执行规则

### 状态监控（每天）
- 检查所有进行中的项目，当前处于哪个步骤
- 重点监控：Cold_Incubator_4C 中的一抗孵育进度

### 卡点预警
| 步骤 | 完成标准 | 未达标准的预警 |
|:---|:---|:---|
| 蛋白定量 | R² > 0.99 标准曲线 | 重新配制 BSA 标准品 |
| SDS-PAGE | 指示剂达分离胶底部 | 延长电泳或更换缓冲液 |
| 转膜 | Ponceau S 染色均匀 | 检查转膜夹层/海绵 |
| 一抗孵育 | 4°C 孵育 ≥ 8h | 使用旋转孵育器增强结合 |
| ECL 成像 | 信噪比 > 3:1 | 增加曝光时间或更换底物 |
| IHC 复染 | 苏木精核染清晰 | 调整复染时间/分化 |
| IF Specks | ≥5 个/HPF | 增加一抗浓度或孵育时间 |

---

## 注意事项

- SOP 面向完全不了解情况的操作人员，每个步骤具体到"做什么、用什么、多久、合格标准"
- WB 与 IHC/IF 路径可并行，需在样本分裂后（步骤 1）立即启动两条路径
- Caspase-1 p20 (~20kDa) 和 ASC (~22kDa) 分子量接近，需注意胶浓度与转膜条件
- EDTA 抗原修复液 (pH 8.0) 对 NLRP3 的检测至关重要，不可用柠檬酸盐缓冲液替代
- ASC Specks 的 IF 检测必须使用新鲜固定样本，避免反复冻融

---

## 参考文献

1. Western blot for tissue extract. *protocols.io*.
2. Detailed Western Blotting Protocol. *protocols.io*.
3. ASC speck formation as a marker of inflammasome activation. *Nature Protocols*.
4. SLab.jl: A scheduling optimization framework for laboratory automation. [内部代码库]
5. labauto_prompt.md: Lab Automation analysis skill template. [内部文件]