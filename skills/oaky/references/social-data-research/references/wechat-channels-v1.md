# 视频号 v1

## 当前能力

视频号首批 4 项能力已经完成逐能力真实付费 smoke，当前均为 `active`：

- `wechat_channels.creator.profile@1`
- `wechat_channels.creator.videos@1`
- `wechat_channels.video.detail@1`
- `wechat_channels.video.comments@1`

每次仍先通过 Registry 确认 effective status；运行时总开关或 allowlist 可能继续收窄可用范围。不可用时记录覆盖
缺口，不要改用供应商 endpoint 绕过。

## 稳定标识与分页

- 创作者 ID 是 `v2_…@finder` 格式的 finder username，不是显示昵称或 `sph…` 短号。
- 作品 `objectId` 和 `objectNonceId` 是可能超过 JavaScript 安全整数范围的 64 位整数，必须以字符串传递。
- 作品列表与评论接口返回 provider 固定页；Registry 将请求页长收敛为 20，避免裁掉一页后直接跨游标丢数据。
- `nextCursor` 是 capability 绑定的不透明游标，只能原样回传给同一能力。

## 研究顺序

单账号研究优先按“资料 → 一页近期作品 → 少量代表作品详情 → 必要时一页一级评论”执行。每项当前报价
9 点；先根据研究问题选择必要证据，只有新内容会改善结论时才继续详情、评论或下一页。

详情响应可能包含短时媒体地址和解密信息，但统一输出只保留公开元数据、封面、稳定链接和互动指标，不输出
`decode_key`、临时 token 或供应商 request id。评论只读取一级页，不自动展开二级回复或抓完全部历史。

## 示例

```json
{
  "action": "invoke",
  "capabilityId": "wechat_channels.creator.profile",
  "capabilityVersion": 1,
  "input": {
    "creatorId": "v2_060000231003b20faec8c4e7811fcbd0c705e435b07740d41e3e6f7d05e1a1eee4c83a9b3c34@finder"
  },
  "freshness": "prefer_cache",
  "resultMode": "inline"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "wechat_channels.creator.videos",
  "capabilityVersion": 1,
  "input": {
    "creatorId": "v2_060000231003b20faec8c4e7811fcbd0c705e435b07740d41e3e6f7d05e1a1eee4c83a9b3c34@finder"
  },
  "page": { "limit": 20 },
  "freshness": "prefer_cache",
  "resultMode": "auto"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "wechat_channels.video.detail",
  "capabilityVersion": 1,
  "input": { "contentId": "14968135489986697747" },
  "freshness": "prefer_cache",
  "resultMode": "inline"
}
```

```json
{
  "action": "invoke",
  "capabilityId": "wechat_channels.video.comments",
  "capabilityVersion": 1,
  "input": { "contentId": "14968135489986697747" },
  "page": { "limit": 20 },
  "freshness": "prefer_cache",
  "resultMode": "auto"
}
```
