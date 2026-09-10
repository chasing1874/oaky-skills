# 即刻 Social Data v1

仅在研究公开即刻用户和动态时加载本参考。首批三项能力已完成免费 live smoke 并进入 `active`；运行时
状态和报价仍始终以 Registry 为准，只有 `describe_capability` 返回 `active` 时才执行下面的 `invoke` 示例。

## 能力与边界

| capability | 输入 | 单次边界 | 免费数据源 |
| --- | --- | --- | --- |
| `jike.creator.profile@1` | 公开主页的 `creatorId`，即 username slug | 1 个公开创作者 | `https://m.okjike.com/users/<username>` 的 SSR `__NEXT_DATA__` |
| `jike.creator.posts@1` | 同上 | 主页公开展示的一页近期动态，最多 20 条 | 同一个公开主页的 SSR 首屏 |
| `jike.post.detail@1` | 24 位 hex `contentId` + ID 一致的精确 `contentUrl` | 1 条公开动态 | `originalPosts`、`reposts` 或 `posts` 公开详情页 SSR |

三项都只允许一次 `public_web` Attempt，`maxPaidAttempts=0`，没有 TikHub fallback。公开网页不可用、身份
不一致或响应 shape 改变时应直接失败，不能返回猜测数据，也不能为了补结果切换到付费 Provider。

## 身份与 URL

- `creatorId` 使用公开主页 URL 中的 username slug，并以页面 `pageProps.username` / `user.username` 交叉
  校验。不要依赖内部 `user.id`：公开资料页可能不返回它。
- username 只接受字母、数字、下划线和连字符，保留原始大小写。标准主页 URL 为
  `https://m.okjike.com/users/<creatorId>`。
- `contentId` 必须是 24 位十六进制字符串，统一转成小写；不要把展示文本、topic ID 或作者内部 ID 当作
  动态 ID。
- 详情路径是内容身份的一部分：原动态使用 `/originalPosts/<id>`，转发使用 `/reposts/<id>`，其它已知
  类型使用 `/posts/<id>`。仅有 ID 时不能猜路径，因此详情调用要求同时传入精确的 `contentUrl`。
- 先用 `common.entity.resolve` 规范化公开主页或动态链接；短链、任意端口、账号密码、站外 host 和不匹配的
  ID/URL 都不应传给即刻 capability。

## 调用示例

以下示例适用于当前已开放的三项 capability。调用前仍先 `describe_capability`，避免使用过期状态或报价。

```json
{
  "action": "invoke",
  "capabilityId": "jike.creator.profile",
  "capabilityVersion": 1,
  "input": { "creatorId": "jikerij" },
  "freshness": "prefer_cache",
  "resultMode": "inline"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "jike.creator.posts",
  "capabilityVersion": 1,
  "input": { "creatorId": "jikerij" },
  "page": { "limit": 20 },
  "freshness": "prefer_cache",
  "resultMode": "auto"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "jike.post.detail",
  "capabilityVersion": 1,
  "input": {
    "contentId": "5d24336768075a0011cb95bd",
    "contentUrl": "https://m.okjike.com/reposts/5d24336768075a0011cb95bd"
  },
  "freshness": "prefer_cache",
  "resultMode": "inline"
}
```

## 首屏列表与停止条件

- `jike.creator.posts` 只承诺公开主页 SSR 当前展示的首屏近期动态，不代表完整历史，也不自动翻页。
- v1 没有可靠的公开 next cursor。结果返回 `hasMore=false` 和 `nextCursor=null`；传入非空 cursor 应失败，
  不能构造或跨账号复用游标。
- 主页合法返回空 `posts` 时可视为没有公开动态；缺失 `posts` 字段、作者身份不一致或列表中混入其它外层
  作者时属于响应异常，不能伪装成空页。
- 做创作者对标时，默认资料 + 首屏动态已经足够建立近期样本；如果问题要求完整历史，应明确说明当前能力
  不支持，而不是重复请求同一首屏。

## 原动态与转发

- 列表和详情中的 ID、作者、发布时间与互动指标始终属于外层动态。
- 转发动态的外层正文或封面缺失时，可以用 target 的公开正文或首图补足内容展示；不得用 target 的 ID、
  作者或互动指标覆盖外层身份。
- `collected` 是当前查看者状态，不是收藏数；标准 `favoriteCount` 保持 `null`。未提供的播放、弹幕等指标也
  保持 `null`，不能当作 0。

## 价格与验收边界

三项 Provider 成本上限均为 `¥0`，但仍通过 Call/Attempt/Cache/审计链路执行；首版用户售价为资料和详情
`1 点/次`、动态列表 `1 点/页`，用于覆盖标准化、缓存和审计服务。价格以生效的不可变 Pricing Plan 为权威，
本 reference 不作为永久价格源。

现有即刻 parser 和创作者追踪 discovery 只证明仓库已有公开网页解析基础。2026-07-22 已另外在强制清空
TikHub Key 的条件下完成三项 `querySocialData` 免费 live smoke：资料、首屏动态和转发详情各有且只有一次
`public_web` Attempt，三次 Provider 成本均为 0，标准结果非空，总计结算 3 点；跨用户读取被拒绝，输出脱敏，
临时 User、Call、Transaction、Job 均清零，目标 Cache 已删除或恢复。因此三项按逐 capability 证据进入
`active`。

公开 SSR 是网页内部合同，页面改版或限流都可能使能力暂时失败；失败时应保留审计证据并关闭能力，不能把
fixture 成功当作线上可用。

核对来源：

- `https://m.okjike.com/users/jikerij`（公开主页 SSR 样本）
- `https://m.okjike.com/reposts/5d24336768075a0011cb95bd`（公开转发动态 SSR 样本）
