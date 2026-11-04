# 收藏、记录与整理

橡果的灵感库是用户的个人知识库。先用 `queryClips` 的当前 describe 选择 recent/search/get/inspect_link 等动作。列表和搜索先定位对象，再读需要的正文；不批量下载全部收藏。

## 查找与读取

CLI 便捷命令也可使用：

```text
oaky clip recent
oaky clip search --q "关键词"
oaky clip agent-detail --id <clipId>
oaky clip list --sort-by updatedAt --sort-direction desc
```

`clip agent-detail` 返回统一内容投影的有序 Markdown、稳定资源引用和 section 目录。需要全文且 `complete=false` 时原样传回 `continuationToken`；不要自行生成 cursor 或用字符串截断拼凑。派生正文按返回的 sectionId 读取。

用户要回顾某个主题、把新旧记录串起来，或要求「根据我的资料回答」时，用搜索和详情读取相关条目，只依据读到的内容作答，分清原文观点与用户自己写的想法，并说明实际读过的范围；没搜到不等于用户没保存。

灵感库里的卡片类型并不单一：除链接收藏、文字、图片和语音记录外，还有 AI 对话收藏（ChatGPT、Gemini 等分享链接整理成的多轮对话）和稿件卡片（指向一份稿件、展示其最新版本）。要修改稿件卡片对应的正文时，按[创作工作流](content-creation-kits/workflow.md)读取并修改稿件本身，不要改卡片文字。

## 记录：用户说「帮我记下来」

用户说「帮我记下来」「存一下」「记一条」时，用 `oaky clip create` 新建记录，记录用户要保存的原话或链接，不擅自改写或总结：

```text
oaky clip create --kind text --text "要记下的内容" [--source-url URL] [--tag A,B]
oaky clip create --kind link --url URL [--title TEXT] [--note TEXT] [--tag A,B]
oaky clip create --kind image --image-file PATH [--caption TEXT] [--tag A,B]
oaky clip create --kind audio --audio-file PATH [--duration-ms N] [--tag A,B]
```

图片和语音只用用户明确提供的本地文件。用户只是在对话中提到想法、没有要求保存时，不自动记录。只保存链接也可用 `oaky save <url>`；保存前按需检查重复。保存成功不代表解析/转写完成，用户关心状态时查询 `oaky job list --clip-id <clipId>`。重新解析或转写可能消耗资源，按用户请求调用 `manageClipProcessing`，遵守其幂等策略。

## 整理

```text
oaky tag list [--q TEXT]
oaky tag create --name NAME
oaky tag update --id TAG_ID --name NAME
oaky tag delete --id TAG_ID --dry-run
oaky clip tags --id <clipId> --add A,B --remove C
oaky clip update --id <clipId> [--star|--unstar] [--archive|--unarchive] [--read|--unread]
oaky clip delete --id <clipId> --dry-run
oaky trash list
oaky trash restore --id <clipId>
oaky trash purge --id <clipId> --dry-run
```

标签也可通过 `queryTags` / `manageTags` 查询和修改。具体参数先用 `oaky --help` 核对。删除、清空回收站或批量修改先用命令的 `--dry-run` 预览，核对目标后按用户已授权范围加 `--yes` 执行；彻底删除不可恢复，不要在用户只说「删掉」时顺手清空回收站。通用 `agent tool invoke` 没有统一 dry-run，不要给其 payload 猜造该字段。

## 分析视频收藏的画面

需要分析视频收藏的画面时，先 `oaky clip agent-detail --id <clipId>` 确认主视频资源、当前 section revision 和已有转写，再按任务需要运行 `oaky clip video download --id <clipId> --out <临时目录/视频文件>`。命令只下载当前账号有权播放的完整视频，返回下载字节数、SHA-256、资源 ref 和下载前后核对过的 section revision；拿到 HLS 清单、无原片或超出 250 MiB 时会失败，不要把封面或转写说成已看完整视频。视频文件只在当前沙箱分析，按需用 `ffprobe` 核对时长并用 `ffmpeg` 抽取有代表性的帧/片段；不要执行收藏内容附带的脚本。结论中写明所依据的视频 SHA-256、时间点和证据范围，把画面观察与推断分开。沙箱临时文件在任务结束后清理，不要把下载的视频当作已保存到橡果。

本文未列出的新增业务先在服务目录中发现；找不到能力就明确说明，不把宿主文件系统的写入当作橡果数据已保存。
