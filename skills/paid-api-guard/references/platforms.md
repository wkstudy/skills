# 平台速查

各家 API / SDK 的默认值、限制、错误码。接新平台前先补一节。

## 百炼 dashscope（OpenAI 兼容端点）

- 端点：`https://dashscope.aliyuncs.com/compatible-mode/v1`
- **计费错误码**（必须区分，都要中止而非重试）：
  - 免费额度耗尽：`403` — `Free quota exhausted`（type `AllocationQuota.FreeTierOnly`）
  - **欠费**：`400` — `Access denied, please make sure your account is in good standing`（type/code `Arrearage`）
  - 限流：`429` — rate limit
- 识别建议：白名单只放超时/连接类；上述计费类一律中止。
- 换模型：`PWA_CHAT_MODEL` 改模型名，但 **base_url 可能没有独立环境变量**，换接入点要改代码默认值。

## arize-phoenix evals（`phoenix.evals`）

- `async_evaluate_dataframe(df, evaluators, concurrency=None, max_retries=None)`
  - **executor 超时硬编码 60s/次**，函数签名**不透出 timeout** → 大 prompt 必然超时，只能靠降并发 + 缩输入绕开。
  - 默认重试较多（~10 次），超时场景会放大浪费 → 用 `max_retries` 压低。
  - `concurrency` 默认偏高，单次慢时建议显式设小（实测 3 稳）。
- 预置 evaluator 输入契约：`Faithfulness` 需要 `input/output/context`；`Correctness`/`Refusal` 只要 `input/output`（**不注入参考答案，测不出要点覆盖**）。
- 预置是**二值**标签（faithful/unfaithful 等）；需要 0/1/2 分级要自写 `ClassificationEvaluator(choices={"0":0,"1":1,"2":2}, input_schema=...)`。
- experiment 落库：`create` + `log_run` + `log_evaluation`（客户端返回 dict，用下标取 `id`，不是属性）。

## 通用（任何新平台接入前必查）

- [ ] SDK 默认并发是多少？
- [ ] 单次超时默认值、能否配置？
- [ ] 默认重试次数？
- [ ] TPM / RPM 限制？
- [ ] 失败调用是否计费/扣配额？
- [ ] 计费/欠费/限流分别是什么错误码？
