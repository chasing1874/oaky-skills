# 通用研究工作流

## 选择工具

- 用户自己的收藏、字幕、正文与笔记：`queryClips`。
- Registry 当前已启用的平台公开数据：`querySocialData`；先搜索 capability 确认实时范围。
- 普通网页事实与新闻：`webSearch`。

三者可以组合，但不要用公开社媒查询替代用户收藏，也不要把搜索引擎摘要当作平台指标。

## 从链接开始

当用户只给主页、作品或短链接：

```json
{
  "action": "invoke",
  "capabilityId": "common.entity.resolve",
  "capabilityVersion": 1,
  "input": { "url": "https://www.bilibili.com/video/BV18SCrBGE9E/" },
  "page": { "limit": 1 },
  "freshness": "prefer_cache",
  "resultMode": "inline"
}
```

使用返回的稳定 creatorId/contentId 调当前 active 的平台能力。不要自己从短链接猜 ID；解析器不支持该平台时
保留原链接并改走当前可用的搜索或用户已提供证据。

链接查询参数中的设备 ID、分享者 ID、会话 ID 或跟踪 ID 都不能当作 creatorId。解析结果只有 contentId、
没有 creatorId 时，先读取该作品详情，并从标准输出的 `item.creator.platformCreatorId` 取得创作者 ID；
如果详情也没有，就保留缺口，不从原始 query string 猜值。

研究用户收藏中已有的特定创作者时，先用 `queryClips(action=list_creators)` 查看完整紧凑目录，再用选中条目的
`creatorId` 调 `queryClips(action=list_by_creator)`；需要平台公开数据时使用条目的
`platformUserId`，不要混用两个 ID，也不要先依赖全文或语义搜索猜作者。`queryClips(search)`
和平台内容搜索返回的都只是候选，必须用作者显示名、主页 URL 或稳定 platformCreatorId 核验后才能放进目标
证据集；用户自身或其他创作者的内容要单独归类，不能混写成目标事实。

## 研究计划模板

在调用前内部列出：

| 目的 | capability | 页数 | 预计点数 | 停止条件 |
| --- | --- | ---: | ---: | --- |
| 确认对象 | common.entity.resolve | 0-1 | 当前报价 | 已有稳定 ID 时跳过 |
| 账号背景 | active 的 creator profile | 0-1 | 当前报价 | 未启用时跳过并记录缺口 |
| 作品样本 | active 的 creator posts/videos/notes | 0-2 | 当前报价乘页数 | 未启用或样本足够时停止 |
| 深挖作品 | active 的 content/video/note detail | 0-3 | 当前报价乘条数 | 已覆盖代表样本 |
| 受众反馈 | active 的 content/video/note comments | 0-3 | 当前报价乘页数 | 主题饱和，或继续一页的边际收益很低 |

用户给出预算时，按当前报价计算下一步是否仍在预算内；会超出时先缩小问题并用已有证据交付。用户未给预算时，
不补默认点数或调用次数上限，依据证据充分性、样本覆盖和新增页的边际收益自动停止。

## 分页

- 外部列表返回 `page.nextCursor` 时，下一页仍调用 `invoke`，原样传 cursor。
- 不解析、不修改、不跨 capability 复用 cursor。
- 临时 Dataset 用 `get_result(callId, cursor)` 分页读取；这只是读取同一结果。
- 任何时候都不能因为 `hasMore=true` 自动抓到底。

## 失败恢复

- `invalid_capability_input`：调用 `describe_capability` 后只修正报错字段。
- `feature_disabled`：停止本轮外部社媒查询，并说明当前环境尚未开放该工具能力。
- `capability_disabled`：停止该能力，重新搜索 active capability；不要换 endpoint 或绕过状态。
- `invalid_cursor`：回到上一页返回的原始 cursor；找不到则停止，不从头制造重复费用。
- `provider_*` 或访问繁忙：说明覆盖缺口；除非工具明确建议重试，不立即循环重试。
- 结果为 0 条：先检查 ID/关键词和平台，再决定换关键词；0 条不等于“平台没有相关内容”。
- `status=running`：稍后用 `get_result` 查询同一个 callId，不重复 `invoke`。
