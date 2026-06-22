# 彩豚实时语音识别翻译 接口文档

## Host 地址

- **中国区:** `https://aibt.caitun.com`
- **海外区:** `https://limevoice-api.dotcopilot.ai`

---

## 目录

1. [登录接口](#登录接口)
2. [语言列表接口](#语言列表接口)
3. [获取长链接配置接口](#获取长链接配置接口)
4. [长链接接口](#长链接接口)

---

## 登录接口

### 接口信息

- **接口地址:** `{host}/api/ai-headset/login`
- **请求方法:** `POST`
- **Content-Type:** `application/json`

### 请求参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| platform | String | 是 | 组织/平台 ID |
| secret | String | 是 | 密钥 |
| did | Boolean | 是 | 设备标识，固定为 true |
| is_third | Boolean | 是 | 是否为第三方应用，固定为 true |
| code | String | 是 | 用户 ID |
| email | String | 否 | 用户邮箱 |
| nickname | String | 否 | 用户昵称 |

### 请求示例

```json
{
  "platform": "org_001",
  "secret": "your_secret_key",
  "did": true,
  "is_third": true,
  "code": "user_123",
  "email": "",
  "nickname": ""
}
```

### 返回数据结构

#### 成功响应

```json
{
  "status": 0,
  "message": "success",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiredAt": 1704067200
  }
}
```

#### 返回字段说明

| 字段名 | 类型 | 说明 |
|--------|------|------|
| status | Int | 状态码，0 表示成功 |
| message | String | 响应消息 |
| data.token | String | 访问令牌，用于后续请求认证 |
| data.expiredAt | Long | Token 过期时间戳（秒级） |

#### 失败响应

```json
{
  "status": 1,
  "message": "Invalid signature"
}
```

---

## 语言列表接口

### 接口信息

- **接口地址:** `{host}/api/ai-headset/caitun/639-languages`
- **请求方法:** `GET`
- **认证方式:** Bearer Token

### 请求头

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| Authorization | String | 是 | Bearer {token} |

### 返回数据结构

#### 成功响应

```json
{
  "status": 0,
  "message": "ok",
  "data": {
    "list": [
      {
        "Name": "中文",
        "Tap": "点击开始说",
        "EnterText": "输入文本",
        "Listening": "输入中。。。",
        "Key": "中文",
        "Value": "zh",
        "Voice": "zh-CN-XiaoxiaoNeural"
      },
      {
        "Name": "英语",
        "Tap": "Tap to speak",
        "EnterText": "Enter text",
        "Listening": "Listening...",
        "Key": "English",
        "Value": "en",
        "Voice": "Vivian"
      }
    ]
  }
}
```

#### 返回字段说明

| 字段名 | 类型 | 说明 |
|--------|------|------|
| status | Int | 状态码，0 表示成功 |
| message | String | 响应消息 |
| data.list | Array | 语言列表 |
| data.list[].Name | String | 语言名称（本地化显示） |
| data.list[].Tap | String | 点击提示文本 |
| data.list[].EnterText | String | 输入提示文本 |
| data.list[].Listening | String | 监听中提示文本 |
| data.list[].Key | String | 语言键（用于显示） |
| data.list[].Value | String | 语言代码（用于接口传参） |
| data.list[].Voice | String | 语音合成声音名称 |

---

## 获取长链接配置接口

### 接口信息

- **接口地址:** `{host}/api/ai-headset/caitun/websockets`
- **请求方法:** `POST`
- **Content-Type:** `application/json`
- **认证方式:** Bearer Token + 签名验证

### 请求头

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| Authorization | String | 是 | Bearer {token} |
| Content-Type | String | 是 | application/json |

### 请求参数

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| organization_id | String | 是 | 组织 ID |
| user_id | String | 是 | 用户 ID |
| app_package_name | String | 是 | 应用包名 |
| timestamp | Long | 是 | 当前时间戳（秒级） |
| signature | String | 是 | 签名值 |

### 签名算法

**签名步骤:**
1. 拼接规范查询字符串（参数按字母顺序排列）:
   ```
   canonicalQueryString = "app_package_name={encodedAppPackageName}&organization_id={encodedOrganizationId}&timestamp={timestamp}&user_id={encodedUserId}"
   ```
2. 使用 secret 作为密钥进行 HMAC-SHA256 加密
3. 将加密结果转换为十六进制字符串

**示例:**
```
timestamp = 1704067200
app_package_name = "com.example.app"
organization_id = "org_001"
user_id = "user_123"

canonicalQueryString = "app_package_name=com.example.app&organization_id=org_001&timestamp=1704067200&user_id=user_123"
signature = HMAC-SHA256(canonicalQueryString, secret)
```

### 请求示例

```json
{
  "organization_id": "org_001",
  "user_id": "user_123",
  "app_package_name": "com.example.app",
  "timestamp": 1704067200,
  "signature": "a1b2c3d4e5f6..."
}
```

### 返回数据结构

#### 成功响应

```json
{
  "status": 0,
  "message": "success",
  "data": {
    "err_code": 0,
    "err_msg": "ok",
    "token": "eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9...",
    "expired_at": 1782099019,
    "ai_dialogue": "wss://autodlbjb1.xnestle.com:25864/v1/websocket/xiaozhi",
    "agent_id": "2",
    "agent_key": "ak_b3c39bbadf6859ecea0ec89f911111e9",
    "tts": "https://autodlbjb1.xnestle.com:25864/v1/tts"
  }
}
```

#### 返回字段说明

| 字段名 | 类型 | 说明 |
|--------|------|------|
| status | Int | 状态码，0 表示成功 |
| message | String | 响应消息 |
| data.err_code | Int | 错误码，0 表示成功 |
| data.err_msg | String | 错误消息 |
| data.token | String | WebSocket 连接令牌 |
| data.expired_at | Long | Token 过期时间戳（秒级） |
| data.ai_dialogue | String | WebSocket 连接基础 URL |
| data.agent_id | String | Agent ID，用于标识客户端 |
| data.agent_key | String | Agent Key，用于加密通信 |
| data.tts | String | TTS 服务 URL |

---

## 长链接接口

### 接口信息

- **协议:** WebSocket
- **认证方式:** Bearer Token

### 连接 URL 构建

```
{ai_dialogue}?agent_id={agent_id}&agent_key={agent_key}&source_lang={sourceLang}&target_lang={targetLang}&disable_interrupt=1&step_mode={stepMode}
```

### URL 参数说明

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| agent_id | String | 是 | Agent ID（从配置接口获取） |
| agent_key | String | 是 | Agent Key（从配置接口获取） |
| source_lang | String | 是 | 源语言代码，如 "en"、"zh" |
| target_lang | String | 是 | 目标语言代码，如 "zh"、"en" |
| disable_interrupt | Int | 是 | 是否禁用打断，1 表示禁用 |
| step_mode | String | 是 | 步骤模式 |

### step_mode 参数说明

| 值 | 说明 |
|----|------|
| asr | 仅语音识别 |
| llm | 语音识别 + 大模型处理（翻译结果通过 chat 消息流式返回） |
| tts | 完整流程（语音识别 + 大模型 + TTS，翻译结果在 tts 消息中返回） |

### 连接头

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| Authorization | String | 是 | Bearer {wsToken} |

### 连接 URL 示例

```
wss://autodlbjb1.xnestle.com:25864/v1/websocket/xiaozhi?agent_id=2&agent_key=ak_b3c39bbadf6859ecea0ec89f911111e9&source_lang=en&target_lang=zh&disable_interrupt=1&step_mode=tts
```

---

## WebSocket 通信协议

### 连接流程

1. 建立 WebSocket 连接
2. 客户端发送 `hello` 消息
3. 服务端返回 `hello` 消息（包含 session_id）
4. 客户端发送 `listen` 消息（state=start）
5. 客户端开始发送音频数据
6. 服务端返回识别和翻译结果

---

### 客户端发送消息

#### 1. Hello 消息

连接建立后立即发送，用于初始化会话。

```json
{
  "type": "hello",
  "version": 1,
  "transport": "websocket",
  "device_id": "android_abc12345",
  "device_name": "Android VoiceAgent",
  "device_mac": "00:11:22:33:44:55",
  "audio_params": {
    "format": "opus",
    "sample_rate": 16000,
    "channels": 1,
    "frame_duration": 60
  },
  "token": "eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9..."
}
```

**字段说明:**

| 字段名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| type | String | 是 | 消息类型，固定为 "hello" |
| version | Int | 是 | 协议版本，固定为 1 |
| transport | String | 是 | 传输协议，固定为 "websocket" |
| device_id | String | 是 | 设备唯一标识 |
| device_name | String | 是 | 设备名称 |
| device_mac | String | 是 | 设备 MAC 地址 |
| audio_params | Object | 是 | 音频参数配置 |
| audio_params.format | String | 是 | 音频格式，固定为 "opus" |
| audio_params.sample_rate | Int | 是 | 采样率，固定为 16000 |
| audio_params.channels | Int | 是 | 声道数，固定为 1 |
| audio_params.frame_duration | Int | 是 | 帧时长（毫秒），固定为 60 |
| token | String | 是 | WebSocket Token |

#### 2. Listen 状态消息

收到 hello ack 后发送，通知服务端开始监听。

```json
{
  "type": "listen",
  "mode": "manual",
  "state": "start"
}
```

**字段说明:**

| 字段名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| type | String | 是 | 消息类型，固定为 "listen" |
| mode | String | 是 | 监听模式，固定为 "manual" |
| state | String | 是 | 监听状态：start/stop |

#### 3. 音频数据（二进制消息）

- **格式:** Opus 编码音频数据
- **参数:** 16kHz, mono, 60ms frame duration
- **发送方式:** 二进制 WebSocket 消息

#### 4. 切换语言消息

在识别过程中切换源语言和目标语言。

```json
{
  "session_id": "session_12345",
  "type": "lang",
  "source_lang_code": "en",
  "target_lang_code": "zh"
}
```

**字段说明:**

| 字段名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| session_id | String | 是 | 会话 ID（从 hello ack 获取） |
| type | String | 是 | 消息类型，固定为 "lang" |
| source_lang_code | String | 是 | 新的源语言代码 |
| target_lang_code | String | 是 | 新的目标语言代码 |

---

### 服务端返回消息

#### 1. Hello Ack 消息

服务端确认连接成功。

```json
{
  "type": "hello",
  "session_id": "session_12345"
}
```

**字段说明:**

| 字段名 | 类型 | 说明 |
|--------|------|------|
| type | String | 消息类型，固定为 "hello" |
| session_id | String | 会话 ID，用于后续切换语言等操作 |

#### 2. 语音识别结果（STT）

```json
{
  "type": "stt",
  "dialog_id": "dialog_001",
  "text": "你好"
}
```

**字段说明:**

| 字段名 | 类型 | 说明 |
|--------|------|------|
| type | String | 消息类型，固定为 "stt" |
| dialog_id | String | 对话 ID，用于关联识别和翻译结果 |
| text | String | 识别文本内容 |

#### 3. 翻译结果（TTS）

当 `step_mode=tts` 时，翻译结果通过此消息返回。

```json
{
  "type": "tts",
  "dialog_id": "dialog_001",
  "state": "sentence_start",
  "text": "Hello"
}
```

**字段说明:**

| 字段名 | 类型 | 说明 |
|--------|------|------|
| type | String | 消息类型，固定为 "tts" |
| dialog_id | String | 对话 ID |
| state | String | TTS 状态：start/sentence_start/stop |
| text | String | 翻译文本（sentence_start 时包含） |

**state 取值说明:**
- `start`: TTS 任务开始，随后会发送二进制音频数据
- `sentence_start`: 句子开始，包含完整的翻译文本
- `stop`: TTS 任务结束，音频流传输完成

#### 4. 对话消息（Chat）

当 `step_mode=llm` 时，翻译结果通过此消息流式返回。

```json
{
  "type": "chat",
  "dialog_id": "dialog_001",
  "text": "How are you?",
  "finish": false
}
```

**字段说明:**

| 字段名 | 类型 | 说明 |
|--------|------|------|
| type | String | 消息类型，固定为 "chat" |
| dialog_id | String | 对话 ID |
| text | String | 对话文本内容（流式输出，需要客户端拼接） |
| finish | Boolean | 是否为最终结果 |

**注意:** 
- 当 `finish=false` 时，客户端需要将 text 追加到之前的文本后面
- 当 `finish=true` 时，表示当前对话的翻译结果已完整

#### 5. Token 消耗通知

```json
{
  "type": "token",
  "token_type": "asr",
  "tokens": 100
}
```

**字段说明:**

| 字段名 | 类型 | 说明 |
|--------|------|------|
| type | String | 消息类型，固定为 "token" |
| token_type | String | Token 类型：asr/llm/tts |
| tokens | Int | 消耗的 Token 数量 |

#### 6. 音频数据（二进制消息）

TTS 音频流数据，在 `tts.start` 和 `tts.stop` 之间发送。

- **格式:** Opus 编码音频数据
- **参数:** 16kHz, mono, 60ms frame duration
- **接收方式:** 二进制 WebSocket 消息

**二进制数据解析:**

二进制数据包含头部和载荷两部分，根据 type 字段不同有不同的解析方式：

**Type = 0（简单格式）:**
```
[0]: type (1 byte) = 0
[1]: reserved (1 byte)
[2-3]: size (2 bytes, big-endian)
[4-...]: opus payload
```

**Type = 3（带 dialog_id 格式）:**
```
[0]: type (1 byte) = 3
[1]: reserved (1 byte)
[2-3]: size (2 bytes, big-endian)
[4-11]: dialog_id (8 bytes, big-endian)
[12-...]: opus payload
```

**解析步骤:**
1. 读取第 0 字节获取 type
2. 读取第 2-3 字节获取 size（大端序）
3. 如果 type = 3：
   - 读取第 4-11 字节获取 dialog_id（转为 BigInteger 再转字符串）
   - 载荷从第 12 字节开始，到第 (4 + size) 字节结束
4. 如果 type = 0：
   - 载荷从第 4 字节开始，长度为 size

---

## 错误码说明

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| 1 | 无效时间戳 |
| 2 | 无效签名 |
| 3 | 无效组织 |
| 4 | 无效用户 |
| 5 | 无效应用 |
| 6 | Token 过期 |
| 999 | 未知错误 |

---
