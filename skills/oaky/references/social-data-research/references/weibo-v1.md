# 微博 Social Data v1

仅在研究公开微博账号和作品时加载本参考。三项能力已经注册，但是否可调用、报价和限制以
`describe_capability` 返回为准；返回 `disabled` 时只能查看契约，不能执行下面的 `invoke` 示例。

## 能力与边界

| capability | 输入 | 单次边界 |
| --- | --- | --- |
| `weibo.creator.profile@1` | 十进制字符串 `creatorId` | 1 个公开创作者 |
| `weibo.creator.posts@1` | 十进制字符串 `creatorId` | 1 页，最多 20 条 |
| `weibo.post.detail@1` | 十进制字符串 `contentId`；可选匹配的 `contentUrl` | 1 条公开微博 |

每次调用都会计费；一次 `invoke` 只对应一次读取，不要为同一对象重复调用来「补数据」。

## 身份与 URL

- `creatorId` 必须是稳定的微博 numeric UID，并始终作为正整数字符串传递。昵称、个性域名和主页展示名会变，
  不能作为稳定主键。
- `contentId` 必须是微博的十进制作品 ID，也始终保留为字符串，禁止转成 JavaScript number。
- 标准创作者 URL 为 `https://weibo.com/u/<creatorId>`；标准作品 URL 为
  `https://m.weibo.cn/detail/<contentId>`。
- 桌面端作品链接里的 base62 `mblogid` 需要先无损转换成十进制 ID。不要在模型侧手算；优先使用当前
  active 的 `common.entity.resolve`，再把返回的稳定 ID 传给微博 capability。
- `contentUrl` 只接受微博公开作品长链，且解析出的十进制 ID 必须与 `contentId` 一致。不要传短链、
  非微博域名、任意端口、Cookie、数据源或 endpoint。

## 调用示例

只有 `describe_capability` 明确返回 `active` 后才可执行以下示例。

```json
{
  "action": "invoke",
  "capabilityId": "weibo.creator.profile",
  "capabilityVersion": 1,
  "input": { "creatorId": "1722594714" },
  "freshness": "prefer_cache",
  "resultMode": "inline"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "weibo.creator.posts",
  "capabilityVersion": 1,
  "input": { "creatorId": "7277477906" },
  "page": { "limit": 20 },
  "freshness": "prefer_cache",
  "resultMode": "auto"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "weibo.post.detail",
  "capabilityVersion": 1,
  "input": {
    "contentId": "5092682368025584",
    "contentUrl": "https://m.weibo.cn/detail/5092682368025584"
  },
  "freshness": "prefer_cache",
  "resultMode": "inline"
}
```

## 分页与停止条件

- `weibo.creator.posts` 单页最多 20 条。一次 `invoke` 只取一页，不能为凑满数量暗中自动翻页或裁掉半页后
  跳过记录。
- 下一页只传上次 Call 返回的 opaque `nextCursor`。cursor 绑定 capability 和 `creatorId`；不能跨账号复用、
  解析、修改或自行构造。
- 明确返回 `list: []` 且同时带终止分页信号时，才把空页视为合法末页。缺少列表或终止信号的未知 shape
  必须失败，不能伪装成“没有更多”。
- 先取 1 页近期微博。只有第一页无法回答问题、用户明确需要更长时间窗、下一页预期能改善证据，且不突破用户显式预算时，才继续 1 页。

## 主帖与转发

- 列表和详情中的每个标准内容对象都以外层微博的十进制 ID、作者、正文、媒体和互动指标为准。
- 如果外层微博是转发，外层的“转发时评论”和外层互动仍属于这一条微博；被转发原帖的正文、媒体和指标
  不得拼接进外层字段，也不能用原帖 ID 覆盖外层 ID。
- v1 标准输出没有完整的 repost 关系对象时，宁可不投影嵌套原帖，也不要把两条内容混成一条。需要核对
  原帖时，使用公开 canonical URL 单独取证，并把两条证据分别记录。
- 点赞、评论、转发分别映射到标准指标；数据源未提供的指标保持 `null`，不要当成 0。
