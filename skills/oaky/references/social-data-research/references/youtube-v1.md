# YouTube 能力 v1

已注册的能力（当前是否可用、报价和页大小以 `describe_capability` 返回为准）：

- `youtube.creator.profile`：用稳定的 `UC...` channel ID 读取频道资料。
- `youtube.creator.videos`：读取频道近期公开视频的一页有界样本。
- `youtube.video.detail`：用 11 位 video ID 或匹配的公开视频 URL 读取详情。

研究单个频道通常先取资料，再取一页近期视频。频道视频会优先读取 YouTube
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

频道近期视频中的相对发布时间、订阅量或互动字段可能缺失；缺失值保持 `null`，不要从展示文本或同频道
其他视频推断。跨频道比较时，同时记录样本窗口、发布时间和订阅量级，避免把公开 feed 的近期样本写成完整
频道表现。

服务端可能先读取公开页面，必要时再使用其他数据来源；模型不选择、也不指定来源。
