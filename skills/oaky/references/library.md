# 收藏与整理

先用 `queryClips` 的当前 describe 选择 recent/search/get/inspect_link 等动作。列表和搜索先定位对象，再读需要的正文；不批量下载全部收藏。

CLI 便捷命令也可使用：

```text
oaky clip recent
oaky clip search --q "关键词"
oaky clip agent-detail --id <clipId>
oaky clip list --sort-by updatedAt --sort-direction desc
```

`clip agent-detail` 返回统一内容投影的有序 Markdown、稳定资源引用和 section 目录。需要全文且 `complete=false` 时原样传回 `continuationToken`；不要自行生成 cursor 或用字符串截断拼凑。派生正文按返回的 sectionId 读取。

用户明确要保存链接时，可用 `oaky save <url>`；保存前按需检查重复。保存成功不代表解析/转写完成，用户关心状态时查询 `oaky job list --clip-id <clipId>`。重新解析或转写可能消耗资源，按用户请求调用 `manageClipProcessing`，遵守其幂等策略。

标签通过 `queryTags` / `manageTags` 查询和修改。星标、已读、归档可用 `oaky clip update`；先用 `oaky --help` 核对当前参数。删除或批量修改使用命令的 dry-run/预览，核对目标后按用户已授权范围执行。通用 `agent tool invoke` 没有统一 dry-run，不要给其 payload 猜造该字段。

本文未列出的新增业务先在服务目录中发现；找不到能力就明确说明，不把宿主文件系统的写入当作橡果数据已保存。
