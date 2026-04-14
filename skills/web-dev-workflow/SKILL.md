---
name: web-dev-workflow
description: Multi-agent web development workflow. Use when a web app should be delivered through four required subagents for PRD, frontend design, code development, and review, with Ignis backend/deploy and a Vue 3 plus Vite frontend stack.
---

# Web Dev Workflow

在当前任务是“交付一个完整 web 产品，并且必须通过多 agent 协作完成 PRD、前端设计、代码开发和 review”时使用这个 skill。

## 核心原则

- 这个 workflow 必须使用子 agent。
- `prd`、`frontend-design`、`code-dev`、`review` 这 4 个工作都必须通过子 agent 完成。
- 父 agent 只负责分阶段编排、传递工件、检查阶段结果是否满足进入下一阶段，不直接代写 PRD、不直接代写设计、不直接代写主要代码、不直接代做最终 review。

## 必须使用的 skills

- `prd-analysis`
- `frontend-design-spec`
- `vue-frontend-dev`
- `ignis`
- 如有登录需求，再加 `ignis-login`

## 产物约定

- PRD：`.dev/prd_doc.md`
- 前端设计文档：`.dev/frontend_design_doc.md`
- Review 报告：`.dev/review_report.md`

如果仓库已有更强约束，以仓库已有约束为准，但默认沿用以上路径。

## 子 Agent 分工

### 1. PRD Agent

职责：

- 读取原始需求
- 使用 `prd-analysis`
- 输出产品视角 PRD 到 `.dev/prd_doc.md`

规则：

- PRD 必须只保持 product-facing
- 不能混入代码、架构、API 或实现建议
- 只覆盖 `假设`、`核心功能点`、`页面结构`、`用户交互逻辑`、`配色`、`UI`、`UX`

### 2. Frontend Design Agent

职责：

- 读取原始需求和 `.dev/prd_doc.md`
- 使用 `frontend-design-spec`
- 输出前端设计文档到 `.dev/frontend_design_doc.md`

规则：

- 设计文档负责把 PRD 展开成页面级说明
- 必须补齐完整页面清单、关键状态、详细布局、视觉层级和最少文案
- 不要回退去改写 PRD 的产品边界，除非父 agent 明确要求返工

### 3. Code Dev Agent

职责：

- 读取原始需求、`.dev/prd_doc.md`、`.dev/frontend_design_doc.md`
- 负责前后端实现、联调、构建、部署

必须加载的 skills：

- 前端：`vue-frontend-dev`
- 后端与部署：`ignis`
- 登录需求存在时：`ignis-login`

规则：

- 前端默认技术栈：
  - `Vue 3`
  - `Vite`
  - `vue-router`
  - `Pinia`
  - `vue-i18n`
  - `Tailwind CSS v4`
  - `Vite SSR entry + prerender script`
- 后端和部署默认走 `ignis` 的发布链路，不把本地 `dev` 当主发布模型
- 遇到登录需求时，登录链路按 `ignis-login` 约束实现，不自行发明另一套认证模型
- 后端 API、`ignis.hcl`、SDK 用法、部署命令都要以 `ignis` / `ignis-login` 的文档为准，不要猜
- 如果需要登录联调，可先用 `test_password` 完成 smoke test，但正式发布前应移除测试 provider

### 4. Review Agent

职责：

- 在代码开发完成后独立审查当前结果
- 检查需求覆盖、设计还原、关键流程、构建 / 部署风险和回归风险
- 输出 review 结果到 `.dev/review_report.md`

规则：

- 以 code review mindset 为主，优先找问题、风险、漏项和行为回归
- findings 必须优先于总结
- 如果没有发现问题，也要明确写出“无关键 findings”和剩余风险 / 测试缺口
- review agent 不负责重写代码；它只输出审查结论和修改建议

## 推荐执行顺序

1. 父 agent 启动 `prd` 子 agent。
2. 等 `.dev/prd_doc.md` 完成后，再启动 `frontend-design` 子 agent。
3. 等 `.dev/frontend_design_doc.md` 完成后，启动 `code-dev` 子 agent。
4. 代码、构建和部署达到可审查状态后，启动 `review` 子 agent。
5. 如果 review 有关键问题，父 agent 把 `.dev/review_report.md` 交回新的或继续使用中的 `code-dev` 子 agent 修复。
6. 修复后重新启动 `review` 子 agent，直到没有阻塞性交付问题，或明确说明剩余风险。

## 父 Agent 编排规则

- 父 agent 需要显式告诉每个子 agent 它的唯一职责和输入工件。
- 不要把所有阶段揉进一个子 agent。
- 不要让 `review` 子 agent 同时承担开发职责。
- 不要让 `frontend-design` 子 agent 直接开始写代码。
- PRD 和设计文档是代码开发的前置工件；如果两者缺失，不应直接进入实现。
- 如果需求变更导致 PRD 失效，应先回到 `prd` 或 `frontend-design` 阶段更新工件，再继续开发。

## 给子 Agent 的输入建议

### PRD Agent 输入

- 原始需求
- 平台范围、语言范围、登录要求、后端约束等已知限制
- 明确要求写入 `.dev/prd_doc.md`

### Frontend Design Agent 输入

- 原始需求
- `.dev/prd_doc.md`
- 明确要求写入 `.dev/frontend_design_doc.md`

### Code Dev Agent 输入

- 原始需求
- `.dev/prd_doc.md`
- `.dev/frontend_design_doc.md`
- 当前仓库代码
- 是否需要登录
- 是否需要部署

### Review Agent 输入

- 原始需求
- `.dev/prd_doc.md`
- `.dev/frontend_design_doc.md`
- 当前代码和构建 / 部署结果
- 明确要求写入 `.dev/review_report.md`

## 停止条件

满足以下条件才算 workflow 完成：

- `.dev/prd_doc.md` 已完成
- `.dev/frontend_design_doc.md` 已完成
- 代码开发已完成并达到可运行 / 可构建 / 可部署状态
- `.dev/review_report.md` 已完成
- 没有阻塞性交付问题，或剩余风险已被明确记录

## 失败处理

- 如果 PRD 不完整，退回 `prd` 子 agent 修正
- 如果设计说明不足以指导开发，退回 `frontend-design` 子 agent 补充
- 如果实现偏离 PRD 或设计，退回 `code-dev` 子 agent 修复
- 如果 review 发现高严重度问题，必须先修复再进入交付
