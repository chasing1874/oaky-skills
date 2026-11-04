# 小红书能力 v1

已注册的能力（当前是否可用和报价以 `describe_capability` 返回为准）：

- `xiaohongshu.note.search`：关键词搜索公开笔记。
- `xiaohongshu.creator.profile`：用 `user_id` 读取公开创作者资料。
- `xiaohongshu.creator.notes`：用 `user_id` 分页读取公开笔记。
- `xiaohongshu.note.detail`：用 `note_id` 读取单篇详情；已知图文或视频时显式传 `contentType`。
- `xiaohongshu.note.comments`：用 `note_id` 分页读取公开一级评论。

## 示例

```json
{
  "action": "invoke",
  "capabilityId": "xiaohongshu.note.search",
  "capabilityVersion": 1,
  "input": {
    "query": "AI 效率工具",
    "sort": "relevance",
    "noteType": "all",
    "publishedWithin": "week"
  },
  "page": { "limit": 20 },
  "freshness": "prefer_cache",
  "resultMode": "auto"
}
```

```json
{
  "action": "search_capabilities",
  "platform": "xiaohongshu",
  "query": "搜索公开笔记",
  "limit": 8
}
```

```json
{
  "action": "invoke",
  "capabilityId": "xiaohongshu.creator.profile",
  "capabilityVersion": 1,
  "input": { "creatorId": "62c98736000000001501e075" },
  "freshness": "prefer_cache",
  "resultMode": "inline"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "xiaohongshu.creator.notes",
  "capabilityVersion": 1,
  "input": { "creatorId": "62c98736000000001501e075" },
  "page": { "limit": 20 },
  "freshness": "prefer_cache",
  "resultMode": "auto"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "xiaohongshu.note.detail",
  "capabilityVersion": 1,
  "input": {
    "contentId": "697c0eee000000000a03c308",
    "contentType": "image"
  },
  "freshness": "prefer_cache",
  "resultMode": "inline"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "xiaohongshu.note.comments",
  "capabilityVersion": 1,
  "input": {
    "contentId": "697c0eee000000000a03c308",
    "sort": "hot"
  },
  "page": { "limit": 20 },
  "freshness": "prefer_cache",
  "resultMode": "auto"
}
```

小红书搜索单次报价较高，先用一页结果判断关键词质量；相关性不足时优先调整关键词，不自动翻页。
搜索结果不等于某个创作者的完整笔记列表。创作者研究先取一页笔记，再按问题选择少量详情或评论，
不自动抓取全部评论页。
