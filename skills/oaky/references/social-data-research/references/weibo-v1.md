# 微博 Social Data v1

仅在研究公开微博账号和作品时加载本参考。首批三项能力已经注册，但截至 2026-07-22 均为
`disabled`：当前只能用 `describe_capability` 查看契约，不能执行下面的 `invoke` 示例。每项完成自己的
付费 live smoke 并切为 `active` 后，才可按本参考调用；运行时状态和报价始终以 Registry 为准。

## 能力与边界

| capability | 输入 | 单次边界 | TikHub endpoint |
| --- | --- | --- | --- |
| `weibo.creator.profile@1` | 十进制字符串 `creatorId` | 1 个公开创作者 | `GET /api/v1/weibo/web_v2/fetch_user_info` |
| `weibo.creator.posts@1` | 十进制字符串 `creatorId` | `feature=3` 的 1 个 provider 页，最多 20 条 | `GET /api/v1/weibo/web_v2/fetch_user_posts` |
| `weibo.post.detail@1` | 十进制字符串 `contentId`；可选匹配的 `contentUrl` | 1 条公开微博 | `GET /api/v1/weibo/web_v2/fetch_post_detail` |

三项当前都没有免费源，单个 Call 只能发起一次付费 Attempt。单帖详情不能调用可能在内部再次付费的
parser fallback；Social Data Worker 直接请求唯一详情 endpoint，避免一次用户调用产生两次供应商费用。

## 身份与 URL

- `creatorId` 必须是稳定的微博 numeric UID，并始终作为正整数字符串传递。昵称、个性域名和主页展示名会变，
  不能作为稳定主键。
- `contentId` 必须是微博的十进制作品 ID，也始终保留为字符串，禁止转成 JavaScript number。
- 标准创作者 URL 为 `https://weibo.com/u/<creatorId>`；标准作品 URL 为
  `https://m.weibo.cn/detail/<contentId>`。
- 桌面端作品链接里的 base62 `mblogid` 需要先无损转换成十进制 ID。不要在模型侧手算；优先使用当前
  active 的 `common.entity.resolve`，再把返回的稳定 ID 传给微博 capability。
- `contentUrl` 只接受微博公开作品长链，且解析出的十进制 ID 必须与 `contentId` 一致。不要传短链、
  非微博域名、任意端口、Cookie、provider 或 endpoint。

## 预备调用示例

以下示例用于锁定未来调用合同，当前 capability 为 `disabled`，不要执行。只有
`describe_capability` 明确返回 `active` 后才可使用。

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

- `weibo.creator.posts` 固定请求 `feature=3`，provider 单页最多 20 条。一次 `invoke` 只取一页，不能为凑满
  数量暗中自动翻页或裁掉半页后跳过记录。
- 下一页只传上次 Call 返回的 opaque `nextCursor`。cursor 内绑定 capability 和 `creatorId`，并保存
  provider 的 `page/since_id`；不能跨账号复用、解析、修改或自行构造。
- 明确返回 `list: []` 且同时带终止分页信号时，才把空页视为合法末页。缺少列表或终止信号的未知 shape
  必须失败，不能伪装成“没有更多”。
- 先取 1 页近期微博。只有第一页无法回答问题、用户明确需要更长时间窗、下一页预期能改善证据，且不突破用户显式预算时，才继续 1 页。

## 主帖与转发

- 列表和详情中的每个标准内容对象都以外层微博的十进制 ID、作者、正文、媒体和互动指标为准。
- 如果外层微博是转发，外层的“转发时评论”和外层互动仍属于这一条微博；被转发原帖的正文、媒体和指标
  不得拼接进外层字段，也不能用原帖 ID 覆盖外层 ID。
- v1 标准输出没有完整的 repost 关系对象时，宁可不投影嵌套原帖，也不要把两条内容混成一条。需要核对
  原帖时，使用公开 canonical URL 单独取证，并把两条证据分别记录。
- 点赞、评论、转发分别映射到标准指标；provider 未提供的指标保持 `null`，不要当成 0。

## 价格与验收边界

2026-07-22 已逐项核对 TikHub OpenAPI endpoint-info：上述三个 endpoint 都是自营接口，
`endpoint_cost=0.001 USD`。仓库 2026-07-13 的微博 creator discovery 记录也使用同一价格。按本项目
`$1 = ¥7.2` 的成本上限口径，三项 Provider 成本均为 `¥0.0072/次`；首版用户售价为资料/详情
`2 点/次`、作品列表 `2 点/页`。价格以生效的不可变 Pricing Plan 为权威，reference 不作为永久价格源。

价格查询没有拉取微博数据，也不是 live smoke。仓库已有的单帖 parser 回归，以及曾使用
`fetch_user_posts` 完成的隔离 PostgreSQL 两轮 creator discovery，只证明旧 parser/discovery 链路在当时样本下
可用；它们没有经过 `querySocialData` 的 Call、Attempt、标准输出、积分结算、用户隔离、脱敏和清理链路，
更没有分别覆盖资料、列表和详情。因此三项仍保持 `disabled`，不能从旧证据推断已经发布。

2026-07-22 的两次独立授权 live run 均在第一项 `weibo.creator.profile` fail-fast：原样本和官方演示 UID
各产生一次 Attempt，Provider 都返回明确“不扣费”的同一错误；实际 Provider 成本与用户结算均为 0，
临时 User、Call、Transaction、active Job 和微博 Cache 回查为 0。列表与详情 endpoint 两次都没有发出。
这说明当前 profile endpoint 尚不可作为可发布能力，也不能把未执行的列表/详情标为已验证。

切换任一 capability 为 `active` 前，必须为该项单独执行带精确费用确认的付费 live smoke，并验证：唯一一次
Attempt、非空标准结果、稳定字符串 ID、分页或详情边界、Provider 成本、2 点结算、返回脱敏、跨用户隔离和
finally 清理。失败后不得在原费用确认下自动重跑。

核对来源：

- `https://api.tikhub.io/openapi.json`（OpenAPI 与 endpoint-info 价格证据）
- `https://docs.tikhub.io/186826054e0`（`get_endpoint_info?endpoint=<provider endpoint>` 查询合同；迁移的
  `source_url` 逐项保存了实际查询 URL，不包含凭证）
- `https://tikhub.io/weibo-api`（微博产品与基础价格说明）
- `https://docs.tikhub.io/381269405e0`（用户资料）
- `https://docs.tikhub.io/381269407e0`（用户微博列表）
- `https://docs.tikhub.io/381269404e0`（单条微博详情）
