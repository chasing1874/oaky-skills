# B 站能力 v1

已注册的能力（当前是否可用和报价以 `describe_capability` 返回为准）：

- `bilibili.content.search`：关键词搜索视频。
- `bilibili.creator.profile`：用 UP 主 `mid` 读取资料。
- `bilibili.creator.videos`：用 UP 主 `mid` 分页读取投稿列表。
- `bilibili.video.detail`：用 BV/AV 号读取详情。
- `bilibili.video.comments`：用 BV/AV 号读取一级评论。

研究单个 UP 主时，通常先取资料，再取一页近期投稿；只有样本不足、下一页预期能改善覆盖，且不突破用户显式预算时才使用返回的原始 cursor 继续一页。
投稿列表是有界公开样本，不要把一两页写成完整历史。

## 示例

```json
{
  "action": "invoke",
  "capabilityId": "bilibili.creator.profile",
  "capabilityVersion": 1,
  "input": { "creatorId": "203680252" },
  "page": { "limit": 1 },
  "freshness": "prefer_cache",
  "resultMode": "inline"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "bilibili.creator.videos",
  "capabilityVersion": 1,
  "input": { "creatorId": "203680252" },
  "page": { "limit": 20 },
  "freshness": "prefer_cache",
  "resultMode": "inline"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "bilibili.video.comments",
  "capabilityVersion": 1,
  "input": {
    "contentId": "BV18SCrBGE9E",
    "sort": "hot"
  },
  "page": { "limit": 20 },
  "freshness": "prefer_cache",
  "resultMode": "auto"
}
```

B 站查询的数据来源由服务端选择，模型不选择、也不指定来源。跨账号比较时同时记录粉丝量级与发布时间，避免直接用绝对播放量下结论。
