# wkstudy-skills

个人 skill 集合，供 Claude Code / 各类 AI agent 终端使用。遵循 [skills.sh](https://skills.sh/) 与 [anthropics/skills](https://github.com/anthropics/skills) 的约定：一目录一 skill，`SKILL.md` 为主定义。

## Skills

| Skill | 描述 | 安装 |
| --- | --- | --- |
| [**paid-api-guard**](skills/paid-api-guard) | 调用付费外部 API（LLM / embedding / rerank / 三方 SaaS）前的防护手册：并发、超时、TPM/RPM 配额、计费与欠费、失败语义、完整性断言。随踩坑随累积案例。 | `npx skills add https://github.com/wkstudy/skills --skill paid-api-guard` |

## 安装

任选一个 agent 终端安装（示例用 claude-code，换成你自己的终端即可）：

```bash
npx skills add https://github.com/wkstudy/skills --skill paid-api-guard --agent claude-code
```

- `--skill <名字>`：按 skill 名安装
- `--agent <名字>`：装到哪个 agent（`npx skills add --help` 看支持列表；`*` 表示全部，如 kiro-cli / cursor / codex 等）
- `-g`：装到全局；`-y`：跳过交互

## 新增一个 skill

```bash
cp -R template skills/<新skill名>
# 编辑 skills/<新skill名>/SKILL.md 的 frontmatter（name / description）
```

约定：

- `SKILL.md` 必需，frontmatter 只需 `name` + `description`；`description` 要写清**何时触发**（这是被正确调用的唯一依据，宁可"主动"一点）
- `references/` 放按需加载的补充文档；`scripts/` 放可执行脚本；`assets/` 放输出用模板
- 不提交平台安装产物（`.kiro/` `.claude/` `.skill-meta.json` 等，已在 `.gitignore`）

## License

MIT
