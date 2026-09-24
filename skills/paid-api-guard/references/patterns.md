# 防护代码模式

可直接抄改。语言用 Python 示意，思路通用。

## 1. Dry-run 开关（先零成本验控制流）

把所有外部调用收敛到一个函数，用开关切 mock。这样循环/重试/落库/断言全部能在不花钱的情况下验证。

```python
async def call_llm(prompt, *, dry_run=False):
    if dry_run:
        return {"score": 1, "note": "[dry-run] fake"}   # 固定假响应
    return await real_client.ainvoke(prompt)
```

跑全量前先 `--dry-run` 跑一遍，确认：进度更新对、失败分支对、落库字段全、完整性断言能触发。

## 2. 成本熔断（不靠"额度用完"兜底）

```python
class Budget:
    def __init__(self, max_calls):
        self.max_calls, self.used = max_calls, 0
    def charge(self):
        self.used += 1
        if self.used > self.max_calls:
            raise RuntimeError(f"超出调用预算 {self.max_calls}，主动中止")

budget = Budget(max_calls=200)
for item in items:
    budget.charge()
    ...
```

## 3. 错误分类：白名单，不是黑名单

```python
RETRIABLE = ("timeout", "connection", "temporarily")   # 只有这些才重试
def is_retriable(err: str) -> bool:
    low = err.lower()
    return any(k in low for k in RETRIABLE)

# 其余一律中止——新出现的错误类型默认安全（不会被误当成可重试而空跑）
```

反面教材：用黑名单 `if "quota" in err: stop`，漏掉 `Arrearage`（欠费）就会一路空跑。

## 4. 完整性断言（失败必须"响"）

```python
def assert_complete(expected_ids, scores):
    got = [i for i in expected_ids
           if any(v is not None for v in scores.get(i, {}).values())]
    missing = [i for i in expected_ids if i not in got]
    if missing:
        raise RuntimeError(f"不完整：应 {len(expected_ids)} 实得 {len(got)}，"
                           f"缺 {missing[:8]}。多为额度/超时，检查后重跑")
    return len(got)
```

铁律：**宁可误报失败，不可漏报成功**。"运行成功但结果是空的"比失败危险得多。

## 5. 重试有上限且可见

```python
for attempt in range(1, MAX_RETRIES + 1):
    try:
        return await call()
    except Exception as e:
        if not is_retriable(str(e)):
            raise                      # 不可重试的立即抛
        log(f"重试 {attempt}/{MAX_RETRIES}: {e}")
# 耗尽记 None + 错误，不要无限重试
```

## 6. 阶段产出落库，可独立重放

把"贵阶段"（如 Agent 执行）的产出先落库，"轻阶段"（如判分）独立读库重跑。

好处：判分/口径出问题时，只重跑判分（几分钟），不用重跑 Agent（几十分钟）。

```python
await persist(agent_outputs)          # 贵阶段，落库
scores = await judge(load(run_id))    # 轻阶段，可反复重跑（--rejudge）
```

## 7. 先落已得，再校验完整性

部分成功的结果要保住，别因为一次超时把整轮白跑：

```python
await write_scores(scores)        # 先落已拿到的
assert_complete(expected, scores) # 再校验，不全就标 failed（但数据已保住）
```

## 8. 配额未知时：低起步 + 429 退避探测

不知道对方 TPM/RPM 时，别按"无限配额"盲跑。先查文档/响应头/控制台，**查不到就问用户**（他可能知道账号档位）；仍不确定就用自适应策略——低并发起步，把 429 当探测信号，撞到就退避降速，让限流器自己找到天花板。

```python
concurrency = 2                       # 未知配额 → 低起步，不要拉满
for attempt in range(1, MAX_RETRIES + 1):
    try:
        return await call()
    except Exception as e:
        if is_rate_limit(str(e)):     # 429 必须在可重试白名单里
            await asyncio.sleep(2 ** attempt)   # 指数退避
            continue
        if not is_retriable(str(e)):
            raise
# 顺带：从响应用量字段反推单次 token / 实际速率，逐步校准并发
```

关键：**429 不进白名单 → 一撞限流就崩**；进了白名单 + 退避 → 配额未知最坏也只是变慢，不会失败。
