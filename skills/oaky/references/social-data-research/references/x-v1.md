# X / Twitter Social Data v1

仅在研究公开 X 内容时加载本参考。各 capability 当前是否可用、报价和限制以 `describe_capability`
返回为准。

## 能力

| capability | 输入 | 单次边界 |
| --- | --- | --- |
| `x.creator.profile@1` | `restId` 或 `handle`，严格二选一 | 1 个创作者 |
| `x.creator.posts@1` | `restId` 或 `handle`，严格二选一 | 固定 20 条的 1 页 |
| `x.post.detail@1` | 字符串 `postId`，可选匹配的 `postUrl` | 1 条帖子 |

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
- 不允许同时传 `restId` 和 `handle`，也不允许传数据源、endpoint、API key 或任意额外字段。

## 分页与停止

- `x.creator.posts` 每次只取一页。不要为了凑满用户指定数量而暗中多翻页。
- 后续页只使用上次 Call 返回的 `nextCursor`；它同时绑定 capability 与当前创作者，不能跨账号复用，也不要解析或自行构造。
- 通常先取 1 页评估证据；只有用户明确需要更长历史、下一页预期能改善覆盖，且不突破用户显式预算时再继续。

## 结果解释

- `creator.profile` 返回公开昵称、handle、头像、简介、认证状态及可用的粉丝、关注、发帖数。
- `creator.posts` 和 `post.detail` 统一返回 `SocialDataContent`。纯文字为 `text`，带图片为 `image`，视频或 GIF 为 `video`。
- `shareCount` 聚合 retweet 与 quote；`favoriteCount` 对应公开 bookmark 数（若数据源提供）。缺失指标保留 `null`，不要当成 0。
- 引用贴、reply/thread 关系、外链和完整媒体变体目前不是 v1 标准输出字段；需要这些证据时可用标准正文和 canonical URL 继续人工核对，不要声称已返回完整线程。
- 已删除、受保护、被限制或地区不可见的帖子可能为空或报错。不要无限重试。

## 详情的数据来源

- `x.post.detail` 由服务端选择数据来源，可能先读公开页面，必要时再使用付费来源；模型不选择、也不指定来源。
- 标准结果不包含数据源名称、endpoint、request ID、缓存地址、鉴权或签名参数；不要尝试从结果里推断。
