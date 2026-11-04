# 知乎能力 v1

已注册的能力（每项独立发布，当前是否可用和报价以 `describe_capability` 返回为准）：

- `zhihu.creator.profile`：用用户主页 `url_token` 读取公开创作者资料。
- `zhihu.creator.posts`：显式选择 `article` 或 `pin`，读取对应的一页内容。
- `zhihu.content.detail`：显式选择 `answer`、`article`、`question` 或 `pin`，读取单条详情。

## 使用边界

- 列表的 `contentType` 必填。一次调用只读取文章或想法中的一种、一页数据，不能把两类内容暗中合并为两次计费请求。
- 回答列表不在 `zhihu.creator.posts@1` 的公开 schema 中，当前没有可靠的按作者列回答方式；服务端会在计费前
  拒绝 `contentType: "answer"`。需要回答时，用已知回答链接逐条读取详情，并说明不是完整回答列表。
- 详情的 `contentType` 必填。服务端只按该类型读取，不猜测类型，也不因一种类型返回空数据而尝试另一种类型。
- 资料为空、详情没有有效公开字段或列表结构未知时按失败处理。列表明确返回空数组且分页已结束时是合法空页，
  运行时返回 `items: []`。
- 游标只续取同一个创作者、同一种内容类型的下一页；不要自动遍历完整历史。
- 标准结果不包含数据源 endpoint、request ID、缓存地址、鉴权字段或原始响应。

## 示例

以下示例只在 `describe_capability` 返回对应能力可用时执行。

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
