# Dya Skills

Claude Code 技能库 —— 面向软件工程的 AI 辅助工作流。

## 这是什么？

本仓库包含为 [Claude Code](https://claude.ai/code) 设计的技能（Skill）。每个技能定义了一套结构化工作流，将复杂任务分解为多个认知独立的角色，通过分离的子代理执行，防止单一代理角色的认知盲点。

## 已有技能

### multi-role-workflow

将软件工程任务分解为 **4 个认知独立角色**，每个角色以独立子代理运行：

```
Task → Advisor → Executor → Grader → Executor(fix) → Dreamer → Done
```

| 角色 | 软件工程对应 | 职责 |
|------|-------------|------|
| **Advisor** | Tech Lead / 架构师 | 策略分析、架构设计、7 大设计原则审查、验收标准定义 |
| **Executor** | 研发工程师 | 按计划实现、编写测试、处理边界条件 |
| **Grader** | Code Reviewer / QA | 8 维对抗性审查：正确性、完整性、代码质量、7 大设计原则、鲁棒性、测试质量、性能/安全、回归风险 |
| **Dreamer** | 知识管理 | 提取可复用模式、prompt 模板、检查表、知识库条目 |

**适用场景：** 新功能开发、重构、Bug 修复、API/Schema 变更、安全敏感任务、架构决策、技术选型。

**内置 7 大设计原则：** 单一职责、开闭原则、里氏替换、依赖倒置、接口隔离、最少知识、合成复用。

## 安装

### 方式一：克隆到用户技能目录（推荐，所有项目可用）

```bash
git clone git@github.com:raydya/skills.git ~/.claude/skills/dya
```

### 方式二：克隆到项目技能目录（仅当前项目使用）

```bash
git clone git@github.com:raydya/skills.git /path/to/your-project/.claude/skills/dya
```

### 方式三：作为 Git 子模块

```bash
cd /path/to/your-project
git submodule add git@github.com:raydya/skills.git .claude/skills/dya
```

### 验证安装

在 Claude Code 会话中输入需要架构决策或复杂实现的任务，技能会根据任务特征自动匹配触发。也可以直接引用：

```
/multi-role-workflow 实现用户认证模块，支持 JWT 刷新和撤销
```

## 目录结构

```
skills/
└── multi-role-workflow/
    ├── SKILL.md                  # 技能定义：工作流、角色、红旗警告
    └── prompts/
        ├── advisor.md            # Advisor 子代理 prompt 模板
        ├── executor.md           # Executor 子代理 prompt 模板
        ├── grader.md             # Grader 子代理 prompt 模板
        └── dreamer.md            # Dreamer 子代理 prompt 模板
```

## 更新

技能会持续迭代优化（prompt 模板改进、审查维度调整、设计原则更新等）。获取最新版本：

### 克隆方式安装 → git pull

```bash
cd ~/.claude/skills/dya && git pull origin main
```

### 子模块方式安装 → 更新子模块

```bash
cd /path/to/your-project && git submodule update --remote .claude/skills/dya
```

### 查看变更

```bash
cd ~/.claude/skills/dya && git log --oneline -10
```

## 贡献

技能遵循 TDD 流程开发（RED-GREEN-REFACTOR），新增或修改技能需要先编写压力测试场景验证基线行为。

1. Fork 本仓库
2. 按 `superpowers:writing-skills` 规范创建或修改技能
3. 提交 PR

## License

[Apache 2.0](LICENSE)
