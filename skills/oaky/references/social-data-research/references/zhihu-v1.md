# 知乎能力 v1

当前基础包按 capability 独立发布：

- `zhihu.creator.profile`：`active`；用用户主页 `url_token` 读取公开创作者资料。
- `zhihu.creator.posts`：`active`；显式选择 `article` 或 `pin`，读取对应的一页内容。
- `zhihu.content.detail`：`active`；显式选择 `answer`、`article`、`question` 或 `pin`，读取单条详情。

## 使用边界

- 列表的 `contentType` 必填。一次调用只访问文章或想法的一种 endpoint 和一页数据，不能把两类内容暗中合并为两次付费请求。
- 回答列表不在 `zhihu.creator.posts@1` 的公开 schema 中。TikHub 当前唯一的按作者回答列表 endpoint 持续失败，
  且没有等价替代；服务端会在 Call、积分预占和 Provider Attempt 前拒绝 `contentType: "answer"`。
- 详情的 `contentType` 必填。服务端直达唯一匹配的 endpoint，不猜测类型，也不因一种类型返回空数据而尝试另一种类型。
- Provider 返回 HTTP 200 但资料为空、详情没有有效公开字段，或列表缺少已知 collection 时按失败处理。列表已识别到
  `data` / `items` 空数组且分页明确结束时，是合法空页；运行时返回 `items: []`，但发布 smoke 仍使用至少 1 条内容的
  独立稳定样本验证字段归一化。
- 游标只续取同一个创作者、同一种内容类型的下一页；不要自动遍历完整历史。
- 标准结果不会返回 TikHub endpoint、request ID、缓存地址、鉴权字段或原始响应。

## 示例

以下示例只适用于 capability 切为 active 且运行时 allowlist 已开放之后。

```json
{
  "action": "invoke",
  "capabilityId": "zhihu.creator.profile",
  "capabilityVersion": 1,
  "input": { "creatorId": "ming-he-43-93" },
  "freshness": "prefer_cache",
  "resultMode": "inline"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "zhihu.creator.posts",
  "capabilityVersion": 1,
  "input": {
    "creatorId": "ming-he-43-93",
    "contentType": "article",
    "sort": "created"
  },
  "page": { "limit": 20 },
  "freshness": "prefer_cache",
  "resultMode": "auto"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "zhihu.content.detail",
  "capabilityVersion": 1,
  "input": {
    "contentId": "2054145988235880002",
    "contentType": "answer",
    "contentUrl": "https://www.zhihu.com/question/37811449/answer/2054145988235880002"
  },
  "freshness": "prefer_cache",
  "resultMode": "inline"
}
```

## Provider 契约与价格证据

2026-07-22 先读取 TikHub 当前公开 OpenAPI 和 endpoint-info 元数据，随后按专用付费闸门调用已排到的
知乎数据 endpoint：

| 用途 | Method / path | 关键参数 | 当前 Provider 单价 |
| --- | --- | --- | --- |
| 创作者资料 | `GET /api/v1/zhihu/web/fetch_user_info` | `user_url_token` | `$0.001` |
| 回答列表（未纳入 v1 发布 schema） | `GET /api/v1/zhihu/web/fetch_user_answers` | `user_url_token`, `offset`, `limit`, `sort_type` | `$0.001` |
| 文章列表 | `GET /api/v1/zhihu/web/fetch_user_articles` | `user_url_token`, `offset`, `limit`, `sort_type` | `$0.001` |
| 想法列表 | `GET /api/v1/zhihu/web/fetch_user_pins` | `user_url_token`, `offset`, `limit` | `$0.001` |
| 回答详情 | `GET /api/v1/zhihu/web/fetch_answer_detail` | `answer_id` | `$0.001` |
| 文章详情 | `GET /api/v1/zhihu/web/fetch_column_article_detail` | `article_id` | `$0.001` |
| 问题详情 | `GET /api/v1/zhihu/web/fetch_question_detail` | `question_id` | `$0.001` |
| 想法详情 | `GET /api/v1/zhihu/web/fetch_pin_detail` | `pin_id` | `$0.001` |

官方 OpenAPI：`https://api.tikhub.io/openapi.json`。当前 Clipbase 售价统一为资料/详情 `2 点/次`、作品列表
`2 点/页`；Provider 价格按 `$1 = ¥7.2` 的当前项目上限口径折算为每次最多 `¥0.0072`。上线前仍需用
新的 endpoint-info 和价格版本复核，不能把本文件当成永久价格源。

## 2026-07-22 付费 smoke 结果

- `zhihu.creator.profile` 的唯一 TikHub Attempt 成功，资料标准化、¥0.0072 成本、2 点结算、脱敏与清理通过。
- 随后的旧候选 `zhihu.creator.posts` / `answer` 唯一 Attempt 被 `fetch_user_answers` 拒绝，Provider 明确说明本次请求
  不扣费；测试立即停止且没有自动重试。这个结果与仓库 2026-07-13 已记录的“多个有效账号持续 HTTP 400”
  外部降级一致，请求使用的 `user_url_token`、`offset`、`limit`、`sort_type` 与当前 OpenAPI 契约一致。
- 官方当前没有其他按作者完整列回答的 endpoint；知乎匿名 `/api/v4/members/.../answers` 在无登录态下返回
  401，因此没有用全站搜索后过滤作者的残缺结果冒充完整列表。`answer` 已在首次 active 前从作品列表 v1
  schema 移除。
- 新授权的 `zhihu_remaining` 随后按 fail-fast 执行：文章列表唯一 Attempt 成功，¥0.0072 成本、2 点结算与非空
  归一化通过；想法列表使用同一个文章作者时未解析出内容记录，唯一 Attempt 已发起并保守记录 ¥0.0072，Call 失败后
  释放积分，四类详情未继续执行。本轮实际 2 个 Provider 请求，成本 ¥0.0144（`$0.002` 上限口径），未超过授权。
- 仓库没有证据证明文章样本作者拥有公开想法，也没有保留本次 raw response，因此不把该结果追认为想法列表发布证据。
  运行时已区分“明确终止的合法空页”和“未知 collection”，付费 gate 新增独立
  `SOCIAL_DATA_SMOKE_ZHIHU_PIN_CREATOR_ID`，并提供 `zhihu_after_article` 范围：只跑想法列表和四类详情，共
  5 Call / `$0.005` / 10 点，不重复支付已通过的文章列表。
- 两次执行的临时 User、Call、Credit Transaction、Account、Entitlement 和 Job 均已清理，shared cache 已恢复。
- 随后的 `zhihu_after_article` 固定范围执行 5 个 Call：回答、文章、问题、想法四类详情，以及使用独立非空作者
  样本的想法列表全部成功。每项各有唯一一次 TikHub Attempt 并记录 ¥0.0072，合计 Provider 成本 ¥0.036、
  临时用户结算 10 点；标准化、分页、脱敏、审计和 finally 清理全部通过，没有重试或额外调用。

知乎资料、作品列表和详情现已分别按自身证据切为 `active`。作品列表的公开契约仍只有 `article | pin`；
作者回答列表没有随本轮验证恢复，`contentType: "answer"` 仍会在 Call 和 Provider Attempt 前被拒绝。
