# 海狗公益(HiGO)AI辅助问诊API设计

## 设计与实现原则

1. 使用Python，以FastAPI为框架提供API服务。
2. 同LLM的交互，以LangGraph为框架。
3. API封装需要符合 OpenAI API 标准。
4. 使用LangSmith实现可观察性和运行评估，用于调试、测试和监控 AI 应用程序的性能。

## 新瑞鹏宠物医疗大模型Vet1+ API 接口说明

1. 使用OpenAI API标准

```.evn
OPENAI_BASE_URL=https://gateway.haoshouyi.com/vet1-scope/v1/
OPENAI_API_KEY=your_api_key
LLM_MODEL=ds-vet-answer-32B
```

## 汪喵灵灵宠物多模态模型vetmew API 接口调用说明

### 1. 调用入口

| 环境 | HTTPS 服务地址 |
|------|----------------|
| 正式 | <https://platformx.vetmew.com> |

---

### 2. 认证方式（HMAC-SHA256 签名）

#### 2.1 签名算法

```text
signature = base64(
  hmac_sha256(
    path + body + nonce + timestamp,
    api_secret
  )
)
```

#### 2.2 参数说明

| 字段       | 含义                                   |
|------------|----------------------------------------|
| api_secret | 你的 API 密钥                          |
| path       | 请求路径，例如 `/open/v1/chat`         |
| body       | 请求体 JSON 字符串（**顺序必须固定**） |
| nonce      | 8 位随机字符串                         |
| timestamp  | Unix 时间戳（秒级，5 分钟内有效）      |

---

#### 2.3 Python 签名示例

```python
import base64
import hashlib
import hmac

def generate_signature(path: str,
                       body: str,
                       nonce: str,
                       timestamp: str,
                       api_secret: str) -> str:
    """
    生成 HMAC-SHA256 + Base64 签名
    """
    raw = path + body + nonce + timestamp
    signature = base64.b64encode(
        hmac.new(
            api_secret.encode(),
            raw.encode(),
            hashlib.sha256
        ).digest()
    ).decode()
    return signature


# 示例调用
if __name__ == "__main__":
    path = "/open/v1/chat"
    body = '{"msg":"我的狗生病了","breed":1,"birth":"2024-07-01","gender":1,"nick_name":"大黄","fertility":1}'
    nonce = "zisbnzzr"
    timestamp = "1732072594"
    api_secret = "1pkni7hm42zl4tlhwn92mpfto9i957le"

    sign = generate_signature(path, body, nonce, timestamp, api_secret)
    print("Signature:", sign)   # MM8dJpELERXSZVWn+uWcK7/e+k1CQlae8qHttptq/x4=
```

---

### 3. 请求头

| 请求头       | 说明                                  |
|--------------|---------------------------------------|
| X-ApiKey     | API Key                               |
| X-Timestamp  | 同上时间戳                            |
| X-Nonce      | 同上随机串                            |
| X-Signature  | 上述生成的签名                        |
| Content-Type | application/json                      |

---

### 4. curl 示例

```bash
curl -X POST https://platformx-test.vetmew.com:21006/open/v1/chat \
  -H "Content-Type: application/json" \
  -H "X-ApiKey: vmcd9a2dfa0227fef6" \
  -H "X-Nonce: zisbnzzr" \
  -H "X-Timestamp: 1732072594" \
  -H "X-Signature: MM8dJpELERXSZVWn+uWcK7/e+k1CQlae8qHttptq/x4=" \
  -d '{"msg":"我的狗生病了","breed":1,"birth":"2024-07-01","gender":1,"nick_name":"大黄","fertility":1}'
```

---

### 5. HTTP 状态码

| 状态码 | 含义           |
|--------|----------------|
| 200    | 成功           |
| 400    | 请求参数错误   |
| 500    | 服务内部错误   |

---

### 6. 业务错误码

| 错误码 | 含义                                   |
|--------|----------------------------------------|
| 0      | 成功                                   |
| 6001   | 无效 API KEY                           |
| 6002   | 调用过于频繁                           |
| 6003   | 调用次数不足                           |
| 6004   | 签名参数无效                           |
| 6005   | 签名无效                               |
| 6006   | 业务参数无效                           |
| 6007   | 会话已结束，请新建会话                 |
| 6008   | 同一个会话无法同时接收多个请求         |
| 6009   | 输入内容含敏感词汇，请核对后再发送     |
| 6101   | 服务内部错误                           |

---

## 宠物信息获取接口说明

根据宠物ID，通过该接口获取宠物信息。

### 调用入口

| 环境 | HTTPS 服务地址 |
|------|----------------|
| 正式 | <https://platformx.vetmew.com> |

---

### 认证方式（HMAC-SHA256 签名）

#### 签名算法

```text
signature = base64(
  hmac_sha256(
    path + body + nonce + timestamp,
    api_secret
  )
)
```

#### 参数说明

| 字段       | 含义                                   |
|------------|----------------------------------------|
| api_secret | 你的 API 密钥                          |
| path       | 请求路径，例如 `/open/v1/pet/info`     |
| body       | 请求体 JSON 字符串（**顺序必须固定**） |
| nonce      | 8 位随机字符串                         |
| timestamp  | Unix 时间戳（秒级，5 分钟内有效）      |

参数举例：
api_secret: "1pkni7hm42zl4tlhwn92mpfto9i957le"
path: "/open/v1/pet/info"
body: {"pet_id": "123456"}
nonce: "zisbnzzr"
timestamp: "1732072594"

---

#### 返回结果

```json
{
  "code": 0,
  "data": {
    "pet_id": "123456",
    "pet_kind": 1,
    "breed": 1,
    "nick_name": "大黄",
    "birth": "2024-07-01",
    "gender": 1,
    "weight": 10,
    "fertility": 1,
    "medical_history": [
        {"illness_date":"...", "consultation_date":"...", "treatment":"...", "diagnosis":"...", "remark":"...", "summary", "..."},
        {...},
        {...},
    ]
  },
  "message": "成功"
}
```

#### Python 签名示例

```python
import base64
import hashlib
import hmac

def generate_signature(path: str,
                       body: str,
                       nonce: str,
                       timestamp: str,
                       api_secret: str) -> str:
    """
    生成 HMAC-SHA256 + Base64 签名
    """
    raw = path + body + nonce + timestamp
    signature = base64.b64encode(
        hmac.new(
            api_secret.encode(),
            raw.encode(),
            hashlib.sha256
        ).digest()
    ).decode()
    return signature


# 示例调用
if __name__ == "__main__":
    path = "/open/v1/pet/info"
    body = '{"pet_id": "123456"}'
    nonce = "zisbnzzr"
    timestamp = "1732072594"
    api_secret = "1pkni7hm42zl4tlhwn92mpfto9i957le"

    sign = generate_signature(path, body, nonce, timestamp, api_secret)
    print("Signature:", sign)   # MM8dJpELERXSZVWn+uWcK7/e+k1CQlae8qHttptq/x4=
```

---
