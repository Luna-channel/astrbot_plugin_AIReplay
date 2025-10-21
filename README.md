# AIReplay — AstrBot 主动续聊与提醒插件

> **作者**：LumineStory  
> **仓库**：<https://github.com/oyxning/astrbot_plugin_AIReplay>  

AIReplay 为 AstrBot 提供「**定时/间隔主动续聊**」「**人格上下文**」「**最近聊天记录拼接**」「**免打扰**」与「**定时提醒**」能力。它可以在**最后一条消息后的指定时长**或**每天固定时间**，由 Bot 主动自然切入对话；也可以设置**免打扰时段**、**追加时间字段**，以及为会话创建**一次性/每日提醒**。

---

## ✨ 功能特性

- **两类主动触发**
  - **间隔触发**：最后一条消息后 *N* 分钟自动续聊（`after_last_msg_minutes`）。
  - **每日触发**：每天两档固定时间（`daily.time1` / `daily.time2`），如冲突自动错峰 1 分钟。
- **自然续聊**
  - 继承或覆盖**人格**（system prompt）。
  - 携带**最近 N 条聊天记录**构成上下文（`history_depth`）。
  - 可配置**自定义提示词**（支持占位符：`{now}`、`{last_user}`、`{last_ai}`、`{umo}`）。
- **免打扰**
  - `quiet_hours` 支持 **跨天**区间（如 `22:00-07:30`）。免打扰内不触发主动回复。
- **时间字段**
  - 可选在主动消息前追加 `[YYYY-MM-DD HH:MM]` 等时间标签（`append_time_field` + `time_format`）。
- **定时提醒**
  - 一次性/每日提醒：`/aireplay remind add ...`，到点直接向会话发送提示。
- **订阅模型**
  - 仅向**已订阅会话**发送主动消息。支持 `manual`（默认）与 `auto` 两种订阅方式。

> 说明：不同 IM 平台对“主动消息”的支持情况不同，请以 AstrBot 对目标平台的支持为准。

---

## 📦 安装

1. 从 Release 或本仓库获取插件包，解压到：  
   `AstrBot/data/plugins/astrbot_plugin_aireplay/`
2. 启动（或重启）AstrBot。
3. 进入 WebUI → **插件** → 启用 **AIReplay**。
4. （可选）在插件配置页完成参数设置。

目录结构示例：

```
AstrBot/
└─ data/
   └─ plugins/
      └─ astrbot_plugin_aireplay/
         ├─ main.py
         ├─ _conf_schema.json
         ├─ metadata.yaml
         ├─ README.md
         └─ requirements.txt
```

## 🧩 配置项（`_conf_schema.json`）

| 键名 | 类型 | 默认值 | 说明 |
|---|---|---:|---|
| `enable` | bool | `true` | 启用/停用插件。 |
| `timezone` | string | `""` | IANA 时区（示例 `Asia/Shanghai`、`America/Los_Angeles`）。为空使用系统时区。 |
| `after_last_msg_minutes` | int | `0` | 最后一条消息后 N 分钟主动续聊。`0` 代表关闭此触发器。 |
| `daily.time1` | string | `""` | 每日触发时间 1（`HH:MM`，24h）。 |
| `daily.time2` | string | `""` | 每日触发时间 2（`HH:MM`）。若与 `time1` 相同会自动 **+1 分钟** 错峰。 |
| `quiet_hours` | string | `""` | 免打扰时段（`HH:MM-HH:MM`），支持跨天。免打扰内不主动触发。 |
| `history_depth` | int | `8` | 携带最近聊天条数（从当前会话历史提取）。 |
| `persona_override` | text | `""` | 覆盖默认人格的 System Prompt（留空使用当前会话人格）。 |
| `custom_prompt` | text | `""` | 自定义提示词模板（占位符：`{now}`、`{last_user}`、`{last_ai}`、`{umo}`）。 |
| `append_time_field` | bool | `false` | 在主动消息前追加时间字段。 |
| `time_format` | string | `"%Y-%m-%d %H:%M"` | 时间字段格式（Python `strftime`）。 |
| `subscribe_mode` | string(`manual`/`auto`) | `manual` | 订阅方式：`manual` 需要手动 `/aireplay watch`；`auto` 有互动即自动纳入。 |
| `_special.provider` | string | `""` | （可选）固定使用的 Provider ID，留空按会话默认。 |
| `_special.persona` | string | `""` | （可选）固定人格 ID，留空按会话默认。 |

### 自定义提示词占位符示例

```text
你是暖心助理，请以关怀口吻自然续聊。
时间：{now}
用户上一句：{last_user}
你的上一句：{last_ai}
会话ID：{umo}
请给一个小建议并追问一句。
```

---

## 🚀 快速上手

1. 在要“主动续聊”的会话里发送：

```
/aireplay on
/aireplay watch
```

2. 设定触发方式（二选一或同时使用）：

- **间隔触发**（最后消息后 30 分钟）  
  ` /aireplay set after 30 `

- **每日触发**（每天 09:00 与 19:00）  
  ```
  /aireplay set daily1 09:00
  /aireplay set daily2 19:00
  ```

3. （可选）设置免打扰：  
   ` /aireplay set quiet 22:00-07:30 `

4. （可选）设置自定义提示词、历史深度：  
   ```
   /aireplay set prompt 你是暖心助理，参考用户上一句与上下文，自然续聊并提供一个贴心建议。
   /aireplay set history 8
   ```

---

## 💬 指令全集

- 基础
  - `/aireplay on` / `/aireplay off` — 启用/停用
  - `/aireplay watch` / `/aireplay unwatch` — 订阅/退订当前会话（仅订阅会话会接收主动消息）
  - `/aireplay show` — 查看当前会话订阅状态与核心配置摘要
- 触发设置
  - `/aireplay set after <分钟>` — 设置“最后消息后 N 分钟触发”；设为 `0` 可关闭
  - `/aireplay set daily1 <HH:MM>` — 设置每日触发时间 1
  - `/aireplay set daily2 <HH:MM>` — 设置每日触发时间 2
  - `/aireplay set quiet <HH:MM-HH:MM>` — 设置免打扰时段（可跨天）
  - `/aireplay set history <N>` — 设置携带最近聊天条数
  - `/aireplay set prompt <文本>` — 设置自定义提示词（支持占位符）
- 提醒
  - `/aireplay remind add <YYYY-MM-DD HH:MM> <内容>` — 新增**一次性**提醒
  - `/aireplay remind add <HH:MM> <内容> daily` — 新增**每日**提醒
  - `/aireplay remind list` — 查看提醒列表
  - `/aireplay remind del <ID>` — 删除提醒

---

## 🧠 工作原理（简述）

- **调度循环**：插件内部维护一个 ~30 秒的调度循环 tick：
  1. 若 `enable=false`，跳过。
  2. 读取时区、免打扰、历史深度与每日时间点。
  3. 对每个**已订阅**会话：
     - 若当前处于免打扰，跳过。
     - 若距离最后消息已超过 `after_last_msg_minutes`，触发“间隔续聊”。
     - 若当前分钟命中 `daily.time1/time2`，触发“每日续聊”。
     - 利用“上次触发标签”去重（同一分钟不重复）。
  4. 扫描**提醒**：到点发送一次性提醒；每日提醒按 HH:MM 命中即发。
- **上下文拼接**：从 `ConversationManager` 取会话历史；若不可用则退化使用本插件的轻量历史缓存。
- **人格策略**：默认沿用当前会话人格，可通过 `persona_override` 或 `_special.persona` 覆盖。
- **Provider 选择**：默认使用会话当前 Provider，可通过 `_special.provider` 固定。

---

## 🧪 使用建议

- **避免骚扰**：生产环境建议维持 `subscribe_mode=manual`，只对明确订阅的会话主动续聊。  
- **时间与时区**：跨时区部署时建议显式设置 `timezone`，并在 UI 中确认每日触发时刻。  
- **自定义提示词**：结合 `{last_user}` 与 `{last_ai}` 能显著提高“自然切入”感。  
- **历史深度**：`history_depth` 不宜过大，以免模型成本上涨或上下文“漂移”。  

---

## 🔧 故障排查

- **没有主动消息？**
  - 检查 `/aireplay show` 是否显示 `subscribed=true`、`enable=true`。
  - 目标平台是否支持“主动消息”？（以 AstrBot 支持为准）
  - 处于免打扰时段？触发时刻是否刚好被挡住？
  - `after_last_msg_minutes=0` 且未设置 `daily.time1/time2`？
  - `timezone` 是否正确？
- **续聊内容空白或很机械？**
  - 添加/优化 `custom_prompt`。
  - 提高或降低 `history_depth`，并检查是否携带到上一轮用户与 AI 内容。
  - 考虑设定更贴合任务的人格（`persona_override`）。
- **提醒未触发？**
  - 一次性提醒使用本地时间字符串对比（精确到分钟）；检查格式与分钟级命中。  
  - 每日提醒需在命中 `HH:MM` 时刻才触发。

---

## 📄 示例

**设置“最后消息后 45 分钟续聊”，每日 09:00/20:00 提醒关心**

```
/aireplay on
/aireplay watch
/aireplay set after 45
/aireplay set daily1 09:00
/aireplay set daily2 20:00
/aireplay set prompt 你是温柔体贴的生活助理，请在不打扰的前提下继续话题，并给一个可执行的小建议。
```

**添加提醒**

```
/aireplay remind add 2025-10-22 09:30 项目早会
/aireplay remind add 21:45 休息眼睛 daily
```

---

## 🗂️ 数据与持久化

- 插件数据目录：`AstrBot/data/plugins/astrbot_plugin_aireplay/`
  - `state.json`：会话订阅状态、上次触发标签等。
  - `reminders.json`：提醒列表。

> 文件仅存储必要元数据，不保存模型输出。请注意服务器文件权限和备份策略。

---

## 🤝 贡献

欢迎 Issue 与 PR！
- Repo：<https://github.com/oyxning/astrbot_plugin_AIReplay>
- 建议方向：多时区会话粒度覆盖、更多重复触发保护策略、可视化提醒管理、更多 Scheduler 精度选项。

---

## 🧾 版本与变更

- **1.0.0**：首个稳定版。支持间隔/每日续聊、免打扰、时间字段、自定义提示词、提醒、订阅机制与人格/历史拼接。

---

## 📜 许可证

MIT
