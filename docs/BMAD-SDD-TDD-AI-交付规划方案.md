# 基于 BMAD-METHOD + SDD (GitHub Spec Kit) + TDD 的 AI Agent 协作全流程交付规划方案

| 文档元数据 | 内容 |
|-----------|------|
| 文档编号 | DOC-PLAN-001 |
| 版本 | v0.2（已按专家评审团意见修订） |
| 状态 | 已评审-待批准 |
| 评审记录 | v0.1 → 专家评审团整合评审（6 Critical / 11 Warning / 6 Suggestion）→ v0.2 全部采纳 |
| 适用范围 | 6-10 人团队、Greenfield（绿地）项目为主；存量项目见 §十四 |
| 变更记录 | v0.1 初稿；v0.2 落实评审意见 C1-C6、W1-W11、S1-S6 |
| 评审人 | 产品/架构/开发/QA/DevOps/PMO/AI 工程 七方代表 |

---

## 〇、术语与方法定义（含来源）

本方案采用的三大方法论均为**业界真实框架**，可安装、可执行，非自定义助记符：

| 方法论 | 定义与来源 | 核心资产 |
|--------|-----------|---------|
| **BMAD-METHOD** | Breakthrough Method for Agile AI-Driven Development，开源项目 `bmad-code-org/BMAD-METHOD`，安装命令 `npx bmad-method install` | 角色化 Agent（Analyst / PM / Architect / Dev / QA / UX / Scrum Master）+ 引导式工作流（brief→PRD→架构→Epic/Story→dev→review）+ 文档分片（对抗上下文丢失） |
| **SDD / Spec Kit** | GitHub 的规范驱动开发工具包 `github/spec-kit`，CLI 为 `specify-cli`（`uvx --from git+https://github.com/github/spec-kit.git specify init`） | 项目宪法 `constitution.md` + 工件链 `spec.md / plan.md / tasks.md` + 命令链 `constitution → specify → clarify → plan → tasks → analyze → implement → converge` |
| **TDD** | 测试驱动开发，Red-Green-Refactor 循环（Kent Beck） | 分层测试金字塔（单元/集成/E2E） |

**AI 运行时分工原则（解决"双头指挥"问题）**：
- **BMAD-METHOD 与 Spec Kit 承担"流程与 persona 层"**：定义做什么、按什么顺序、产出什么工件、由哪个角色视角生成；
- **Qoder / Codex 承担"执行运行时层"**：在 human 指令下消费工件、生成代码/测试/文档；
- 冲突时以本方案 §八"AI Agent 治理矩阵"为准，**Agent 不得修改已锁定（baseline tag）的规格与契约文件**，只能提 proposal PR。

**术语统一口径（全文一致）**：
- **规格 (Spec)**：需求与行为的正式描述，SSOT；**契约 (Contract)**：接口/数据模型的可机读定义（OpenAPI/JSON Schema）；
- **门禁 (Gate)**：阶段出口的自动/人工判定条件，编号 `G-PhaseN`；**决策门 (Decision Gate)**：变更请求的 BMAD 评审入口；
- **SRS**：需求规格说明书（可由 `spec.md` 汇编生成）；**PRD**：产品需求文档（BMAD `bmad-prd` 产物）；
- **AC**：验收标准，从属于 FR/US（每条 FR ≥1 条 AC）；**TC**：测试用例，从属于 AC。

---

## 一、总体框架概览

```
┌────────────────────────────────────────────────────────────────────────┐
│      Axure 在线原型（输入证据，不可直接修改） + 非 UI 需求补源 §四.4      │
└──────────────────────────────┬─────────────────────────────────────────┘
                               ▼
Phase 0 ──▶ Phase 1 ──▶ Phase 2 ──▶ Phase 3 ──▶ Phase 4 ──▶ Phase 5
输入核验     对齐(BMAD)   规格(SDD)   设计(SDD+ADR) 实现(TDD)   交付验证
   │           │            │            │            │           │
   ▼           ▼            ▼            ▼            ▼           ▼
 采集核验   constitution   spec.md     plan.md      tasks.md    验收报告
 RACI+SLA   clarify       contracts    analyze      implement   converge
 Gate:G0    Gate:G1       Gate:G2      Gate:G3      Gate:G4     Gate:G5
                               ▲                        │
                               └── 变更请求 Decision Gate ┘  ← 运营反馈闭环回流
```

**不可跳过顺序（五阶段口径）**：先对齐 → 再规格 → 后设计 → 再实现 → 终验证。渐进式导入时 Phase 3 允许**轻量化**（仅 1-2 页 ADR + 契约清单），但**不允许跳过**。

---

## 二、各阶段目标、原则、适用边界及衔接关系

### Phase 0：输入源核验（新增，DoR 硬条件）

| 维度 | 内容 |
|------|------|
| **目标** | 确认 Axure 原型可被工程化采集，显式登记信息缺口与非 UI 需求来源 |
| **原则** | ① 不核验不启动；② 缺口显式记录优于隐性遗漏 |
| **出口门禁 G0** | 见 §四.1 核验清单全部勾选或有降级方案 |

### Phase 1：BMAD 目标与决策对齐阶段

| 维度 | 内容 |
|------|------|
| **目标** | 建立项目愿景共识、明确角色职责（含 SLA）、生成项目宪法 constitution.md |
| **核心原则** | ① 先对齐再行动；② 决策记录透明可追溯；③ 每项活动**有且仅有一个** Accountable |
| **适用边界** | 项目启动、需求变更决策、跨团队冲突解决；不适用于纯技术实现细节 |
| **衔接** | 输出 `constitution.md` + RACI（含 SLA）+ 决策日志 → Phase 2 规格编写的约束输入 |

**BMAD-METHOD 在本阶段的工作流**：

| 步骤 | BMAD 工作流/Agent | 交付物 |
|------|------------------|--------|
| 背景锚定 (Why) | `bmad-agent-analyst` 引导 brainstorm | 项目 brief |
| 角色建模 (Who) | PM agent 引导 RACI 建模 + SLA 谈判 | RACI 矩阵（每行仅 1 个 A，含决策期限数值） |
| 目标对齐 (What) | `bmad-prd` 前置的目标工作坊 | OKR/成功标准清单 |
| 决策框架 (How) | Spec Kit `/speckit.constitution` 固化团队硬约束 | `constitution.md` + 决策日志模板 + Decision Gate SLA 定义 |

### Phase 2：SDD 规格建立阶段

| 维度 | 内容 |
|------|------|
| **目标** | 将原型+补源需求转化为 Spec Kit 工件（`spec.md`）与契约（OpenAPI/JSON Schema/Gherkin） |
| **核心原则** | ① `specs/` 目录即 SSOT；② 契约优先，实现滞后；③ 每条需求可机读追溯（§六） |
| **适用边界** | 接口定义、数据模型、业务规则形式化；视觉微调走原型豁免流程（§十一.3） |
| **衔接** | 输出 `spec.md`（经 clarify）+ 契约 + 用户故事 → Phase 3 设计输入 |

### Phase 3：系统设计 + 契约静态校验阶段

| 维度 | 内容 |
|------|------|
| **目标** | 架构设计、模块划分、技术选型；对契约做**静态**一致性校验（不依赖实现） |
| **核心原则** | ① 设计可追溯到规格条目；② ADR 记录权衡；③ 接口向后兼容（expand-contract） |
| **适用边界** | 系统架构、模块交互、数据流；轻量项目可压缩为 ADR 速记（不跳过） |
| **衔接** | 输出 `plan.md`（含 research/data-model/contracts）+ ADR + C4 → Phase 4 输入 |

### Phase 4：TDD 驱动实现阶段

| 维度 | 内容 |
|------|------|
| **目标** | Red-Green-Refactor 循环实现代码；每个 PR 跑消费者契约测试 |
| **核心原则** | ① 没有失败的测试就不写生产代码；② 测试即文档；③ 小步快跑 |
| **适用边界** | 业务逻辑、算法、集成层；一次性脚本可标 `TDD: relaxed`（§十二.4） |
| **衔接** | 输出通过测试的代码 + `tasks.md` 状态更新 → Phase 5 验证 |

### Phase 5：交付与运营反馈阶段

| 维度 | 内容 |
|------|------|
| **目标** | `/speckit.converge` 做规格↔实现差异收敛；UAT 前置于生产发布；建立运营反馈闭环 |
| **核心原则** | ① 交付物可追溯；② 上线即监控；③ 反馈驱动迭代回流 Phase 0/1 |
| **适用边界** | 发布部署、验收、运维、需求回流 |
| **衔接** | 输出验收报告 + 追溯矩阵 + 运维看板 + 迭代 Backlog → 下轮 Phase 0/1 |

---

## 三、Phase 间衔接与工件流转总表

| 上游 | 工件（仓库路径） | 下游消费方式 | 出口门禁 |
|------|-----------------|-------------|---------|
| Phase 0 | `docs/input-verification.md` | 决定需求采集降级方案 | G0：核验清单通过或有降级签核 |
| Phase 1 | `.specify/memory/constitution.md` | 所有后续 Agent 提示词注入 | G1：RACI 唯一 A + SLA 数值化 |
| Phase 2 | `specs/NNN-<feature>/spec.md` | 设计/任务拆解依据 | G2：clarify + analyze 无 blocking |
| Phase 2 | `specs/NNN/contracts/*.yaml`（OpenAPI/JSON Schema） | 契约静态校验 | G2：Spectral Lint 0E + oasdiff 无破坏性变更 |
| Phase 3 | `specs/NNN/plan.md` + `docs/adr/ADR-NNN.md` | tasks 拆解输入 | G3：设计评审 Checklist 通过 |
| Phase 4 | 源代码 + 测试 + `specs/NNN/tasks.md` 勾选状态 | 流水线追溯校验 | G4：追溯矩阵无孤条目 + 分层覆盖达标 |
| Phase 5 | converge 差异报告 + 验收追溯矩阵 | UAT 签字 | G5：AC 全通过 + 回退演练完成 |

---

## 四、从 Axure 原型提取需求的方法（含采集工程与非 UI 补源）

### 4.1 Phase 0 输入源核验清单（G0 门禁）

- [ ] `.rp` 源文件**或**完整 HTML 导出包（含 `data/` 目录，真实数据在 `data/document.js` 等文件中，而非 JS 动态渲染的页面空壳）可获取
- [ ] 分享链接访问控制确认：无密码 / 有密码 / 需登录态——已建立采集专用的访问凭证与授权
- [ ] Notes/批注可导出为 CSV（可提取性取决于导出方式，逐项验证）
- [ ] 原型快照归档位置与命名规则确定：`prototype-snapshots/<迭代>/`，记录 axshare 链接 + 导出包 SHA-256 哈希 + 发布日期
- [ ] **Axure 采集适配器**负责人与工作量已估算（需自研：解析导出包→结构化 JSON）
- [ ] 更新感知机制：Axure Cloud 无 webhook → 采用每日轮询导出包哈希 diff 或人工发布快照触发
- [ ] 降级方案确认：采集失败时改用"页面截图 + 多模态解析 + 人工走查表"，并显式记录信息缺口清单

### 4.2 原型解析四步工作流

```
Step 1 结构提取：Sitemap / 批注 / 交互事件(OnClick/OnChange) / 组件矩阵（表单、表格、弹窗、导航）
Step 2 流程梳理：角色与权限边界 / Happy Path + Edge Cases / 状态转换节点 / 页面跳转有向图
Step 3 模块拆解：按业务域分组 / 按技术层分组(前端组件、BFF、领域服务、数据层) / 共享能力识别 / 依赖图
Step 4 规格生成：模块→FR-xxx / 流程→US-xxx / 交互→AC-xxx(从属于 FR/US) / 全局约束→NFR-xxx
```

### 4.3 原型映射表（核心交付物）

| 原型页面/组件 | FR | US | AC | 契约 operationId | 数据实体 | 优先级 | 豁免标记 |
|--------------|-----|-----|-----|-----------------|---------|--------|---------|
| P01-登录页 | FR-auth-001 | US-001 | FR-auth-001-01~03 | FR-auth-001_login | User, Session | P0 | — |
| P09-装饰性 banner | — | — | — | — | — | P3 | N/A+理由：纯视觉无交互 |

**映射覆盖阈值（全文统一）**：关键流程与可交互元素 **100%** 必须映射；装饰性元素允许豁免但须标 `N/A + 理由` 且**经 PO 批准**；"豁免项未经 PO 批准"即为风险触发条件（见 §十二.1 R01）。

### 4.4 非 UI 需求补源表（原型不承载的需求，防止"虚假绿灯"）

| 需求类别 | 来源 | 责任人 | 门禁 |
|---------|------|--------|------|
| 后端业务规则/计算 | 领域专家访谈 + 决策表 | BA | 每条规则有 FR+AC |
| 系统集成/接口依赖 | 上下游系统接口文档 | 架构师 | 契约静态校验通过 |
| 数据模型与迁移 | DBA 数据字典 + 存量 schema 逆向 | DBA | expand-contract 方案评审 |
| 合规与隐私 | 法务/安全清单 | PMO | 高危项零遗漏 |
| 批量处理与迁移 | 业务方运营数据 | BA | 容量压测纳入 §九.3 流水线 |
| 第三方依赖 | 供应商 SLA | 架构师 | 依赖风险登记 |
| 容量/SLA/可靠性 | SLO 评审会 | SRE | NFR 量化 + 验证 stage 存在 |

### 4.5 原型变更入口规则（SSOT 治理）

**Axure = 输入证据（不可直接驱动开发）；`specs/` = SSOT。** 原型变更只能通过 Decision Gate 进入规格：

```
原型快照更新 → AI 生成《原型-spec 差异报告》→ Decision Gate（A=PO，SLA 2 个工作日）
   ├── 接受 → 更新 spec.md + 契约版本 +1 → 重新 analyze
   ├── 延期 → 记录原因 → 下轮 Backlog
   └── 拒绝 → spec 不变，差异报告归档
```

### 4.6 AI Agent 在原型解析阶段的职能

| 职能 | 具体动作 | 人审点 |
|------|---------|--------|
| 原型转文本 | 经采集适配器读取导出包，提取页面/组件/交互 | 结构完整性抽查 |
| 流程补全 | 识别断裂用户路径，提示缺失流程 | 业务正确性审核 |
| PRD/spec 草稿 | 生成 `spec.md` 草稿与 PRD | PO 评审 |
| 一致性校验 | 命名/交互模式/状态标注扫描 | 冲突裁决 |

---

## 五、BMAD-METHOD / Spec Kit / TDD 的分工与协同

### 5.1 BMAD-METHOD Agent 与执行运行时的分工矩阵

| 流程阶段 | BMAD persona（流程层） | Qoder/Codex（执行层） | 人工 Accountable |
|---------|----------------------|----------------------|------------------|
| Brief/PRD | `bmad-agent-analyst`、`bmad-prd` | 文档生成与润色 | PO |
| 架构 | `bmad-create-architecture` | C4 图/ADR 初稿 | Architect |
| Epic/Story | `bmad-create-epics-and-stories` | 故事拆解建议 | PO |
| 开发 | `bmad-dev-story`（Scrum Master 引导） | TDD 循环内代码/测试生成 | Dev Lead |
| 评审 | `bmad-code-review`、`bmad-qa` | 预审报告 | Dev Lead（合并权在人） |

### 5.2 Spec Kit：命令 ↔ 工件 ↔ 门禁 ↔ 角色映射表

仓库目录布局：`.specify/memory/constitution.md`；`specs/NNN-<feature>/{spec.md, plan.md, tasks.md, contracts/, research.md, data-model.md, quickstart.md}`；`prototype-snapshots/`；`docs/adr/`。

| Spec Kit 命令 | 工件 | 门禁 | 角色 (R/A) |
|--------------|------|------|-----------|
| `/speckit.constitution` | `.specify/memory/constitution.md` | G1 出口 | Architect (A) |
| `/speckit.specify` + `/speckit.clarify` | `specs/NNN/spec.md` | G2 出口 | BA (R) / PO (A) |
| `/speckit.plan` | `specs/NNN/plan.md` + research/data-model/contracts | G3 出口 | Architect (A) |
| `/speckit.tasks` | `specs/NNN/tasks.md`（TC/AC 标注） | G4 入口 | Dev Lead (R/A) |
| `/speckit.analyze` | 跨文档一致性报告（阻断 breaking change） | G4 入口 | QA (R) |
| `/speckit.implement` | 代码 + tasks 勾选状态 | G4 出口 | Dev (R) |
| `/speckit.converge` | 规格↔实现差异收敛报告（回灌 tasks.md） | G5 出口 | PM (A) |

### 5.3 契约验证的三类时机（修正 v0.1 三处矛盾）

| 验证类型 | 工具 | 发生时机 | 门禁归属 |
|---------|------|---------|---------|
| **契约静态校验** | Spectral（spec lint）、oasdiff / buf breaking（破坏性 diff） | Phase 3，CI 对 spec 文件运行，**不依赖实现** | G3 |
| **消费者驱动契约测试** | Pact（consumer 编写 → provider verify） | Phase 4，每个 PR 的 CI 运行 | G4 |
| **已部署服务契约一致性** | Schemathesis / Dredd 打 Staging | Pipeline `Deploy-Staging` **之后** | G5 前 |

### 5.4 TDD 分层与 Red-Green-Refactor

```
1. RED：从 AC 生成 TC（TC-<AC号>-<NN>），先单元后集成
   - RED 判定分语言表述：动态语言(JS/Python) → 运行期失败即 RED；
     静态语言(Java/C#) → 允许先建最小签名桩使其编译，断言失败即为 RED；
     唯一硬约束：禁止先写实现再补测试。
2. GREEN：只写让测试通过的最少代码
3. REFACTOR：消除重复、改善命名，保持全绿
4. 循环 → 下一个 AC
```

| 测试层级 | 工具 | 占比指导 | 有效性指标（防自证式测试） |
|---------|------|---------|--------------------------|
| 单元 | JUnit/Jest/pytest | ≥70%（金字塔非倒置） | 变异测试得分 ≥60%（PIT/Stryker，试点期 ≥40%）；每 TC 平均断言 ≥2 |
| 集成 | Testcontainers（DB/MQ） | 20-30% | 覆盖真实 IO 路径 |
| E2E | Playwright | ≤10% | 关键业务流全覆盖 |

注：MSW 定位为**前端组件层 Mock**，不算集成测试工具。Flaky 定义：同一 commit 无代码变更重跑结果不一致；处置：自动隔离至 `@quarantine` 标签、24h 内修复、纳入 §十二.2 DoD 门禁。

---

## 六、追溯 ID 与自动化规范（新增章节）

**ID 体系（全文唯一口径）**：

```
FR-<module>-<NNN>     功能需求        例：FR-auth-001
NFR-<category>-<NNN>  非功能需求      例：NFR-perf-001
US-<NNN>              用户故事        例：US-001
AC-<FR号>-<NN>        验收标准(从属FR) 例：FR-auth-001-01
TC-<AC号>-<NN>        测试用例(从属AC) 例：TC-FR-auth-001-01-01
ADR-<NNN>             架构决策记录     例：ADR-007
契约 operationId      = <FR号>_<action>  例：FR-auth-001_login
```

**机读锚点**：
- 测试方法注解：`@Spec("FR-auth-001-01")` / jest `it('TC-FR-auth-001-01-01: ...')`
- Commit trailer：`Spec-Ref: FR-auth-001`、`Story: US-001`、`Co-authored-by: <agent>/<model>`（AI 产出溯源）
- PR 模板必填：关联 FR/US 列表 + `TDD: strict|relaxed` 标注

**Pipeline stage 0 — Traceability（追溯完整性校验）**：
解析上述标记自动生成 `reports/matrix.md`（字段：原型页 | FR | US | AC | TC | operationId | 代码位置 | 测试报告条目 | 验收结论 | 状态）；**存在未覆盖的 AC、未关联 FR 的 TC、或无 Spec-Ref 的合入 commit → 流水线直接 fail**。禁止人工填表作为唯一手段。

---

## 七、AI Agent 全生命周期职能矩阵

| 阶段 | 子任务 | AI Agent 职能 | 人工介入点 |
|------|--------|-------------|-----------|
| Phase 0 | 采集核验 | 导出包哈希 diff、缺口清单汇编 | 降级方案签核 |
| Phase 1 | 启动会 | 议程生成、历史项目参考、风险预警 | 最终决策（A=见 RACI） |
| Phase 1 | RACI/OKR | 初稿推荐 + SLA 数值提醒 | 角色确认、优先级排序 |
| Phase 2 | 需求提取 | 采集结果→spec.md 草稿 | PO 业务正确性审核 |
| Phase 2 | 接口设计 | 数据流→OpenAPI 初稿 | RESTful 风格确认（A=Architect） |
| Phase 2 | 文档生成 | 规格格式化、版本管理（**仅提 PR，不合并**） | 术语统一审查 |
| Phase 3 | 架构方案 | C4 图、模式推荐、ADR 初稿 | 技术选型决策 |
| Phase 3 | 契约静态校验 | Spectral/oasdiff 报告解读 | 破坏性变更裁决 |
| Phase 4 | 代码脚手架 | Spec→骨架（`/speckit.implement` 驱动） | 架构约束确认 |
| Phase 4 | 测试生成 | AC→TC（标注 TC ID） | 边界条件补充 + 变异得分复核 |
| Phase 4 | 编码实现 | TDD 循环内代码生成 | Code Review（容量见 §十三.1 公式） |
| Phase 4 | 变更评审 | PR 影响分析、风险标注 | 合并审批（人） |
| Phase 5 | 验收映射 | 追溯矩阵生成与覆盖度验证 | 最终验收签字 |
| Phase 5 | 部署脚本 | Pipeline/Runbook 生成（**禁触凭据**） | 基础设施审批 |
| Phase 5 | 缺陷分析 | 根因分析（数据分级见 §八） | 修复方案确认 |
| Phase 5 | 文档交付 | converge 报告、手册汇编 | 发布审批 |

---

## 八、AI Agent 治理矩阵（权限、安全与兜底）

**分级授权模型**（取代 v0.1"100% 人审"的二元模式）：

| 风险级 | 任务类型 | 授权方式 | 降级/升级触发 |
|--------|---------|---------|--------------|
| 低风险 | 格式化、文档汇编、单测补充 | AI 自主 + 抽检 10% | 连续 20 次零缺陷保持；抽检发现问题→升级为必审 |
| 中风险 | 业务实现代码、契约 proposal | 人工必审后合并 | Review 驳回率 >15% → 升级为双人审 |
| 高风险 | 迁移脚本、权限配置、支付链路 | 双人审 + ADR | 不可降级 |

**治理矩阵（任务 × 权限）**：

| 任务 | 允许上下文（数据分级） | 允许写操作 | 必须人审项 | 审计要求 |
|------|----------------------|-----------|-----------|---------|
| 原型解析 | 原型导出包（公开级） | `specs/*/drafts/` | 需求缺口清单 | 会话日志入 `logs/ai/` |
| spec/契约产出 | 规格库 | **仅 proposal PR**，禁直改 baseline tag 后文件 | 全部 | prompt 版本 + 模型 ID 记录 |
| 代码生成 | 规格 + 代码库 | 功能分支，禁推受保护分支 | 全部（按风险分级） | commit trailer 标 `Co-authored-by` |
| 部署脚本生成 | IaC 模板 | 仅生成，禁执行 | 凭据引用改占位符审查 | 禁接触真实秘钥/生产凭据 |
| 缺陷分析 | **脱敏日志**（禁生产数据直连） | 无写权限 | 修复方案 | 数据出域审批记录 |

**安全硬规则**：
1. **提示注入防护**：原型批注 / Jira 描述 / 用户输入均属**不可信指令源**，注入 Agent 上下文前做指令/数据分离与转义；
2. **依赖幻觉防护**：AI 生成代码引入的**新依赖必须过 SCA（Trivy/Dependabot）+ lockfile 人工审查 + 私有源白名单**，防包抢注与不存在的恶意包；Pipeline stage 1 增加 SCA + 秘钥扫描（gitleaks）；
3. **AI 资产版本化**：`constitution.md`、project-context、rules/prompt 集、模型与版本清单入 Git，变更走 PR 评审（交付物见 §十）；
4. **供应商锁定兜底**：Qoder/Codex 任一不可用时降级为纯 Spec Kit CLI + 人工流程，每季度演练一次；
5. **许可证与归属**：AI 合入代码统一 CL 检查；成本配额：token/故事点设预算上限，超限告警至 PMO。

---

## 九、软件生命周期管理机制

### 9.1 全景流程

```
输入核验 → 需求管理 → 设计评审 → 开发实施 → 测试验证 → CI/CD → 发布部署 → UAT验收 → 运营反馈
                                                                              ▲（UAT 前置于生产发布）
    ▲──────────────────── 迭代闭环（converge + 反馈回流 Phase 0/1）◀─────────┘
```

### 9.2 各阶段管理机制与门禁

| 阶段 | 管理机制 | 门禁条件（可自动判定） | 详见§十二.2 度量 |
|------|---------|----------------------|-----------------|
| 输入核验 | Phase 0 清单 | G0：核验通过或降级签核 | 缺口登记数 |
| 需求管理 | Backlog + Decision Gate（SLA 2 工作日） | stage 0 追溯：唯一 ID + ≥1 AC + 负责人 | 变更率 |
| 设计评审 | 规格驱动 Checklist + 契约**静态**校验 | G3：Spectral 0E + oasdiff 无破坏性变更 + ADR 编号齐全 | 评审问题密度 |
| 开发实施 | TDD 循环 + 每日构建 + Pact | G4：stage 0-3 全绿 + 分层覆盖达标 + 变异得分达标 | 覆盖率/存活率 |
| 测试验证 | 分层测试 + Staging 契约一致性 + NFR 验证 | Blocker/Critical 清零，Major ≤3 且有排期承诺（见 §十一.4 分级定义） | 缺陷逃逸率 |
| CI/CD | GitOps 流水线（§9.3） | 全 stage 绿灯 | 构建通过率 |
| 发布部署 | 金丝雀 + **迁移向后兼容**（§十二.3） | 健康检查 + 回滚 Runbook + 本迭代回滚演练通过 | MTTR/成功率 |
| UAT 验收 | 业务方参与，**先于生产发布** | G5：converge 无未解释差异 + 追溯矩阵 100% | UAT 一次通过率 |
| 运营反馈 | 埋点 + NPS + SLA 监控 | 反馈项 100% 登记 Backlog 或豁免说明 | 满意度/可用性 |

### 9.3 CI/CD Pipeline（补齐追溯、SCA、性能、部署后契约验证）

```yaml
Stages:
  0. Traceability:   解析 Spec-Ref/TC 注解 → 生成 matrix.md → 孤条目即 fail
  1. Code Quality:   Lint → SAST → SCA(Trivy/Dependabot) → 秘钥扫描(gitleaks) → 镜像扫描
  1b. Security-DAST: 定时(每周) DAST 打 Staging
  2. Unit Test:      分层覆盖率(新增≥90%/核心分支≥80%/存量棘轮禁降) + 变异得分(Stryker/PIT)
  3. Integration:    Testcontainers 集成测试 → Pact provider verify（消费者契约）
  3b. Perf-Baseline: k6 基线压测（关键 3 接口：P95/吞吐阈值对比上一基线）
  4. Static-Contract: Spectral lint + oasdiff 破坏性变更检查（对 spec 文件，不依赖实现）
  5. Build:          编译/打包 → Docker → Push Registry
  6. Deploy-Staging: Apply → Smoke → Playwright E2E → Schemathesis 打已部署服务
  7. UAT:            业务方验收（先于生产）→ G5 判定
  8. Deploy-Prod:    金丝雀 5%→25%→100%，观察窗与自动回滚触发见 §十二.3
  9. Post-Deploy:    Monitor 观察窗 → converge 报告 → 关闭迭代
```

---

## 十、关键交付物清单及质量标准

| 阶段 | 交付物 | 质量标准 | 负责人 | AI 辅助 |
|------|--------|---------|--------|---------|
| Phase 0 | 输入源核验报告 | G0 清单全项有结论/降级方案 | BA | 哈希 diff 自动汇编 |
| Phase 1 | 项目愿景 + OKR | 成功标准可量化 | PM/PO | 模板 + 历史参考 |
| Phase 1 | `constitution.md` | 含原则/约束/治理规则，Spec Kit 可消费 | Architect | 生成初稿 |
| Phase 1 | RACI（含 SLA） | 每行**仅 1 个 A**、决策期限均有数值 | PM | 冲突检测 |
| Phase 1 | **AI 上下文基线**（project-context、rules/prompt 集、模型版本清单） | 入 Git、变更走 PR、与 spec 同版本 | 技术负责人 | — |
| Phase 2 | 原型映射表 | 关键元素 100%；豁免经 PO 批准 | BA | stage 0 自动校验 |
| Phase 2 | `spec.md`（SRS 由此汇编） | ID 规范 + 每 FR≥1 AC + clarify 无遗留 `[NEEDS CLARIFICATION]` | BA/PO | 从采集结果生成 |
| Phase 2 | 契约（OpenAPI/JSON Schema） | Spectral 0 Error、operationId 合规 | Architect | 从数据模型推导 |
| Phase 2 | 非 UI 需求补源清单 | §四.4 七类逐项有来源/责任人/门禁 | BA | 缺项扫描 |
| Phase 3 | `plan.md` + C4 + ADR | 设计条目 100% 可追溯 FR；ADR 四要素齐全 | Architect | 初稿生成 |
| Phase 3 | 契约静态校验报告 | oasdiff 无破坏性变更或已豁免签核 | QA | 自动生成 |
| Phase 4 | 源代码 | 分层覆盖达标、变异得分达标、commit trailer 齐全 | Dev | TDD 循环生成 |
| Phase 4 | 自动化测试套件 | TC ID 齐全、无 quarantine 超 24h 用例 | Dev/QA | AC→TC 生成 |
| Phase 4 | 性能/安全基线报告 | k6 对比基线无回归；SCA/DAST 高危=0 | QA/SRE | 报告汇编 |
| Phase 5 | converge 差异报告 | 无未解释差异 | PM | 自动对比 |
| Phase 5 | 部署 Runbook + 回滚方案 | 含触发阈值/责任人/RTO；本迭代演练通过 | SRE | 模板生成 |
| Phase 5 | 验收追溯矩阵 + UAT 报告 | 100% AC 验证通过、业务方签字 | PM/QA+**业务方(A)** | 矩阵生成 |
| Phase 5 | 用户手册/运维文档/FAQ | 场景驱动 | 技术写作 | 从规格汇编 |

---

## 十一、跨角色协作模式与决策流程

### 11.1 协作泳道与责任转移

```
业务方(UAT,A) ← PO(A·需求) ← BA(R·spec) ← Architect(R/A·契约) ← Dev(R·代码) ← QA(R·验证) ← SRE(A·发布)
     ▲              ▲____________│____________│____________│____________│____________│
     └──────────────┴──── 每个交接点标注出口门禁 G0-G5；变更统一走 Decision Gate ──────┘
AI Agent：横向支撑所有泳道（权限边界见 §八，仅提 PR / 不合并 / 不越权）
```

### 11.2 决策流程（RACI + Decision Gate，每行仅一个 A，SLA 数值化）

| 决策类型 | Responsible | Accountable | Consulted | Informed | 决策方式与 SLA |
|---------|------------|-------------|-----------|----------|---------------|
| 需求优先级 | BA/PM | **PO** | Dev Lead, QA Lead | 全团队 | BMAD 工作坊，每迭代一次 |
| **需求变更** | BA | **PO** | Architect, QA | 全团队 | Decision Gate，**2 个工作日内裁决** |
| 技术方案选型 | Dev Lead | **Architect** | PM, QA, SRE | 全团队 | ADR + Design Review，3 日 |
| 契约定义与兼容性 | Backend | **Architect** | Frontend Lead, QA | 全团队 | 契约评审会，3 日 |
| **前端契约适配** | Frontend | **Frontend Lead** | Backend | QA | 评审会，3 日（与上两行拆分，保证单 A） |
| **契约冻结豁免** | Dev Lead | **Architect** | PO | QA/PMO | 升级路径：Architect→PO，**1 个工作日** |
| 发布窗口 | SRE | **PM** | QA, PO, 业务方 | 全团队 | Gate，2 日 |
| **回滚决策** | SRE | **值班 SRE**（条件式授权，见 §十二.3） | PM, Dev Lead | 全团队 | **5 分钟内可先斩后奏**，事后 24h 复盘 |
| 缺陷修复优先级 | QA Lead | **PM** | Dev Lead | 开发团队 | 每日 Triage |

### 11.3 原型与规格同步规则

Axure 快照按迭代归档至 `prototype-snapshots/<迭代>/`，与该迭代 `specs/NNN/prototype-ref.md`（axshare 链接 + 导出包哈希 + 发布日期）绑定；原型更新**只能**通过 §四.5 的 Decision Gate 单向进入规格，禁止绕过 spec 直接驱动开发。

### 11.4 缺陷严重级别定义表（统一口径）

| 级别 | 判据 | 示例 | 门禁处置 |
|------|------|------|---------|
| Blocker | 阻断核心流程且无绕过方案 | 登录不可用 | G4/G5 阻断，24h 内响应 |
| Critical | 核心流程错误/资金数据错误/安全漏洞 | 支付金额算错 | 发布前必须清零 |
| Major | 非核心功能失效或有绕过方案 | 导出报表字段缺失 | G5 允许 ≤3 且有排期承诺 |
| Minor | 体验问题/文案 | 按钮错别字 | 记入 Backlog 不阻断 |

---

## 十二、风险控制、质量门禁、度量与回退

### 12.1 风险登记册（v0.2 扩充至 10 条，双周再评估）

| ID | 风险 | 概率 | 影响 | 应对策略 | 负责人 | 触发条件 |
|----|------|------|------|---------|--------|---------|
| R01 | 原型映射虚假绿灯 | 中 | 高 | stage 0 自动校验 + 豁免 PO 签核 | BA | 存在未签核的 N/A 豁免项 |
| R02 | 契约频繁变更导致返工 | 中 | 高 | baseline tag + 变更走 Gate（§11.2） | Architect | 单迭代破坏性变更 >2 次 |
| R03 | AI 代码质量不达标 / Review 瓶颈 | 中 | 高 | 三级授权 + 容量公式（§13.1） | Dev Lead | AI diff 存活率 <70% 或 Review 队列 >2 天 |
| R04 | 覆盖率失真（自证式测试） | 中 | 中 | 分层阈值 + 变异得分抽检 | QA | 变异得分低于阈值 |
| R05 | 原型不可采集/权限缺失 | 中 | 高 | Phase 0 核验 + 降级方案 | BA | G0 清单任一项无结论 |
| R06 | AI 服务不可用/供应商锁定 | 低 | 中 | §八.4 兜底流程 + 季度演练 | 技术负责人 | 服务中断 >4h |
| R07 | 提示注入/秘钥泄露/依赖投毒 | 低 | 高 | §八硬规则 + SCA/gitleaks 门禁 | SRE | 流水线检出高危项 |
| R08 | 回滚与 DB 迁移不一致 | 中 | 高 | expand-contract 规则 + 回滚演练 | SRE/DBA | 演练失败或迁移含破坏性 DDL |
| R09 | 关键人缺席 | 中 | 中 | 每角色 ≥1 备份、决策 SLA 顺延规则 | PMO | 单角色空缺 >3 天 |
| R10 | 工具授权/成本超支 | 中 | 中 | 预算告警 + token/故事点配额 | PMO | 月度成本超预算 20% |

### 12.2 质量门禁（DoR / DoD / 发布 Go/No-Go）

- **DoR**：AC 可测试、估算完成、依赖标注、原型链接（prototype-ref）、G0 核验无缺口
- **DoD**：测试全绿、分层覆盖达标（新增≥90%/核心≥80%/存量棘轮）、变异得分达标、Review 按风险分级通过、无超 24h 的 quarantine 用例、文档/矩阵更新
- **发布 Go/No-Go 清单**（责任人+证据格式统一为 `[检查项] 责任人 | 证据链接 | 不通过即 No-Go`）：
  - [ ] G5：converge 报告无未解释差异（PM | 报告链接）
  - [ ] UAT 通过且**先于生产**（业务方 | 签字单）
  - [ ] Blocker/Critical=0、Major≤3 有排期（QA | 缺陷板）
  - [ ] NFR 达标：P95/可用性/容量（SRE | k6 报告）
  - [ ] 无高危漏洞：SCA/DAST（SRE | 扫描报告）
  - [ ] 回滚 Runbook 本迭代演练通过（SRE | 演练记录）

### 12.3 回退策略（可执行化）

| 触发指标 | 阈值+持续窗口 | 动作 | 责任人 | RTO |
|---------|--------------|------|--------|-----|
| 错误率 | >1% 持续 5min | 金丝雀自动暂停 | 值班 SRE | — |
| P95 延迟 | 超基线 50% 持续 10min | 自动回滚上一版本 | 值班 SRE（5min 先斩后奏权） | <5min |
| 关键业务成功率（支付/登录） | <99% 持续 5min | 自动回滚 + 呼叫升级 | 值班 SRE→PM→PO | <5min |
| AI 生成结果异常 | 变异得分骤降/幻觉依赖检出 | PR 关闭 + 切换人工实现路径 | Dev Lead | 当日 |

**DB 迁移两条硬规则**：① 迁移必须 expand-contract（先加后删），禁止破坏性 DDL 与代码同版本发布，回滚窗口内保留旧列；② 每迭代一次回滚演练，作为 G5 门禁项。

### 12.4 度量体系（7 列可采集口径）

| 指标 | 定义与口径（分子/分母） | 数据源 | 频率 | 责任人 | 目标/基线 | 超限处置 |
|------|------------------------|--------|------|--------|----------|---------|
| 需求变更率 | 迭代中途新增或改写 FR 条数 / 迭代初 FR 总数 | Backlog API | 每迭代 | PO | <10% | 复盘 clarify 质量 |
| 评审问题密度 | Review 发现的有效问题数 / 评审工件页数（页=按 500 词折算） | Review 记录 | 每迭代 | Dev Lead | <2 个/页 | 检查 spec 草稿质量 |
| 分层覆盖率 | 行覆盖(新增≥90%)/分支覆盖(核心域≥80%)/存量棘轮不降 | CI | 每 PR | QA | 按左列 | 阻断合并 |
| 变异得分 | 被杀变异体 / 总变异体（Stryker/PIT） | CI nightly | 每周 | QA | ≥60%（试点≥40%） | 抽检低断言测试 |
| 缺陷逃逸率 | 生产 30 天窗口内 Blocker+Critical 数 / 全部关闭缺陷数 | 缺陷系统 | 每月 | QA Lead | <5% | 补测试缺口清单 |
| 变更前置时间 | commit 合并 → 生产运行中位数 | Git+CD | 每周 | SRE | <2 天 | 流水线瓶颈分析 |
| **AI diff 存活率** | 合入 14 天后仍存在的 AI 生成行数 / 合入时 AI 生成行数（git blame + PR label `ai-generated`） | Git | 每 PR 采集、每周汇总 | Dev Lead | 首季基线，次季 <70% 告警 | prompt/上下文复盘 |
| TDD 合规率 | PR 中 test commit 先于/同于 prod commit 的比例（或标 `TDD: strict`） | Git 顺序分析 | 每迭代 | Dev Lead | ≥90% | 案例复盘 |
| 高危漏洞/秘钥 | SCA+gitleaks 检出高危数；秘钥泄露事件 | CI | 每 PR | SRE | 0 | 阻断 + 应急流程 |
| UAT 一次通过率 | 无返工轮次即签字的 feature 数 / UAT feature 总数 | 验收记录 | 每迭代 | PM | ≥90% | 与 G5 联动分析 |
| token 成本/故事点 | Agent 用量 / 完成故事点 | 供应商账单 | 每月 | PMO | 预算内 | 配额调整 |

---

## 十三、可执行实施路线（爬坡制）

### 13.1 团队假设与评审容量公式

- **前提假设**：6-10 人（1 PO/BA、1 架构、3-4 dev、0.5-1 QA、0.5 DevOps/SRE），具备 Git/单测基础，**无** TDD/BDD/契约测试经验 → 必须 Iteration 0 训练。
- **评审容量公式**：`Review 人日 ≈ AI 生成合入行数 / (300~400 行/人日)`，Review 占团队产能 **≥20-30%**，写入排期；若队列 >2 天触发 R03。

### 13.2 三迭代爬坡路线

| 阶段 | 时长 | 内容（不做业务功能） | 出口 |
|------|------|---------------------|------|
| **Iteration 0** | 1-2 周 | ① `npx bmad-method install` + `specify init` 与 Qoder/Codex 集成可用性验证（不可用时启用替代路径）；② constitution.md 起草；③ Pipeline 骨架 + stage 0 追溯校验脚本；④ Staging 环境 + 监控接入；⑤ Axure 采集适配器开发；⑥ AI 权限/秘钥策略与分支保护；⑦ TDD/Gherkin/Pact 训练营 | G0-G2 演练通过 |
| **Iteration 1** | 2-3 周 | 选 **1 个中等复杂度模块**端到端跑通 Phase 1→5；采集全部指标基线；执行一次回滚演练 | 首个纵切交付 + 基线报告 |
| **Iteration 2 起** | 6 周稳态 | 按 §13.3 执行；**UAT 独立里程碑，前置于生产发布** | 稳态节奏 |

### 13.3 稳态 6 周节奏

| 周次 | 阶段 | 核心活动 | 里程碑 |
|------|------|---------|--------|
| W1 D1 | Phase 0 | 原型快照采集 + G0 核验 | 核验报告 |
| W1 D2-3 | Phase 1 | BMAD 工作坊、constitution 更新、RACI+SLA | G1 |
| W1 D4-5 | Phase 2 | `/speckit.specify`+`clarify`、原型映射表、非 UI 补源 | spec.md 草稿 |
| W2 D1-2 | Phase 2 | 契约定义 + analyze 一致性 → G2 | 契约 baseline tag |
| W2 D3-5 | Phase 3 | `/speckit.plan`、ADR、C4、契约**静态**校验(Spectral/oasdiff) → G3 | plan.md |
| W3 D1 | Phase 4 | `/speckit.tasks` → G4 入口 | tasks.md（TC/AC 标注） |
| W3-W5 | Phase 4 | TDD 循环 + Pact + k6 基线 + 追溯 stage 0 → G4 | 功能代码 + 矩阵无孤条目 |
| W6 D1-2 | Phase 5 | Deploy-Staging + Schemathesis + **回滚演练** | 演练记录 |
| W6 D3-4 | Phase 5 | **UAT 验收（先于生产）** + converge → G5 | 验收报告 |
| W6 D5 | Phase 5 | 生产金丝雀 + 观察窗 + 回顾会 | 发布 + Backlog 回流 |

### 13.4 分期日节奏（三张表）

| 时段 | 上午 | 下午 | 日终 |
|------|------|------|------|
| **规格期** (W1-W2) | 采集/工作坊/spec 编写 | 评审会 + clarify 问答（决策 SLA 倒计时看板） | AI 一致性扫描 + 缺口清单更新 |
| **开发期** (W3-W5) | 09:00 站会15min → TDD 深工作 3h | TDD 深工作 2h → 16:00 Review/AI 预审 1.5h → 17:00 Pipeline | 17:30 看板 + 风险预警 |
| **发布期** (W6) | 部署演练 / UAT 支持 | 缺陷 Triage + converge 差异处理 | Go/No-Go 检查 |

### 13.5 迭代检查清单（统一格式：责任人 | 证据 | 阻断动作）

**Planning 前**（PO | Backlog 链接 | 未满足不排期）：DoR 四项、stage 0 预扫描通过（QA | 扫描报告 | 阻断）
**Review 前**（PM | 矩阵链接 | 阻断）：G4/G5 清单、quarantine 用例=0（QA | 测试报告 | 阻断）

---

## 十四、存量/遗留项目接入指南（Brownfield，新增章节）

本方案主链路假设绿地项目；存量系统**无 Axure 原型、无法补齐全量 TDD**，按以下路径接入：

1. **逆向生成 spec**：从代码/接口/DB schema 反推 `spec.md` 与契约，每条标注置信度（高/中/低），低置信项必须人工确认；
2. **建立差异基线**：converge 首轮作为基线，未解释差异记入债务台账而非缺陷；
3. **增量强制、存量豁免**：仅对新增/变更代码强制 TDD 与覆盖率棘轮（ratchet：只许升不许降）；
4. **高风险模块优先**：按变更热点 × 事故来源排序补测试，不承诺全量重构；
5. **收益预期管理**：接入前给出 1-2 个迭代的安全网建设时间预算，明确"不重构、只加锁"。

---

## 十五、工具链（v0.2 勘误与补齐）

| 用途 | 工具 |
|------|------|
| 原型设计 | Axure RP；**Axure 采集适配器（自研脚本，解析导出包 data/ 目录）** |
| 流程 persona | **BMAD-METHOD（`npx bmad-method install`，bmad-code-org/BMAD-METHOD）** |
| 规格驱动 | **GitHub Spec Kit（`github/spec-kit` / specify-cli）** |
| 需求管理 | Jira / Linear + 自定义字段映射 |
| API 契约与静态校验 | OpenAPI 3.0 + **Spectral（lint）+ oasdiff / buf breaking（破坏性 diff）** |
| 契约测试 | **Pact（消费者驱动）+ Schemathesis/Dredd（已部署一致性）** |
| 数据建模 | dbdiagram.io / PowerDesigner |
| 架构文档 | Structurizr (C4) / Mermaid |
| TDD/BDD 框架 | JUnit/Jest/pytest；**SpecFlow（.NET BDD/Gherkin，测试框架而非规格工具）**；MSW（前端组件层 Mock） |
| 集成测试 | Testcontainers |
| 测试有效性 | **Stryker / PIT（变异测试）** |
| E2E / 性能 | Playwright；**k6 / Locust** |
| 安全 | **Trivy / Dependabot / Renovate（SCA+镜像）、gitleaks（秘钥）、DAST** |
| 追溯矩阵 | **自研脚本（stage 0）/ Xray** |
| CI/CD + 灰度 | GitHub Actions / GitLab CI；**Argo Rollouts / Flagger（金丝雀）** |
| AI 执行运行时 | **Qoder（编码协作）/ Codex（编码+文档+测试）**。执行前验证 `specify init --integration <agent>` 与 BMAD 在目标 agent 下的可用性，不可用时替代路径：纯 CLI + 人工粘贴上下文 |
| 监控 | Grafana + Prometheus / Datadog |

---

## 附录 A：术语表

| 术语 | 定义 |
|------|------|
| SSOT | Single Source of Truth，本方案中指 `specs/` 目录 |
| 规格 (Spec) | 需求与行为的正式描述（spec.md） |
| 契约 (Contract) | 接口/数据的可机读定义（OpenAPI/JSON Schema） |
| 门禁 (Gate) | 阶段出口判定条件，G0-G5 |
| 决策门 (Decision Gate) | 变更请求的 BMAD 评审入口，SLA 见 §11.2 |
| baseline tag | 契约冻结的 git tag，AI 与人仅能通过 PR 变更 |
| 棘轮 (ratchet) | 覆盖率只许升不许降的 CI 规则 |
| 金丝雀 | 5%→25%→100% 渐进放量发布 |
| 追溯矩阵 | stage 0 生成的 FR↔AC↔TC↔代码↔验收映射 |
| AI diff 存活率 | 合入 14 天后仍存在的 AI 生成行占比 |
| expand-contract | 先扩展后收缩的向后兼容 DB 迁移模式 |
| converge | Spec Kit 规格↔实现差异收敛命令 |

## 附录 B：v0.1 → v0.2 评审意见落实对照

23 项评审意见（C1-C6、W1-W11、S1-S6）全部采纳，落实位置见文末《改进对照说明》（评审回复文档）。

---

**核心逻辑：BMAD-METHOD 对齐流程与 persona · Spec Kit 锁定工件与契约 · TDD 保障质量 · Qoder/Codex 加速执行 · 治理矩阵守住安全边界**  
**不可跳过顺序：核验 → 对齐 → 规格 → 设计（可轻量） → 实现 → 验证**
