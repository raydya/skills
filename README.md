# Dya Skills

Claude Code 技能库 —— 面向软件工程的 AI 辅助工作流。

## 这是什么？

本仓库包含为 [Claude Code](https://claude.ai/code) 设计的技能（Skill）。每个技能定义了一套结构化工作流，将复杂任务分解为多个认知独立的角色，通过分离的子代理执行。

**核心理念：认知分离防止盲点。** 设计架构的人不能审查自己的设计。写代码的人不能给自己的代码打分。

## 已有技能

### multi-role-workflow — 多角色软件工程流水线

将软件工程任务分解为 **4 个认知独立角色**，按 **5 个阶段**流水线执行：

```
Task → Advisor → Executor → Grader → Executor(fix) → Dreamer → Done
         │           │           │            │            │
         ▼           ▼           ▼            ▼            ▼
    架构+验收标准  生产级代码   代码审查     修复+重测    知识沉淀
```

每个箭头 = 启动一个独立子代理。每个阶段产出具体的、可传递的制品。无阶段可跳过。

#### 角色与阶段

| 阶段 | 角色 | SE 对应 | 做什么 | 不能做什么 |
|------|------|---------|--------|-----------|
| 1 | **Advisor** | Tech Lead / 架构师 | 任务分类、架构模式选择、模块拆解、API 契约、数据模型、技术选型、风险评估、验收标准定义、边界条件枚举、**7 大设计原则逐条审查** | 写代码、做实现、评审具体代码 |
| 2 | **Executor** | 研发工程师 | 严格按计划实现、遵循 7 大设计原则、编写覆盖验收标准的测试、处理所有边界条件、匹配项目代码规范 | 自行设计架构、自我审查、偏离计划 |
| 3 | **Grader** | Code Reviewer / QA | 对抗性独立审查 **8 个维度**、输出带严重级别的问题清单、给 file:line 精确定位、出具正式 VERDICT 结论 | 修改代码、建议替代架构、温和通过 |
| 4 | **Executor(fix)** | 研发工程师 | 修复所有 Critical/Major 问题、逐条 before/after 展示、重跑所有测试、对误报给出证据反驳 | 引入新功能、重构无关代码 |
| 5 | **Dreamer** | 知识管理 / 文档工程师 | 提取设计模式目录、沉淀可复用 prompt 片段、产出检查表和知识库条目、做流程回顾、**反馈建议给 skill 本身** | 评判代码质量、重做已完成的工作 |

#### 顾问的完整分析覆盖

Advisor 产出的分析文档是**流水线的契约**——Executor 遵照执行，Grader 对照审查。包含：

- **任务分类**：类型、复杂度、影响域、爆炸半径
- **架构方案**：推荐模式（MVC/六边形/CQRS/Pipeline）、模块拆解、数据流、API 契约、数据模型、集成点
- **7 大设计原则逐条审查**（✅/⚠️/❌ + 证据/风险说明）
- **技术选型**：多选一对比表（选项/推荐/理由）
- **风险评估**：风险项、严重程度、概率、缓解措施
- **验收标准**：可证伪的具体标准（禁止模糊词如"代码质量高"）
- **非功能需求**：性能、安全、可维护性、兼容性
- **边界条件**：空值、错误路径、并发、大输入、超时、网络失败、竞态条件

#### 审查员的 8 维审查

Grader 以对抗性视角独立审查，力求找出问题而非验证通过：

| 维度 | 检查内容 |
|------|---------|
| 1. 正确性 | 逻辑错误、算法错误、API 误用、假设是否成立 |
| 2. 完整性 | 验收标准是否全部满足、边界条件是否全部处理、是否缺文件/处理器/迁移 |
| 3. 代码质量 | 命名、结构、注释、DRY 违反 |
| 4. **7 大设计原则** | 逐条检查违反情况并定位 file:line |
| 5. 鲁棒性 | 错误处理、空值安全、并发、资源管理、输入校验 |
| 6. 测试质量 | 断言强度、边界覆盖、错误路径测试、测试是否会失败 |
| 7. 性能/安全 | 算法复杂度、N+1 查询、授权检查、数据暴露、依赖风险 |
| 8. 回归风险 | 现有功能影响、调用方更新、共享类型变更、迁移路径 |

每个 finding 格式：`[严重级别] [分类] — 摘要 | Location: file:line | Detail | Fix direction`，最后出具正式 VERDICT（PASS / PASS WITH MINOR / NEEDS FIXES / FAIL）。

#### 知识沉淀的六大产出

Dreamer 在所有阶段完成后运行，萃取可复用资产：

1. **设计模式与架构目录** — 含适用场景和使用参考链接
2. **可复用 Prompt 片段** — 逐字保存经实践验证有效的指令
3. **检查表** — 实现前检查、代码审查检查、部署/迁移检查、测试检查
4. **知识库条目** — `[[条目名]]` 格式，含 Why 和 How to apply
5. **流程回顾** — 什么做得好、什么可以改进、是否缺少角色
6. **Skill 自改进建议** — 根据本次运行反馈建议修改 prompt 模板或 SKILL.md

#### 七大设计原则

贯穿 Advisor → Executor → Grader 的完整闭环：

| # | 原则 | 英文 | 各角色如何体现 |
|---|------|------|--------------|
| 1 | 单一职责 | Single Responsibility | Advisor 架构评审；Executor 每个类/函数只有一个变化原因；Grader 检查多职责类 |
| 2 | 开闭原则 | Open-Closed | Advisor 评估扩展性；Executor 新功能=新文件/类而非改旧代码；Grader 检查是否用 if-else 堆砌新功能 |
| 3 | 里氏替换 | Liskov Substitution | Advisor 评估继承体系；Executor 子类不弱化父类契约；Grader 检查子类是否破坏父类行为 |
| 4 | 依赖倒置 | Dependency Inversion | Advisor 评估依赖方向；Executor 依赖抽象不依赖具体，使用 DI；Grader 检查高层是否导入底层细节 |
| 5 | 接口隔离 | Interface Segregation | Advisor 评估接口粒度；Executor 用多个聚焦接口而非一个胖接口；Grader 检查调用方是否依赖未用方法 |
| 6 | 最少知识 | Law of Demeter | Advisor 评估模块耦合度；Executor 只调用直接协作者，禁止 `a.b.c.d()`；Grader 检查链式调用 |
| 7 | 合成复用 | Composite Reuse | Advisor 评估继承深度；Executor 优先组合/聚合而非继承；Grader 检查 >2 级继承层级 |

参考来源：[设计模式的七大原则](https://github.com/guanguans/notes/blob/master/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%E7%9A%84%E4%B8%83%E5%A4%A7%E5%8E%9F%E5%88%99.md)

#### 触发场景

自动匹配以下软件工程任务类型：

- 新功能开发 / 从零构建
- 有回归风险的重构
- 复杂子系统 Bug 修复
- API / Schema / 协议变更
- 涉及安全或数据完整性
- 需要架构决策
- 引入新库/框架
- 需要提取可复用模式

**跳过：** 拼写修正、格式化、单行日志、修改常量、零设计选择的任务。

## 安装

### 方式一：克隆到用户技能目录（推荐）

所有项目共享，Claude Code 全局可用：

```bash
git clone git@github.com:raydya/skills.git ~/.claude/skills/dya
```

### 方式二：克隆到项目技能目录

仅当前项目使用：

```bash
git clone git@github.com:raydya/skills.git /path/to/your-project/.claude/skills/dya
```

### 方式三：作为 Git 子模块

跟随项目版本管理：

```bash
cd /path/to/your-project
git submodule add git@github.com:raydya/skills.git .claude/skills/dya
git commit -m "Add dya skills as submodule"
```

### 验证安装

技能根据任务特征自动触发。提供复杂任务即可看到流水线启动。也可以显式引用：

```bash
# 在 Claude Code 会话中
/multi-role-workflow 实现用户认证模块，支持 JWT 签发、刷新和撤销
```

## 目录结构

```
skills/
├── README.md
├── LICENSE
└── multi-role-workflow/
    ├── SKILL.md                  # 技能定义：工作流、角色、触发条件、红旗警告
    └── prompts/
        ├── advisor.md            # Advisor 子代理 prompt（Tech Lead）
        ├── executor.md           # Executor 子代理 prompt（Developer + Fix Phase）
        ├── grader.md             # Grader 子代理 prompt（Code Review + 8维审查）
        └── dreamer.md            # Dreamer 子代理 prompt（Knowledge Manager）
```

## 贡献

技能遵循 TDD 流程开发（RED-GREEN-REFACTOR），新增或修改技能需要先编写压力测试场景验证基线行为。

**原则：没有失败的测试就不写技能。** 必须先看子代理在无技能时的基线行为（会犯什么错、会用什么借口），再写针对性的技能来解决这些具体失败模式。

1. Fork 本仓库
2. 按 `superpowers:writing-skills` 规范创建或修改技能
3. 提交 PR

## License

[Apache 2.0](LICENSE)
