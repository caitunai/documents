# Caitun Real-time Speech Recognition and Translation API Documentation

## Host Address

- **China Region:** `https://aibt.caitun.com`
- **Overseas Region:** `https://limevoice-api.dotcopilot.ai`

---

## Table of Contents

1. [Login API](#login-api)
2. [Language List API](#language-list-api)
3. [WebSocket Configuration API](#websocket-configuration-api)
4. [WebSocket Connection API](#websocket-connection-api)

---

## Login API

### API Information

- **Endpoint:** `{host}/api/ai-headset/login`
- **Method:** `POST`
- **Content-Type:** `application/json`

### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| platform | String | Yes | Organization/Platform ID |
| secret | String | Yes | Secret key |
| did | Boolean | Yes | Device identifier, fixed to true |
| is_third | Boolean | Yes | Whether it's a third-party application, fixed to true |
| code | String | Yes | User ID |
| email | String | No | User email |
| nickname | String | No | User nickname |

### Request Example

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

### Response Data Structure

#### Success Response

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

#### Response Field Description

| Field | Type | Description |
|-------|------|-------------|
| status | Int | Status code, 0 indicates success |
| message | String | Response message |
| data.token | String | Access token for subsequent request authentication |
| data.expiredAt | Long | Token expiration timestamp (seconds) |

#### Failure Response

```json
{
  "status": 1,
  "message": "Invalid signature"
}
```

---

## Language List API

### API Information

- **Endpoint:** `{host}/api/ai-headset/caitun/639-languages`
- **Method:** `GET`
- **Authentication:** Bearer Token

### Request Headers

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Authorization | String | Yes | Bearer {token} |

### Response Data Structure

#### Success Response

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

#### Response Field Description

| Field | Type | Description |
|-------|------|-------------|
| status | Int | Status code, 0 indicates success |
| message | String | Response message |
| data.list | Array | Language list |
| data.list[].Name | String | Language name (localized display) |
| data.list[].Tap | String | Tap prompt text |
| data.list[].EnterText | String | Input prompt text |
| data.list[].Listening | String | Listening prompt text |
| data.list[].Key | String | Language key (for display) |
| data.list[].Value | String | Language code (for API parameters) |
| data.list[].Voice | String | Voice synthesis voice name |

---

## WebSocket Configuration API

### API Information

- **Endpoint:** `{host}/api/ai-headset/caitun/websockets`
- **Method:** `POST`
- **Content-Type:** `application/json`
- **Authentication:** Bearer Token + Signature Verification

### Request Headers

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Authorization | String | Yes | Bearer {token} |
| Content-Type | String | Yes | application/json |

### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| organization_id | String | Yes | Organization ID |
| user_id | String | Yes | User ID |
| app_package_name | String | Yes | Application package name |
| timestamp | Long | Yes | Current timestamp (seconds) |
| signature | String | Yes | Signature value |

### Signature Algorithm

**Signature Steps:**
1. Construct canonical query string (parameters in alphabetical order):
   ```
   canonicalQueryString = "app_package_name={encodedAppPackageName}&organization_id={encodedOrganizationId}&timestamp={timestamp}&user_id={encodedUserId}"
   ```
2. Encrypt using HMAC-SHA256 with secret as the key
3. Convert the encryption result to a hexadecimal string

**Example:**
```
timestamp = 1704067200
app_package_name = "com.example.app"
organization_id = "org_001"
user_id = "user_123"

canonicalQueryString = "app_package_name=com.example.app&organization_id=org_001&timestamp=1704067200&user_id=user_123"
signature = HMAC-SHA256(canonicalQueryString, secret)
```

### Request Example

```json
{
  "organization_id": "org_001",
  "user_id": "user_123",
  "app_package_name": "com.example.app",
  "timestamp": 1704067200,
  "signature": "a1b2c3d4e5f6..."
}
```

### Response Data Structure

#### Success Response

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

#### Response Field Description

| Field | Type | Description |
|-------|------|-------------|
| status | Int | Status code, 0 indicates success |
| message | String | Response message |
| data.err_code | Int | Error code, 0 indicates success |
| data.err_msg | String | Error message |
| data.token | String | WebSocket connection token |
| data.expired_at | Long | Token expiration timestamp (seconds) |
| data.ai_dialogue | String | WebSocket connection base URL |
| data.agent_id | String | Agent ID for client identification |
| data.agent_key | String | Agent Key for encrypted communication |
| data.tts | String | TTS service URL |

---

## WebSocket Connection API

### API Information

- **Protocol:** WebSocket
- **Authentication:** Bearer Token

### Connection URL Construction

```
{ai_dialogue}?agent_id={agent_id}&agent_key={agent_key}&source_lang={sourceLang}&target_lang={targetLang}&disable_interrupt=1&step_mode={stepMode}
```

### URL Parameter Description

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| agent_id | String | Yes | Agent ID (obtained from configuration API) |
| agent_key | String | Yes | Agent Key (obtained from configuration API) |
| source_lang | String | Yes | Source language code, e.g., "en", "zh" |
| target_lang | String | Yes | Target language code, e.g., "zh", "en" |
| disable_interrupt | Int | Yes | Whether to disable interruption, 1 means disabled |
| step_mode | String | Yes | Step mode |

### step_mode Parameter Description

| Value | Description |
|-------|-------------|
| asr | Speech recognition only |
| llm | Speech recognition + LLM processing (translation results returned via chat messages in streaming) |
| tts | Full pipeline (speech recognition + LLM + TTS, translation results returned in tts messages) |

### Connection Headers

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| Authorization | String | Yes | Bearer {wsToken} |

### Connection URL Example

```
wss://autodlbjb1.xnestle.com:25864/v1/websocket/xiaozhi?agent_id=2&agent_key=ak_b3c39bbadf6859ecea0ec89f911111e9&source_lang=en&target_lang=zh&disable_interrupt=1&step_mode=tts
```

---

## WebSocket Communication Protocol

### Connection Flow

1. Establish WebSocket connection
2. Client sends `hello` message
3. Server returns `hello` message (containing session_id)
4. Client sends `listen` message (state=start)
5. Client starts sending audio data
6. Server returns recognition and translation results

---

### Client Messages

#### 1. Hello Message

Sent immediately after connection establishment to initialize the session.

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

**Field Description:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| type | String | Yes | Message type, fixed as "hello" |
| version | Int | Yes | Protocol version, fixed as 1 |
| transport | String | Yes | Transport protocol, fixed as "websocket" |
| device_id | String | Yes | Unique device identifier |
| device_name | String | Yes | Device name |
| device_mac | String | Yes | Device MAC address |
| audio_params | Object | Yes | Audio parameter configuration |
| audio_params.format | String | Yes | Audio format, fixed as "opus" |
| audio_params.sample_rate | Int | Yes | Sample rate, fixed as 16000 |
| audio_params.channels | Int | Yes | Number of channels, fixed as 1 |
| audio_params.frame_duration | Int | Yes | Frame duration (milliseconds), fixed as 60 |
| token | String | Yes | WebSocket Token |

#### 2. Listen State Message

Sent after receiving hello ack to notify the server to start listening.

```json
{
  "type": "listen",
  "mode": "manual",
  "state": "start"
}
```

**Field Description:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| type | String | Yes | Message type, fixed as "listen" |
| mode | String | Yes | Listen mode, fixed as "manual" |
| state | String | Yes | Listen state: start/stop |

#### 3. Audio Data (Binary Message)

- **Format:** Opus encoded audio data
- **Parameters:** 16kHz, mono, 60ms frame duration
- **Transmission:** Binary WebSocket message

#### 4. Language Switch Message

Switch source and target languages during recognition.

```json
{
  "session_id": "session_12345",
  "type": "lang",
  "source_lang_code": "en",
  "target_lang_code": "zh"
}
```

**Field Description:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| session_id | String | Yes | Session ID (obtained from hello ack) |
| type | String | Yes | Message type, fixed as "lang" |
| source_lang_code | String | Yes | New source language code |
| target_lang_code | String | Yes | New target language code |

---

### Server Messages

#### 1. Hello Ack Message

Server confirms successful connection.

```json
{
  "type": "hello",
  "session_id": "session_12345"
}
```

**Field Description:**

| Field | Type | Description |
|-------|------|-------------|
| type | String | Message type, fixed as "hello" |
| session_id | String | Session ID for subsequent operations like language switching |

#### 2. Speech Recognition Result (STT)

```json
{
  "type": "stt",
  "dialog_id": "dialog_001",
  "text": "你好"
}
```

**Field Description:**

| Field | Type | Description |
|-------|------|-------------|
| type | String | Message type, fixed as "stt" |
| dialog_id | String | Dialog ID for associating recognition and translation results |
| text | String | Recognized text content |

#### 3. Translation Result (TTS)

When `step_mode=tts`, translation results are returned through this message.

```json
{
  "type": "tts",
  "dialog_id": "dialog_001",
  "state": "sentence_start",
  "text": "Hello"
}
```

**Field Description:**

| Field | Type | Description |
|-------|------|-------------|
| type | String | Message type, fixed as "tts" |
| dialog_id | String | Dialog ID |
| state | String | TTS state: start/sentence_start/stop |
| text | String | Translation text (included in sentence_start) |

**State Values:**
- `start`: TTS task started, binary audio data will follow
- `sentence_start`: Sentence started, contains complete translation text
- `stop`: TTS task ended, audio stream transmission completed

#### 4. Chat Message

When `step_mode=llm`, translation results are returned through this message in streaming.

```json
{
  "type": "chat",
  "dialog_id": "dialog_001",
  "text": "How are you?",
  "finish": false
}
```

**Field Description:**

| Field | Type | Description |
|-------|------|-------------|
| type | String | Message type, fixed as "chat" |
| dialog_id | String | Dialog ID |
| text | String | Chat text content (streaming output, requires client-side concatenation) |
| finish | Boolean | Whether it's the final result |

**Note:** 
- When `finish=false`, the client needs to append text to previous text
- When `finish=true`, it indicates the current dialog's translation result is complete

#### 5. Token Consumption Notification

```json
{
  "type": "token",
  "token_type": "asr",
  "tokens": 100
}
```

**Field Description:**

| Field | Type | Description |
|-------|------|-------------|
| type | String | Message type, fixed as "token" |
| token_type | String | Token type: asr/llm/tts |
| tokens | Int | Number of tokens consumed |

#### 6. Audio Data (Binary Message)

TTS audio stream data, sent between `tts.start` and `tts.stop`.

- **Format:** Opus encoded audio data
- **Parameters:** 16kHz, mono, 60ms frame duration
- **Reception:** Binary WebSocket message

**Binary Data Parsing:**

Binary data contains header and payload, with different parsing methods based on the type field:

**Type = 0 (Simple Format):**
```
[0]: type (1 byte) = 0
[1]: reserved (1 byte)
[2-3]: size (2 bytes, big-endian)
[4-...]: opus payload
```

**Type = 3 (With dialog_id Format):**
```
[0]: type (1 byte) = 3
[1]: reserved (1 byte)
[2-3]: size (2 bytes, big-endian)
[4-11]: dialog_id (8 bytes, big-endian)
[12-...]: opus payload
```

**Parsing Steps:**
1. Read byte 0 to get type
2. Read bytes 2-3 to get size (big-endian)
3. If type = 3:
   - Read bytes 4-11 to get dialog_id (convert to BigInteger then to string)
   - Payload starts from byte 12 and ends at byte (4 + size)
4. If type = 0:
   - Payload starts from byte 4 with length of size

---

## Error Code Description

| Error Code | Description |
|------------|-------------|
| 0 | Success |
| 1 | Invalid timestamp |
| 2 | Invalid signature |
| 3 | Invalid organization |
| 4 | Invalid user |
| 5 | Invalid application |
| 6 | Token expired |
| 999 | Unknown error |

---
