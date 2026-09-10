# X / Twitter Social Data v1

仅在研究公开 X 内容时加载本参考。2026-07-22 三项真实 TikHub 路径已在专用付费闸门中逐项通过，
并按 capability 级证据切为 `active`；运行时仍受总开关和 allowlist 控制。

## 能力

| capability | 输入 | 单次边界 | 数据源 |
| --- | --- | --- | --- |
| `x.creator.profile@1` | `restId` 或 `handle`，严格二选一 | 1 个创作者 | TikHub Twitter Web |
| `x.creator.posts@1` | `restId` 或 `handle`，严格二选一 | 固定 20 条的 1 个 provider 页 | TikHub Twitter Web |
| `x.post.detail@1` | 字符串 `postId`，可选匹配的 `postUrl` | 1 条帖子 | 免费公开 HTML 优先；失败后最多 1 次 TikHub fallback |

示例：

```json
{
  "action": "invoke",
  "capabilityId": "x.creator.profile",
  "capabilityVersion": 1,
  "input": { "restId": "1639460977510092803" },
  "freshness": "prefer_cache",
  "resultMode": "inline"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "x.creator.posts",
  "capabilityVersion": 1,
  "input": { "handle": "seekjourney" },
  "page": { "limit": 20 },
  "freshness": "prefer_cache",
  "resultMode": "auto"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "x.post.detail",
  "capabilityVersion": 1,
  "input": {
    "postId": "2075471747114217888",
    "postUrl": "https://x.com/seekjourney/status/2075471747114217888"
  },
  "freshness": "prefer_cache",
  "resultMode": "inline"
}
```

## 身份和 ID 规则

- `restId` 是稳定的 numeric X user ID，但必须始终作为十进制字符串传递，禁止转成 JavaScript number。
- `handle` 是可变的发现身份，允许输入时带一个 `@`；服务端会规范成不带 `@` 的值。
- 已知 `restId` 时优先使用它。只有主页 URL 或用户口述 handle 时才用 `handle`。
- 资料响应如果同时含 `rest_id` 和 `screen_name`，标准结果的 `platformCreatorId` 使用 `rest_id`，`handle` 单独保留。
- `postId` 同样始终保持字符串；不要把 64 位 ID 当数字读写。
- 不允许同时传 `restId` 和 `handle`，也不允许传 provider、endpoint、API key 或任意额外字段。

## 分页与停止

- `x.creator.posts` 每次只取一个 provider 页。不要为了凑满用户指定数量而暗中多翻页。
- 后续页只使用上次 Call 返回的 `nextCursor`；它同时绑定 capability 与当前创作者，不能跨账号复用，也不要解析或自行构造。
- 通常先取 1 页评估证据；只有用户明确需要更长历史、下一页预期能改善覆盖，且不突破用户显式预算时再继续。
- 2026-07-22 核对的 TikHub 官方 X 产品页标价为 `$0.001/request`。按本项目当前汇率上限，Provider 成本上限为 `¥0.0072/次`；生效 Pricing Plan 的售价统一为资料/详情 `2 点/次`、帖子列表 `2 点/页`。

## 结果解释

- `creator.profile` 返回公开昵称、handle、头像、简介、认证状态及可用的粉丝、关注、发帖数。
- `creator.posts` 和 `post.detail` 统一返回 `SocialDataContent`。纯文字为 `text`，带图片为 `image`，视频或 GIF 为 `video`。
- `shareCount` 聚合 retweet 与 quote；`favoriteCount` 对应公开 bookmark 数（若 provider 提供）。缺失指标保留 `null`，不要当成 0。
- 引用贴、reply/thread 关系、外链和完整媒体变体目前不是 v1 标准输出字段；需要这些证据时可用标准正文和 canonical URL 继续人工核对，不要声称已返回完整线程。
- 已删除、受保护、被限制或地区不可见的帖子可能为空或报错。不要无限重试。

## 免费详情与付费 fallback

- `x.post.detail` 先调用 parser 的 `allowPaidFallback: false` 路径。该路径即使 Worker 配有 TikHub Key，也不得暗调 TikHub。
- 只有免费 HTML 网络失败、限流、HTTP 失败或 shape 不完整时，Social Data attempt runner 才能登记并执行一次 `fetch_tweet_detail` fallback。
- attempt runner 自身、持久化或计费错误不得被误判成公开源失败，也不得触发付费 fallback。
- 标准结果不暴露 TikHub 名称、endpoint、request ID、`cache_url`、Authorization、签名参数或 provider cursor。

## 2026-07-22 接口快照

核对来源：

- OpenAPI `V5.3.2`：`https://api.tikhub.io/openapi.json`
- 官方 X API 价格与端点：`https://tikhub.io/twitter-api`
- 官方资料接口文档：`https://docs.tikhub.io/191321710e0`
- 官方详情接口文档：`https://docs.tikhub.io/191321709e0`

官方 X API 页面在 2026-07-22 显示起价 `$0.001/request`。同日付费 smoke 已分别调用以下三个 endpoint，
每项只有一个 TikHub Attempt 且全部成功；资料、帖子页和单帖详情均通过标准化、每项 ¥0.0072 成本、
每项 2 点结算、输出脱敏与清理回查。三项能力当前均为 `active`，运行时以 `describe_capability` 返回的
effective status 为准。

当前请求合同：

- `GET /api/v1/twitter/web/fetch_user_profile`
  - `screen_name?: string`
  - `rest_id?: integer`（Clipbase 用字符串序列化，避免精度损失）
  - 二者只选一个；`rest_id` 优先。
- `GET /api/v1/twitter/web/fetch_user_post_tweet`
  - `screen_name?: string`
  - `rest_id?: integer`
  - `cursor?: string`
- `GET /api/v1/twitter/web/fetch_tweet_detail`
  - `tweet_id: string`

OpenAPI 的 200 响应仍只引用通用 `ResponseModel`，因此 v1 用已验证的 TikHub normalized shape 建 fixture：资料含 `rest_id/screen_name`，timeline 位于 `data.timeline` 且下一页位于 `data.next_cursor`，帖子 ID 为 `tweet_id/id_str/id` 字符串。Provider shape 变化必须先更新 fixture、严格 normalizer 和价格证据，再开放能力。
