# Paid API Guard

调用付费外部 API 前的防护手册。把"昂贵调用"当成不可回滚的支出来对待，避免额度白费、跑到一半挂掉、结果静默为空。

## 安装

```bash
npx skills add https://github.com/wkstudy/skills --skill paid-api-guard --agent kiro-cli
```

## 覆盖什么

- **并发**：自己进程 + 调用方 SDK 的隐藏默认并发；并发过高 → 单次变慢 → 撞超时
- **耗时 / 超时**：单次 P95、调用方超时默认值能不能调、总耗时估算
- **配额**：TPM / RPM 换算，避免限流
- **计费**：免费额度 / 付费 / 欠费的错误码区分，硬预算熔断
- **失败语义**：可重试白名单、部分失败处理、完整性断言（不静默标成功）
- **可重放**：阶段产出落库，避免重跑整条链路

## 结构

```
paid-api-guard/
├── SKILL.md                    主定义 + 三步走 + 铁律
└── references/
    ├── checklist.md            动手前的六维检查清单（每项带"怎么查"）
    ├── patterns.md             防护代码模式（dry-run / 熔断 / 断言 / 白名单）
    ├── cases.md                案例库（随踩坑随加，"检测方法"必填）
    └── platforms.md            平台速查（各家默认值 / 限制 / 错误码）
```

## 怎么用

写/跑任何付费 API 调用前，先过 `checklist.md`；写代码时抄 `patterns.md`；踩了新坑按 `cases.md` 的格式追加一条。

案例库会持续长大 —— 这是它的价值所在。
