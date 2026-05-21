# claude-code skills

自定义 Claude Code Skill 集合。每个 skill 是独立的子目录，通过 symlink 挂载到全局 `~/.claude/skills/`，无需安装依赖，修改即生效。

## 包含的 Skills

| Skill | 描述 |
|-------|------|
| [sediment](sediment/SKILL.md) | 项目知识沉淀：把约定、踩坑、领域知识持续写入 `.ai-sediment/`，通过 `CLAUDE.md` 自动加载，让每次新会话都能拿到项目背景 |

## 安装

```bash
# 克隆仓库
git clone <repo-url> claude-code
cd claude-code

# 挂载所有 skill（逐个 symlink）
ln -sf "$(pwd)/sediment"     ~/.claude/skills/sediment
```

挂载后在任意项目打开 Claude Code，直接使用 `/sediment`。
