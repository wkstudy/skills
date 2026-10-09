# Backend Guard

个人后端踩坑手册。只收录亲历 / 亲学的坑,随踩随加——专治 vibecoding 的前端"本地跑得通、一上并发/规模就崩"。

## 安装

```bash
npx skills add https://github.com/wkstudy/skills --skill backend-guard --agent claude-code
```

(`--agent` 换成你用的终端,如 kiro-cli / cursor / codex;或 `'*'` 装到全部。)

## 定位

- **只收亲历 / 亲学的坑**,不堆通用教科书内容——通用后端规则去 skills.sh 装现成的。
- case 两种来源:**学习**(自己 vibecoding 不会意识到的盲区)和 **实战**(真踩过的)。
- 当前聚焦**数据库锁与并发**一个领域;新领域有 case 后再扩展触发范围。

## 结构

```
backend-guard/
├── SKILL.md                主定义 + 红线清单 + 怎么加 case
└── references/
    ├── cases.md            案例库(学习 + 实战,"来源"和"检测方法"必填)
    └── db-locks.md         数据库锁机制底层规则(第一个深度领域)
```

## 怎么用

写 / 改涉及数据库写操作或并发的代码时,先过 `SKILL.md` 的红线清单;想搞懂锁原理看 `db-locks.md`;学到或踩到新坑,按 `cases.md` 格式加一条,标明来源和检测方法。
