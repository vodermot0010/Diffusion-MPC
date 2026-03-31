# Diffusion-Planner 中文技术报告设计

## 目标

生成一份以中文为主的技术报告式 PDF，面向熟悉自动驾驶规划、轨迹预测、扩散模型和优化控制的读者。报告需要同时覆盖：

- 论文《Diffusion-Based Planning for Autonomous Driving with Flexible Guidance》的方法与实验结论
- 当前 `/data/wyf/lgq/Diffusion-Planner` 仓库中的真实实现
- 论文方法与仓库实现之间的对应关系
- 用户自行新增的 MPC 扩展，而非将其误表述为论文原始内容
- Diffusion-Planner 与 MPC 的耦合方式、接口边界、运行时流程和优化问题表述

报告应优先服务于以下用途：

- 论文阅读后的系统化理解
- 代码实现对照阅读
- 复现与二次开发
- 对外技术汇报或组内分享

## 交付物

最终交付物包括：

- 一份排版完整的 PDF 报告
- 报告源文件，保存在仓库内，便于后续修改
- 生成 PDF 所需的脚本或命令

输出路径约定：

- 源文件放在 `Diffusion-Planner/output/pdf/` 或与之等价的稳定目录
- 最终 PDF 使用稳定、明确的文件名，例如 `diffusion_planner_technical_report_zh.pdf`

## 报告风格

采用技术报告式，而不是讲义式或博客式。具体要求如下：

- 结构严格，章节边界清晰
- 对关键变量、张量、模块、接口给出明确定义
- 尽量给出公式化表述，而不是仅做直观描述
- 论文内容与代码实现严格区分
- 论文原始方法与用户新增 MPC 扩展严格区分
- 对训练、推理、闭环仿真、后处理和优化接口给出明确调用链

文风要求：

- 中文为主
- 术语尽量统一，首次出现可附英文括注
- 不追求营销式表达，避免泛泛而谈
- 明确标注“代码确认”“论文明确写出”“根据实现推断”三种信息来源层级

## 报告结构

建议采用以下章节结构：

### 1. 摘要

概述论文解决的问题、核心方法、代码实现落点，以及当前仓库相较论文新增的 MPC 扩展。

### 2. 背景与问题定义

说明 closed-loop planning 的问题背景、nuPlan 任务设定、联合建模 prediction 与 planning 的动机，以及论文为何选择 diffusion 而不是单纯 behavior cloning。

### 3. 论文方法解析

按论文逻辑解释：

- 任务重定义
- 联合未来轨迹生成目标
- Diffusion ODE 与 score 学习
- encoder 与 decoder 的结构设计
- 历史 agent、静态物体、lane、route 的编码方式
- classifier guidance 的作用

这里需要把论文中的符号和仓库里的实际张量对应起来。

### 4. 论文到代码的映射

把论文各个模块映射到当前仓库中的关键文件、类、函数和脚本，包括：

- 数据预处理
- 数据集读取
- normalizer
- encoder / decoder / DiT
- loss
- training loop
- simulation planner
- guidance

这一部分要求读者可以顺着报告直接定位到代码。

### 5. 训练与推理链路

分别说明：

- 离线数据预处理链路
- 训练阶段前向、加噪、loss 计算与参数更新链路
- 推理阶段 DPM-Solver 采样链路
- 闭环仿真阶段 planner 接口与轨迹回写链路

需要强调 train/eval 差异。

### 6. 论文中的 Guidance 机制

说明 collision guidance、drivable guidance、target speed、comfort 等能量函数的思想，并明确：

- 当前仓库里实际实现了什么
- 哪些能量函数只在论文中描述、仓库里未完整落地

### 7. 用户新增 MPC 扩展

这一章必须明确写出：

- MPC 不是论文原始方法的一部分
- MPC 是在当前仓库中作为后处理 refinement 模块引入
- 当前仓库中存在 candidate sampling 与 MPC refine 两层扩展

并系统说明：

- 为什么要引入 MPC
- 为什么不是把 MPC 融进训练，而是作为推理后处理
- 什么时候触发 candidate sampling
- 什么时候触发 MPC
- 为什么只优化少量近邻

### 8. MPC 优化问题的数学表述

这是报告的重点章节，需要给出明确的优化问题描述，包括：

- 决策变量
- 状态变量
- 控制变量
- ego bicycle-like dynamics
- neighbor simplified dynamics
- 初始条件约束
- 控制上下界
- 速度上下界
- 安全间距软约束及 slack variable
- tracking / heading / speed / smooth / control / progress / safety / neighbor prior 等代价项
- 激活条件
- refinement 接受条件
- blending 机制

需要结合 `mpc_refiner.py` 的真实实现，而不是抽象地描述一个通用 MPC。

### 9. Diffusion 与 MPC 的耦合方式

这一章要讲清楚接口与流程：

- Diffusion 输出什么
- MPC 读取哪些量作为参考轨迹与初值
- ego / neighbor 当前状态如何从模型输入构造
- static obstacle 在当前实现中参与了哪些 gating，而不是直接进入优化变量
- refined 结果如何写回 `outputs["prediction"]`
- 哪些步骤发生在 ego-centric local frame
- 为什么最终仍然由 nuPlan 的 `transform_predictions_to_states` 转回全局轨迹

### 10. 与论文原始方法的差异

明确区分以下三层：

- 论文明确提出的方法
- 官方仓库当前最小实现
- 当前本地仓库中的增强实现

重点列出：

- candidate sampling
- MPC refiner
- worker / memory tuning shell script
- 任何与论文不完全一致的工程化修改

### 11. 复现要点与风险

整理最容易踩坑的点，包括：

- 坐标系
- cos/sin heading 表示
- future horizon 与 current state 拼接方式
- normalization
- neighbor mask
- route lane 的构造
- diffusion sampling 温度 / 步数与论文描述的差异
- 闭环指标与开放环指标差异
- 论文中未充分写清、但代码里决定行为的实现细节

### 12. 结论

总结论文方法价值、代码实现特点，以及新增 MPC 扩展带来的收益与代价。

## 信息来源策略

报告中的每一类结论都应区分来源：

- 论文直接陈述
- 仓库源码直接确认
- 基于代码和论文联合推断

若论文与代码存在偏差，需要明确写出偏差点，不能混写。

## 关键技术要求

### 论文部分

- 需要基于 PDF 原文提取方法、实验、附录中的关键实现细节
- 特别纳入训练细节、推理细节、guidance 细节和附录中的超参数

### 代码部分

- 需要基于当前仓库真实文件逐一确认关键模块
- 文件引用应精确到路径，必要时带行号

### MPC 部分

- 必须根据 `diffusion_planner/optimization/mpc_refiner.py` 与 `diffusion_planner/planner/planner.py` 真实实现书写
- 不得把 static obstacles 描述成已经进入优化约束，如果代码只把它们用于 gating，就应照实描述
- 必须说明 candidate sampling 与 MPC 不是同一层机制

## 版式要求

PDF 需要具备较强可读性：

- 标题、节标题、公式、表格、代码路径引用格式统一
- 对复杂流程可加入简洁框图或流程图
- 对优化问题可使用标准数学排版
- 页边距、字体大小、段间距、页眉页脚应稳定

如果生成过程允许，优先采用支持中文与数学公式的排版方式，保证：

- 中文不乱码
- 公式清晰
- 路径与代码标识可读

## 实现计划

实施阶段按以下顺序执行：

1. 继续读取论文 PDF 的关键章节与附录
2. 复核当前仓库关键实现文件
3. 整理结构化中文报告正文
4. 生成 PDF 源文件
5. 渲染 PDF
6. 渲染检查与版式修正
7. 交付最终 PDF 与源文件路径

## 风险与边界

- 论文中的某些 guidance 能量函数可能未在当前仓库完整实现，报告必须显式说明
- 本地仓库包含用户自定义增强模块，报告必须避免误归因给论文作者
- 如果当前环境缺少中文 PDF 渲染依赖，需要退化到可用的本地生成方式，但仍应优先保证内容正确和结构完整

## 完成判定

以下条件全部满足后，任务才视为完成：

- 已生成中文技术报告式 PDF
- 报告中已清楚区分论文原始方法与用户新增 MPC
- 报告中已清楚写出 MPC 优化问题与 Diffusion-MPC 耦合关系
- 已对 PDF 做基本渲染检查
- 已向用户提供最终文件路径
