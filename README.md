# Claude 开发规范体系

本目录定义了团队使用 Claude Code 进行协作开发时的**三层规范体系**，确保 AI 辅助开发在团队、项目、个人三个维度上保持一致性。

## 三层架构总览

```
Layer 3 · 个人层        ~/.claude/CLAUDE.md
                         ~/.claude/commands/          ← 个人专属命令（本机生效）
                            ↓ 覆盖 / 扩展
Layer 2 · 项目层        <project>/CLAUDE.md
                         <project>/docs/standards/    ← 项目特有规范
                         <project>/.claude/commands/  ← 项目共享命令（提交仓库）
                         <project>/.claude/settings.json ← 项目 Hooks 配置
                            ↓ 覆盖 / 扩展
Layer 1 · 团队层        team-standards/ (独立 Git 仓库)
                         CLAUDE.md
                         docs/standards/              ← 团队规范文档
                         .claude/commands/            ← 团队通用命令（规划中）
```

**优先级：个人层 > 项目层 > 团队层**（下层是基础，上层可覆盖或扩展）

---

## 目录结构（当前实际文件）

```
README.md                                    # 本文件，体系总览
specification-example/
│
├── team/                                        # Layer 1：团队级规范模板
│   ├── CLAUDE.md                                # 团队 Claude 入口 + 工作约束
│   └── docs/standards/
│       ├── architecture.md                      # 架构原则与 ADR 机制
│       ├── coding/
│       │   ├── base.md                          # 通用编码规范（语言无关）
│       │   ├── go.md                            # Go 专项规范
│       │   ├── python.md                        # Python 专项规范
│       │   └── typescript.md                    # TypeScript 专项规范
│       ├── docs-writing.md                      # 文档写作规范（README、ADR、CHANGELOG）
│       ├── security.md                          # 安全开发规范
│       └── testing.md                           # 测试策略与规范
│
├── project/                                     # Layer 2：项目级规范模板
│   ├── CLAUDE.md                                # 项目 Claude 入口模板（需按项目填充）
│   ├── .claude/
│   │   ├── README.md                            # Commands + Hooks 完整使用说明
│   │   ├── settings.json                        # Hooks 配置（lint、防护、通知）
│   │   └── commands/
│   │       ├── README.md                        # 命令列表索引
│   │       ├── review.md                        # /review：全面代码 Review
│   │       ├── test.md                          # /test：生成单元测试
│   │       └── migrate.md                       # /migrate：创建数据库迁移
│   └── docs/standards/
│       ├── build.md                             # 构建规范（环境、命令、CI/CD）
│       └── architecture.md                      # 项目架构说明（可选，覆盖团队层）
│
└── personal/
    └── CLAUDE.md                                # Layer 3：个人配置模板
```

---

## 团队共同维护的内容

三层体系不是"定义完就结束"的文档，而是需要团队持续共建的活文档。以下是各类内容的维护责任和协作方式。

### 规范文档（docs/standards/）

团队所有成员都应参与规范的演进，而非只有技术负责人才能修改。

| 文件                                         | 当前状态               | 维护方式                |
|--------------------------------------------|--------------------|---------------------|
| `team/docs/standards/architecture.md`      | 框架已定，`[团队填写]` 处待补全 | 架构师主导，PR Review 决策  |
| `team/docs/standards/coding/base.md`       | 完整，可直接使用           | 全员提 PR，技术负责人 Review |
| `team/docs/standards/coding/go.md`         | 框架完整，部分填写          | Go 开发者维护            |
| `team/docs/standards/coding/python.md`     | 框架完整，部分填写          | Python 开发者维护        |
| `team/docs/standards/coding/typescript.md` | 框架完整，部分填写          | 前端/TS 开发者维护         |
| `team/docs/standards/testing.md`           | 完整，可直接使用           | 全员提 PR              |
| `team/docs/standards/security.md`          | 完整，可直接使用           | 安全负责人 + 全员提 PR      |
| `team/docs/standards/docs-writing.md`      | 完整，可直接使用           | 全员提 PR              |
| `project/docs/standards/build.md`          | 模板，**每个项目必须填写**    | 项目负责人负责，开发者补充       |
| `project/docs/standards/architecture.md`   | 模板，按需使用            | 项目架构师填写             |

> **搜索 `[团队填写]` 即可找到所有需要集体决策的空白位置。**

### 自定义命令（.claude/commands/）

Commands 是效率工具，鼓励全员贡献。好的个人命令应提升为项目或团队命令。

| 层级  | 位置                                 | 维护者    | 当前内容                                  |
|-----|------------------------------------|--------|---------------------------------------|
| 团队层 | `team-standards/.claude/commands/` | 技术负责人  | **待建设**，脚手架统一注入                       |
| 项目层 | `<project>/.claude/commands/`      | 项目团队全员 | `/review` `/test` `/migrate`（示例，按需扩展） |
| 个人层 | `~/.claude/commands/`              | 个人自维护  | 不提交仓库，个人沉淀                            |

**协作流程：**
```
个人在 ~/.claude/commands/ 验证新命令
    ↓ 效果好，有通用价值
提 PR 到项目 .claude/commands/
    ↓ 多个项目都需要
提 PR 到 team-standards/.claude/commands/
    ↓ 脚手架更新后自动注入新项目
```

**扩展建议：** 以下场景适合沉淀为团队命令（欢迎贡献）：
- `/debug`：系统性排查 bug（读日志 → 假设 → 验证）
- `/release-note`：从 git log 生成发布说明
- `/security-check`：针对 PR 的安全专项检查
- `/onboard`：新成员项目快速上手引导
- `/adr`：创建架构决策记录（ADR）

### Hooks 配置（.claude/settings.json）

Hooks 是 Claude Code 的自动化防护层，应随项目成熟度逐步加强。

| 层级        | 位置                                | 维护者   | 提交仓库             |
|-----------|-----------------------------------|-------|------------------|
| 项目层（团队共享） | `<project>/.claude/settings.json` | 项目负责人 | **是**（质量门禁对全员生效） |
| 个人层（个人偏好） | `~/.claude/settings.json`         | 个人    | 否                |

**当前项目模板已包含的 Hooks（`project/.claude/settings.json`）：**

| Hook         | 事件                         | 作用                     |
|--------------|----------------------------|------------------------|
| 阻止 push main | `PreToolUse` (Bash)        | 防止 Claude 直接推送到保护分支    |
| 自动 lint      | `PostToolUse` (Edit/Write) | 文件修改后自动检查，结果反馈给 Claude |
| 完成通知         | `Stop`                     | 长任务完成后发送桌面通知           |

**建议逐步补充的 Hooks：**
- 阻止删除迁移文件（`PreToolUse`）
- 修改测试文件后自动运行对应测试（`PostToolUse`）
- 检测硬编码密钥（`PreToolUse`，可集成 gitleaks）
- 关键操作写入审计日志（`PostToolUse`）

---

## 快速上手

### 新成员初始化

1. Clone 团队规范仓库：
   ```bash
   git clone <team-standards-repo> ~/team-standards
   ```

2. 配置个人 Claude 设置：
   ```bash
   # 参考 personal/CLAUDE.md 模板
   cp specification/personal/CLAUDE.md ~/.claude/CLAUDE.md
   # 编辑，填写个人偏好
   ```

3. 了解项目规范：
   - 每个项目根目录的 `CLAUDE.md` 是该项目的 Claude 入口
   - `项目/.claude/commands/` 中有该项目的自定义命令，输入 `/` 查看

### 新项目初始化

使用脚手架自动注入项目级规范模板：

```bash
# 示例（脚手架工具，团队自行实现）
team-init-project --name <project-name> --lang go
```

脚手架会自动：
- 在项目根目录创建 `CLAUDE.md`（引用团队规范）
- 创建 `docs/standards/build.md` 模板（待填写）
- 创建 `.claude/` 目录结构（含基础 commands 和 settings.json）

### 贡献新命令

```bash
# 1. 在项目 .claude/commands/ 下新建命令文件
vim .claude/commands/<command-name>.md

# 2. 在 Claude Code 中立即可用（输入 / 查看命令列表）

# 3. 提交，团队共享
git add .claude/commands/<command-name>.md
git commit -m "feat: add /<command-name> command for <用途>"
```

---

## 各层职责边界

| 层级  | 位置                  | 管理者         | 规范文档        | Commands     | Hooks     |
|-----|---------------------|-------------|-------------|--------------|-----------|
| 团队层 | `team-standards` 仓库 | 技术负责人 / 架构师 | 跨项目通用原则     | 通用工作流命令（规划中） | 不适用       |
| 项目层 | 各项目仓库               | 项目负责人 + 全员  | 项目特有约定、构建规则 | 项目特有命令       | 质量门禁、安全防护 |
| 个人层 | `~/.claude/`        | 个人开发者       | 工作风格、个人偏好   | 个人探索命令       | 个人习惯自动化   |

---

## 文件命名约定

| 规范类型     | 文件名                               | 层级      |
|----------|-----------------------------------|---------|
| 架构规范     | `docs/standards/architecture.md`  | 团队 / 项目 |
| 编码基础     | `docs/standards/coding/base.md`   | 团队      |
| 语言规范     | `docs/standards/coding/<lang>.md` | 团队      |
| 测试规范     | `docs/standards/testing.md`       | 团队      |
| 安全规范     | `docs/standards/security.md`      | 团队      |
| 文档规范     | `docs/standards/docs-writing.md`  | 团队      |
| 构建规范     | `docs/standards/build.md`         | 项目（必填）  |
| 自定义命令    | `.claude/commands/<name>.md`      | 项目 / 个人 |
| Hooks 配置 | `.claude/settings.json`           | 项目 / 个人 |

---

## 演进流程

所有内容（规范、命令、Hooks）遵循同一条演进路径：

```
个人发现痛点或好的实践
    ↓
在个人层验证（~/.claude/commands/ 或本地实验）
    ↓
效果好 → 提 PR 到所在项目层
    ↓
经项目负责人 Review 合并
    ↓
发现跨项目通用价值 → 提至 team-standards 仓库
    ↓
技术负责人审批 → 合并 → 脚手架更新 → 新项目自动包含
```
