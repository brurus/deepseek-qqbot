# DeepSeek QQBot

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

DeepSeek QQBot 是一个基于 [LLOneBot](https://github.com/LLOneBot/LLOneBot) 和 [DeepSeek API](https://platform.deepseek.com/) 的智能 QQ 机器人，提供强大的聊天、问答等功能，专为群聊对话场景设计。

内置三条 API 车道，可随时切换：**DeepSeek 官方**、**腾讯云 lkeap**、**Kilo 免密免费车道（不用 Key、不花钱）**。

---

## 前置条件

1. 已安装 [LLOneBot](https://github.com/LLOneBot/LLOneBot) 或兼容 OneBot v11 协议的软件，并设置上报地址为 `http://0.0.0.0:8080`。
2. 准备一个 API Key（**可选**）：
   - 用 DeepSeek 官方：在 [DeepSeek 开放平台](https://platform.deepseek.com/) 申请。
   - 用腾讯云：在 [腾讯云 lkeap](https://cloud.tencent.com/product/lkeap) 申请。
   - **不想申请、不想花钱**：启动时选 `2` Kilo 免密免费车道，或进群后发 `/set api 2`，零凭据直接开跑（详见 [免密免费车道](#免密免费车道kilo)）。

---

## 特点

- **强大 AI 加持**：基于 DeepSeek 的先进语言模型，提供高质量的对话体验。
- **三条车道热切换**：官方 / 腾讯 / 免密免费，群内一句指令切换，无需重启。
- **上下文记忆**：支持保存和追踪对话上下文，提升交互连贯性。
- **深度思考**：支持 R1 推理模型，可选择展示思考过程。
- **内置小工具**：集成了多种实用功能，满足多样化需求。
- **群聊优化**：针对群聊场景专门优化，确保流畅体验。
- **两级管理员**：全局管理员 + 群管理员，指令权限可控。

---

## 安装步骤

### 1. 克隆仓库

```bash
git clone https://github.com/brurus/deepseek-qqbot.git
cd deepseek-qqbot
```

### 2. 创建虚拟环境

```bash
python -m venv .env
# 激活虚拟环境
source .env/bin/activate  # Linux/MacOS
.\.env\Scripts\activate   # Windows
```

### 3. 安装依赖

```bash
pip install -r requirements.txt
```

### 4. 启动机器人

```bash
python app/bot.py
```

### 5. 按提示输入管理员账号和 API Key

![image](https://github.com/user-attachments/assets/d1c021a9-67f2-47d2-a425-9b6440460664)

依次输入：

1. **管理员 QQ 号**（多个用英文逗号隔开）
2. **选择 API 车道**：`0` DeepSeek 官方 / `1` 腾讯云 / `2` Kilo 免密免费车道
3. **API Key**（选 `2` 时跳过，直接回车即可）

配置会写入 `app/config/config.yml`，随时可手动修改。

成功启动后，机器人会开始监听消息并响应指令。

---

## 配置说明

`app/config/config.yml` 主要字段：

```yaml
chat:
  admins: []                 # 全局管理员 QQ 号
  api: deepseek              # 当前车道：deepseek / tencent / kilo
  model: deepseek-chat       # 默认模型
  stream: true               # 流式输出
  max_tokens: 4              # 最大回答长度（单位 k，不超过 8）
  show_reasoning_content: false
  temperature: 1.3           # 仅 V3 生效，R1 不接受此参数

bot:
  base_url: http://localhost:3000   # LLOneBot 地址

server:
  host: 0.0.0.0
  port: 8080                 # 与 LLOneBot 上报端口保持一致

api_list:                    # 车道地址
  deepseek: https://api.deepseek.com
  tencent: https://api.lkeap.cloud.tencent.com/v1
  kilo: https://api.kilo.ai/api/gateway

key_list:                    # 各车道 API Key，免密车道留空
  deepseek: ""
  tencent: ""
  kilo: ""

keyless: [kilo]              # 无需鉴权的车道

model_list:                  # 各车道实际下发给上游的模型名
  deepseek: {chat: deepseek-chat,  reasoner: deepseek-reasoner}
  tencent:  {chat: deepseek-v3,    reasoner: deepseek-r1}
  kilo:     {chat: nvidia/nemotron-3-ultra-550b-a55b:free,
             reasoner: nvidia/nemotron-3-nano-omni-30b-a3b-reasoning:free}
```

> 老版本配置文件升级时，缺失的字段会在读取时自动用默认值补齐，不会报错。

---

## 免密免费车道（Kilo）

不想填 API Key、也不想花钱，可以切到内置的第 3 条车道：

```
/set api 2
```

- 上游是 [Kilo AI 公共网关](https://kilo.ai) 的免费模型池，插件式聚合、无需登录，**不带任何凭据即可调用**。
- 当前默认模型写在 `config.yml` 的 `model_list.kilo` 里，可自行替换成免费池里的其它模型。查看上游有哪些免费模型：

  ```bash
  curl https://api.kilo.ai/api/gateway/models
  ```

  取返回里 `isFree: true` 的模型 id，填进 `model_list.kilo.chat` / `reasoner` 即可。
- 首次启动也可以直接选 `2`，省掉填 Key 这一步。

**用之前请想清楚三件事：**

1. **上游声明免费池的 prompt 可能被记录用于改进其服务**，群聊内容等同于发给了第三方。别在这个车道上聊敏感内容。
2. **免费池随时可能变动**：模型 id 会失效（上游返回 404/502/429），聚合网关的模型名与实际后端也不总是一一对应（回复里自报家门可能对不上名）。出问题就把模型换成上面列表里还能用的。
3. **稳定性不如官方**：本车道适合测试和玩票；长期跑群建议还是用 DeepSeek 官方 Key（很便宜，且稳定）。

---

## 使用方法

### 1. 开启机器人

在需要使用的群聊中发送：

```
/open
```

机器人会在群聊中开启服务，并向管理员发送私信确认。

### 2. 关闭机器人

```
/close
```

机器人会停止当前群聊的服务，并向管理员发送私信确认。

---

## 指令文档

| 指令        | 功能描述                                   | 备注      |
|-------------|------------------------------------------|---------------|
|             | **群组管理员指令**
| `/open`     | 开启当前群聊机器人服务，启用对话功能。          |      |
| `/close`    | 关闭当前群聊机器人服务，暂停对话功能。          |      |
| `/clear`    | 清空上下文记录，重置机器人状态。            |      |
| `/model v3` | 切换当前群组模型为V3                      | 也可用简写`/m v3`|
| `/model r1`  | 切换当前群组模型为R1                     | 也可用简写` /m r1` |
| `/model default`  | 切换当前群组模型为默认（跟随全局）    | 也可用简写` /md` |
| `/think open`  | 展示当前群组R1模型的思考过程              | 也可用简写`/to`       |
| `/think close`  | 关闭当前群组R1模型的思考过程              | 也可用简写`/tc`       |
| `/think default`  | 恢复为跟随全局设置           | 也可用简写`/td`       |
| `/trim`  | 清除冗余信息，清理掉与机器人无关的信息（没有艾特机器人的消息）              |        |
|            | **管理员指令**
| `/set api [id]`    | 更换调用的api，切换前需已配置对应key           |  `id:0` 官方api  <br> `id:1` 腾讯api  <br> `id:2` Kilo免密免费车道（不需要key）  |
| `/balance`  | 查询 DeepSeek API 账户余额。             |     |
| `/admin add [qq]`  | 添加群管理员             |    后可跟多个QQ号，需用`空格`隔开|
| `/admin rm [qq]`  | 移除群管理员             |    后可跟多个QQ号，需用`空格`隔开|
| `/global model v3` | 切换全局模型为V3                      | 也可用简写`/gm v3`|
| `/global model r1`  | 切换全局模型为R1                     | 也可用简写` /gm r1` |
| `/global think open`  | 开启全局R1模型的思考过程              | 也可用简写`/gto`       |
| `/global think close`  | 关闭全局R1模型的思考过程              | 也可用简写`/gtc`       |
| `/stream open`  | 开启流式输出模式           | 也可用简写`/so`       |
| `/stream close`  | 关闭流式输出模式              | 也可用简写`/sc`       |

> `/set api` 的 id 按 `config.yml` 中 `api_list` 的顺序取，增删车道后编号会随之变化。

---

## 常见问题

**Q：启动时报「请确定是否启动了LLonebot」？**
先启动 LLOneBot，确认 `bot.base_url` 填的地址能访问，再启动本程序。

**Q：机器人不回消息？**
确认群里发过 `/open`；确认当前车道的 Key 已配置（`/balance` 可验证官方车道余额）；未配置 Key 时会回复「未配置API Key，请先配置后重试」。

**Q：换模型后报错 404？**
模型 id 失效了（免费车道尤其常见）。`curl https://api.kilo.ai/api/gateway/models` 查可用模型，改 `model_list` 对应车道即可。

**Q：回复很慢？**
免费车道延迟 2～5 秒属正常；官方车道慢可切腾讯车道（`/set api 1`）。

---

## 更新日志

### 2026-10-23

**新增**

- 新增 Kilo 免密免费车道，无需 API Key 即可使用
- 模型名改为按车道从 `model_list` 解析，不再硬编码
- `get_config` 自动补齐旧版 `config.yml` 缺失项，老配置升级不报错
- 初始化流程新增「Kilo 免密免费车道」选项

**修复**

- `/balance` 读取不存在的配置项导致 KeyError
- `bot_manager` 中 `logging.ingo` 拼写错误，非管理员触发指令即崩溃
- `is_group_admin` 漏传群号导致 think/stream/trim 指令报 TypeError
- `/think default` 别名 `/td` 缺斜杠，指令永远匹配不上
- `/set api` 参数非数字时中断处理
- 未配置 API Key 时开放群任意消息导致任务崩溃
- `group_manager` 在 import 阶段强连 LLOneBot，未启动 QQ 端时无法导入
- 群默认配置为类级共享字典，会被实例改脏
- `requirements.txt` 为 UTF-16LE 编码，转为 UTF-8
- 其它：事件校验失败不再崩溃、`bot_id` 不再每条消息重复查询、后台任务不再被 GC 回收、初始化空输入不再崩溃

---

有问题请提issue或加入反馈群516723952

## 效果预览

![效果预览](https://github.com/user-attachments/assets/f6839acf-f99e-4308-900f-635c6cd27082)
