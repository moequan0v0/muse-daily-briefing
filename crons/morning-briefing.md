# 定时任务模板：综合中文晨报

- 执行频率：每天
- 建议时间：08:30（你所在时区的早晨）
- mode：`task`
- 时区：默认 `Asia/Singapore`（新加坡）
- 前置：`prompts/morning.md` 已放到 `~/workspace/news-pipeline/prompts/`

## 任务正文（复制到 cron 的 body，原样使用）

---

阅读并严格执行 ~/workspace/news-pipeline/prompts/morning.md 中的全部指令，产出综合中文晨报。任务触发时间即为 Asia/Singapore 时间的"现在"。完成后将晨报正文作为最终消息输出。在正文末尾另起一行，原样输出以下预告（一个字都不要改）：⏭ 下次播报：今日 20:30 · 综合中文晚报
