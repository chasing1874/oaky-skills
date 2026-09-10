# YouTube 能力 v1

当前已注册并完成逐能力真实 smoke 的 `active` 能力：

- `youtube.creator.profile`：用稳定的 `UC...` channel ID 读取频道资料。
- `youtube.creator.videos`：读取频道近期公开视频的一页有界样本。
- `youtube.video.detail`：用 11 位 video ID 或匹配的公开视频 URL 读取详情。

能力启用后，研究单个频道通常先取资料，再取一页近期视频。频道视频会优先读取 YouTube
公开 Atom feed；公开 feed 只覆盖近期内容，不等于频道完整历史。若结果带 `nextCursor`，原样回传
该 cursor 才能继续下一页；它同时绑定 capability 与频道，不能跨频道复用，也不要解析、修改或自行构造
cursor，更不要默认翻完所有分页。

频道输入必须使用稳定的 `UC...` channel ID。`@handle` 会变化，不能直接当作 `creatorId`；只有
handle 时应先从公开频道 URL 解析出 channel ID。单视频可接受 `watch`、`shorts`、`live` 或
`youtu.be` 链接，但 `contentId` 必须与 URL 中的 11 位 video ID 一致。

## 示例

```json
{
  "action": "invoke",
  "capabilityId": "youtube.creator.profile",
  "capabilityVersion": 1,
  "input": { "creatorId": "UCXuqSBlHAE6Xw-yeJA0Tunw" },
  "page": { "limit": 1 },
  "freshness": "prefer_cache",
  "resultMode": "inline"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "youtube.creator.videos",
  "capabilityVersion": 1,
  "input": { "creatorId": "UCXuqSBlHAE6Xw-yeJA0Tunw" },
  "page": { "limit": 30 },
  "freshness": "prefer_cache",
  "resultMode": "auto"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "youtube.video.detail",
  "capabilityVersion": 1,
  "input": {
    "contentId": "dQw4w9WgXcQ",
    "contentUrl": "https://www.youtube.com/watch?v=dQw4w9WgXcQ"
  },
  "page": { "limit": 1 },
  "freshness": "prefer_cache",
  "resultMode": "inline"
}
```

调用前仍应以 `describe_capability` 返回的当前状态、页大小和报价为准。频道近期视频中的相对发布时间、
订阅量或互动字段可能缺失；缺失值保持 `null`，不要从展示文本或同频道其他视频推断。跨频道比较时，
同时记录样本窗口、发布时间和订阅量级，避免把公开 feed 的近期样本写成完整频道表现。

## Provider 契约、价格与验收边界

2026-07-22 核对了 TikHub 官方 OpenAPI `V5.3.2`、YouTube API 产品页和价格目录。三项能力都先尝试
免费公开源；只有分类后的网络、HTTP、限流或响应结构失败，Social Data attempt runner 才允许执行
最多一次付费 fallback：

| capability | 免费源 | TikHub fallback | 关键参数 |
| --- | --- | --- | --- |
| `youtube.creator.profile` | `GET https://www.youtube.com/channel/<UC...>` | `GET /api/v1/youtube/web/get_channel_info` | `channel_id` |
| `youtube.creator.videos` | `GET https://www.youtube.com/feeds/videos.xml?channel_id=<UC...>` | `GET /api/v1/youtube/web_v2/get_channel_videos` | `channel_id`, `language_code`, `country_code`, `continuation_token?`, `need_format=true` |
| `youtube.video.detail` | watch page，失败后免费 oEmbed | `GET /api/v1/youtube/web_v2/get_video_info_v2` | `video_id`, `need_format=true` |

官方 YouTube API 页面当前标价为 `$0.001/request`；`get_channel_info` 另用官方 `calculate_price`
价格目录回查，`base_price` 同样为 `$0.001`。Clipbase 按 `$1 = ¥7.2` 的当前项目上限口径，把三项
capability 的 Provider 成本上限统一记为 `¥0.0072/次`，售价为资料/详情 `1 点/次`、视频列表
`1 点/页`。运行时仍以生效的不可变 Pricing Plan 和 `describe_capability` 报价为权威，本文不作为
永久价格源。

2026-07-22 已在专用付费闸门中强制三个免费源失败，并逐项调用上述三个 TikHub YouTube endpoint。
三项均以唯一一次付费 Attempt 成功，标准化输出、每项 ¥0.0072 成本、每项 1 点结算、返回脱敏和测试数据
清理均通过；频道资料和频道 feed 的免费链路此前也已独立通过。三项已按自身证据切为 `active`，不再被
同组知乎回答列表的外部失败连带阻断；运行时仍受总开关和 allowlist 控制。

核对来源：

- `https://api.tikhub.io/openapi.json`
- `https://tikhub.io/youtube-api`
- `https://docs.tikhub.io/413417983e0`（频道资料）
- `https://docs.tikhub.io/419083089e0`（频道视频）
- `https://docs.tikhub.io/461775522e0`（视频详情）
