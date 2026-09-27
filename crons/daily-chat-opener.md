# 定时任务模板：日报旁聊开聊（含 3 天清理）

- 执行频率：每天
- 建议时间：00:41（你所在时区的凌晨）
- mode：`task`
- 时区：默认 `Asia/Singapore`（新加坡）

## 任务正文（复制到 cron 的 body，原样使用）

---

# 日报旁聊开聊（每天凌晨，Asia/Singapore）——账房版

你是用户新闻日报的账房，只负责算账和记账，不直接操作旁聊（cron worker 上下文没有 chat.* / cron.* 工具，不要尝试调用）。
用户的主 agent 会在收到你的最终消息后被唤醒，由它执行实际的开聊、删除旧旁聊和投递切换。

## 1. 确定日期
TODAY=$(TZ=Asia/Singapore date +%F)，格式 YYYY-MM-DD。
CUTOFF=$(TZ=Asia/Singapore date -d "2 days ago" +%F)。

## 2. 读取状态文件
STATE=~/workspace/news-pipeline/state/daily_chats.json：JSON 数组，元素形如 {"date":"YYYY-MM-DD","chat_id":"...或null","status":"pending或done"}。
文件不存在则创建并初始化为 []。

## 3. 幂等检查
如果数组中已有 date == TODAY 且 status == "done" 的条目 → 说明主 agent 已处理过，最终消息只输出一行："今日日报旁聊已就绪，无需操作。" 然后结束，不要写交接文件。

## 4. 登记今日待办
如果没有 date == TODAY 的条目，追加 {"date": TODAY, "chat_id": null, "status": "pending"}。

## 5. 算出待删除清单
delete_list = 数组中所有 date < CUTOFF 的条目（即只保留 TODAY、TODAY-1、TODAY-2 三天），每项保留 {date, chat_id}。
只计算、不删除：删除由主 agent 执行，它会先核对旁聊名称再动手。

## 6. 写交接文件
把以下 JSON 写到 ~/workspace/news-pipeline/state/handoff.json（覆盖写入）：
{"date": "<TODAY>", "create_name": "日报 <TODAY>", "delete": [{"date": "...", "chat_id": "..."}]}
delete 为空时写 []。最后把更新后的数组写回状态文件。

## 7. 最终消息
只输出一行，例如：日报开聊已登记：2026-09-28，待删 1 个旧旁聊，待主 agent 开聊并切换投递。
不要输出其他聊天报告。

---

## 主 agent 收到最终消息后的跟进动作（部署时让你的 agent 照做）

1. 新建名为「日报 <TODAY>」的旁聊；
2. 把「综合中文晨报」「综合中文晚报」两个定时任务的投递目标切换到新旁聊；
3. 按 delete 清单删除 3 天前的旧日报旁聊（动手前先核对旁聊名称）；
4. 把新旁聊的 chat_id 回填到 `~/workspace/news-pipeline/state/daily_chats.json` 对应条目，status 置为 `done`。
