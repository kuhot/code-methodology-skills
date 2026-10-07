# code-methodology-skills

> 在karpathy大神4条技能基础上完善的技能。
> 面向编程用户与 AI 编程代理的协作方法论技能集。
>
> 🌐 [English](./README.md) · [中文](./README-zh.md)

本仓库为 Claude Code、Cursor、Copilot 等 AI 编程代理提供一套可复用的行为准则（Skills），核心目标：

- **安全优先**：数据库与不可逆操作必须先备份、可回滚
- **正确优先**：先想清楚、先定义成功标准，再动手
- **最小改动**：外科手术式改动，diff 每一行都可追溯
- **简单优先**：能解决问题的最少代码，拒绝过度设计

优先级：**安全 > 正确 > 可验证 > 最小改动 > 速度**

---

## 目录

- [文件结构](#文件结构)
- [使用方法](#使用方法)
- [技能列表](#技能列表)
- [数据库安全约束](#数据库安全约束)
- [交付报告模板](#交付报告模板)
- [中文版速览](#中文版速览)
- [贡献](#贡献)
- [许可](#许可)

---

## 文件结构

```text
code-methodology-skills/
├── README.md          # 英文版说明
├── README-zh.md       # 中文版说明（本文件）
├── CLAUDE.md          # 技能主文件（中英双语，可被 Claude Code 自动加载）
└── LICENSE            # 许可协议（可选）
```

- `CLAUDE.md` 是核心文件，遵循 Claude Code 的约定，放在仓库根目录即可被自动加载。
- 若用于其他工具（Cursor / Copilot / Cline 等），可将 `CLAUDE.md` 内容复制到对应的规则文件中：
  - Cursor：`.cursorrules`
  - GitHub Copilot：`.github/copilot-instructions.md`
  - Cline：`.clinerules`
  - 通用：`AGENTS.md`

---

## 使用方法

### 方式一：作为仓库级规则（推荐）

在项目根目录放一份 `CLAUDE.md`：

```bash
# 在目标项目中
curl -o CLAUDE.md https://raw.githubusercontent.com/kuhot/code-methodology-skills/main/CLAUDE.md
```

或用 git submodule：

```bash
git submodule add https://github.com/kuhot/code-methodology-skills.git .skills/code-methodology
ln -s .skills/code-methodology/CLAUDE.md CLAUDE.md
```

### 方式二：作为用户级全局规则

放在用户目录，对所有项目生效：

```bash
# Claude Code 全局配置
cp CLAUDE.md ~/.claude/CLAUDE.md
```

### 方式三：作为提示词模板

在对话开头手动粘贴对应 Skill 段落，用于一次性任务。

---

## 技能列表

| # | Skill | 触发场景 | 核心要求 |
|---|-------|---------|---------|
| 0 | 总则 General Principles | 所有任务 | 不确定就问，不猜 |
| 1 | 先想清楚再写代码 Think Before Coding | 需求模糊、影响面大 | 摆出假设、选项、风险 |
| 2 | 简单优先 Simplicity First | 新功能、重构、抽象 | 最少代码，拒绝投机设计 |
| 3 | 外科手术式改动 Surgical Changes | 修改已有代码 | 只碰必须碰的，不顺手改 |
| 4 | 目标驱动执行 Goal-Driven Execution | 多步骤、修 bug、重构 | 先定义成功标准再动手 |
| 5 | 数据库使用约束 Database Safety | 任何数据库读写 | 先备份、可回滚、最小权限 |
| 6 | 验证与交付 Verification & Delivery | 任务完成时 | 报告改动、验证、风险 |

---

## 数据库安全约束

**默认原则：先读后写，先备份后删除，可回滚，最小权限。**

### 硬性规则

- ❌ 禁止无 `WHERE` 的 `UPDATE` / `DELETE`
- ❌ 禁止未经确认的 `DROP` / `TRUNCATE`
- ❌ 禁止硬编码数据库凭据
- ❌ 禁止提交 `.env`、连接串、密钥、令牌
- ✅ 删除或不可逆更新前**必须备份**
- ✅ 生产数据库操作**必须获得用户明确确认**
- ✅ 写操作前先用 `SELECT` 预览影响行
- ✅ 多步写操作放入事务，失败回滚
- ✅ DDL 使用迁移脚本，提供 up/down

### 删除 / 更新前备份示例

```sql
-- 1. 备份受影响数据（表名带时间戳）
CREATE TABLE orders_backup_YYYYMMDD_HHMMSS AS
SELECT * FROM orders WHERE created_at < '2024-01-01';

-- 2. 校验备份行数
SELECT COUNT(*) FROM orders_backup_YYYYMMDD_HHMMSS;

-- 3. 预览将删除的行
SELECT COUNT(*) FROM orders WHERE created_at < '2024-01-01';

-- 4. 执行删除（获得确认后）
DELETE FROM orders WHERE created_at < '2024-01-01';

-- 5. 回滚 SQL（如需恢复）
INSERT INTO orders
SELECT * FROM orders_backup_YYYYMMDD_HHMMSS;
```

### 数据库操作前检查清单

- [ ] 环境：prod / test？
- [ ] 权限：只读 / 写？
- [ ] 备份：已备份并验证？
- [ ] 预览：已 `SELECT` 影响行？
- [ ] 事务：是否需要？
- [ ] 回滚：SQL 已准备？
- [ ] 确认：生产删除 / 更新已获确认？
- [ ] 记录：影响行数 / 备份位置？

---

## 交付报告模板

任何代码或数据库变更完成后，按以下格式报告：

```text
假设：
改动：
验证命令：
验证结果：
风险：
待确认：
```

---

## 中文版速览

### 三条底线

1. **不确定就问**，不要猜，不要替用户做隐含假设。
2. **删除数据前必须备份**，并准备回滚方案。
3. **生产操作必须获得明确确认**，不可自作主张。

### 四步工作法

1. **想清楚**：明确假设、摆出选项、说明风险。
2. **写最小**：能解决问题的最少代码，不写投机性的东西。
3. **改最小**：diff 每一行都能追溯到用户需求。
4. **验证到通过**：先定义成功标准，循环到测试通过为止。

### 核心优先级

**安全 > 正确 > 可验证 > 最小改动 > 速度**

---

## 贡献

欢迎提交 Issue 或 PR 补充新技能。新增技能请遵循以下约定：

- 每条 Skill 包含：**触发场景 + 行为要求 + 输出格式（可选）**
- 中英双语并列
- 涉及数据、安全、生产的规则，一律按**最严格标准**处理
- 保持简洁，避免重复现有 Skill 的内容

---

## 许可

MIT License

---

## 参考

- [Claude Code 官方文档](https://docs.anthropic.com/en/docs/claude-code)
- [AGENTS.md 约定](https://agents.md/)
- https://github.com/multica-ai/andrej-karpathy-skills