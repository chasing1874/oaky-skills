# 抖音能力 v1

已注册的能力（当前是否可用和报价以 `describe_capability` 返回为准）：

- `douyin.content.search`：关键词搜索公开视频、图文或文章。
- `douyin.creator.profile`：用 `sec_user_id` 读取公开创作者资料。
- `douyin.creator.posts`：用 `sec_user_id` 分页读取公开作品。
- `douyin.content.detail`：用 `aweme_id` 读取单篇详情。
- `douyin.content.comments`：用 `aweme_id` 分页读取一级评论。

`creatorId` 只能使用创作者主页/标准输出中的 `sec_user_id`。抖音分享链接 query string 里的 `did`、`iid`、
`social_author_id`、`social_share_id`、`social_share_user_id` 是设备或分享链路标识，不是
`sec_user_id`，不得拿来调用 creator capability。只有作品分享链接时先解析得到 `aweme_id`，调用
`douyin.content.detail`，再从返回的 `item.creator.platformCreatorId` 取得稳定 creatorId。

## 示例

```json
{
  "action": "search_capabilities",
  "platform": "douyin",
  "query": "搜索公开内容",
  "limit": 8
}
```

```json
{
  "action": "invoke",
  "capabilityId": "douyin.content.search",
  "capabilityVersion": 1,
  "input": {
    "query": "AI 产品",
    "sort": "latest",
    "publishedWithin": "week",
    "contentType": "video"
  },
  "page": { "limit": 20 },
  "freshness": "prefer_cache",
  "resultMode": "auto"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "douyin.creator.profile",
  "capabilityVersion": 1,
  "input": { "creatorId": "MS4wLjABAAAA..." },
  "freshness": "prefer_cache",
  "resultMode": "inline"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "douyin.creator.posts",
  "capabilityVersion": 1,
  "input": { "creatorId": "MS4wLjABAAAA..." },
  "page": { "limit": 20 },
  "freshness": "prefer_cache",
  "resultMode": "auto"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "douyin.content.detail",
  "capabilityVersion": 1,
  "input": { "contentId": "7448118827402972455" },
  "freshness": "prefer_cache",
  "resultMode": "inline"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "douyin.content.comments",
  "capabilityVersion": 1,
  "input": { "contentId": "7448118827402972455" },
  "page": { "limit": 20 },
  "freshness": "prefer_cache",
  "resultMode": "auto"
}
```

只有短链接时先用 `common.entity.resolve`；若公开链接仍无法解析，明确说明缺口。搜索只代表当前关键词和当前页，
不能据此推断账号完整作品分布。创作者研究先取一页作品，再按问题选择少量详情或评论，不自动遍历历史作品。
