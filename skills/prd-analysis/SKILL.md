---
name: prd-analysis
description: Use when the task is to analyze a feature request or requirements document and write a concise, product-facing PRD to .dev/prd_doc.md covering only core features, page structure, user interaction logic, color direction, UI, UX, and assumptions.
---

# PRD Analysis

在当前任务是“分析功能需求文档，并整理成产品视角的 PRD”时使用这个 skill。

适用范围：

- 将零散需求整理成清晰 PRD
- 将功能说明收敛为产品层面的页面与交互设计
- 为后续设计或开发输出统一的需求文档

不适用范围：

- 编写代码、技术方案、系统架构或接口设计
- 输出数据库设计、API 约定、状态管理方案或组件拆分方案
- 输出高保真视觉稿

## 输出目标

将输入的需求、想法或功能描述，整理为一份简洁、完整、面向产品的 Markdown 文档，写入 `.dev/prd_doc.md`。

PRD 必须是产品文档，不允许混入实现建议。

## 强制规则

- The PRD must stay product-facing only. No code, architecture, APIs, or implementation advice.
- Write concise Markdown to `.dev/prd_doc.md`.
- Cover only:
  - `核心功能点`
  - `页面结构`
  - `用户交互逻辑`
  - `配色`
  - `UI`
  - `UX`
- Use app constraints when present, especially login, languages, backend requirement, and platform scope.
- If a requirement is missing, make the smallest useful assumption and put it under `假设`.
- Treat the app as a complete product, not an MVP or demo.
- Prefer lightweight copy and compact interfaces.

## 工作流程

1. 先完整读取需求输入，提取产品目标、目标用户、主流程、关键限制和平台范围。
2. 识别显式约束与隐式约束，尤其是登录方式、语言要求、后端依赖、运行平台和设备范围。
3. 只保留产品层面的信息，不写任何实现层内容。
4. 将需求压缩为规定结构的 PRD，并写入 `.dev/prd_doc.md`。
5. 如果存在信息缺口，只做最小有用假设，并明确放在 `假设` 下。

## 输出结构

产出的 `.dev/prd_doc.md` 只允许包含以下部分：

- `假设`
- `核心功能点`
- `页面结构`
- `用户交互逻辑`
- `配色`
- `UI`
- `UX`

不要添加其他章节，除非用户明确要求。

## 各部分要求

### 假设

- 只写需求中缺失、但为了形成完整产品描述所必需的最小假设
- 假设必须谨慎、克制，不要扩展成额外功能
- 假设应该服务于产品闭环，而不是技术实现

### 核心功能点

- 只保留真正影响产品价值和用户流程的功能
- 用产品语言表达，不要写接口、字段、服务或工程细节
- 站在完整产品视角描述，不要把产品降级成 MVP、demo 或占位方案

### 页面结构

- 列出用户会接触到的完整页面或关键视图
- 页面命名要清晰，体现其在流程中的作用
- 如果需求隐含了登录页、结果页、空状态页、详情页或确认步骤，需要主动补齐

### 用户交互逻辑

- 描述用户从进入产品到完成任务的主要操作链路
- 说明关键跳转、状态变化和反馈机制
- 强调主操作与次操作，不要写实现方式

### 配色

- 只描述产品层面的配色方向和主要颜色角色
- 没有品牌规范时，给出合理且克制的颜色建议
- 配色应服务于信息层级、操作强调和整体气质

### UI

- 描述整体界面风格、信息密度、布局倾向和视觉层次
- 优先强调用户真正会感知到的界面特征
- 文案应尽量轻量，界面应偏紧凑，避免冗长说明

### UX

- 描述如何降低认知负担、减少误操作、提升主流程完成效率
- 说明页面间衔接、反馈清晰度和操作节奏
- 强调完整产品体验，而不是功能堆砌

## 写作要求

- 用简洁 Markdown，不写长篇背景铺垫
- 不要写“建议使用”“技术上可采用”“后续可以扩展”这类实现导向内容
- 不要出现代码、数据库、接口、表结构、模块划分、前后端分层、组件命名等内容
- 如果需求带有明确约束，必须体现在 PRD 中，而不是忽略
- 默认文案轻量、界面紧凑、结构清晰

## 自检清单

- 是否已经写入 `.dev/prd_doc.md`
- 是否严格保持为产品文档
- 是否只包含规定的几个部分
- 是否吸收了登录、语言、后端要求、平台范围等约束
- 是否把缺失信息控制为最小假设
- 是否把产品当作完整产品，而不是 demo
