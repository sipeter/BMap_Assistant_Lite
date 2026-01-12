# 🔌 BMap Voice Studio - TTS API 适配指南

如果你希望将自己的 TTS 服务（如 GPT-SoVITS、Bert-VITS 等）接入 **BMap Voice Studio**，只需确保你的服务端遵循以下 HTTP 接口规范即可。

## 1. 接口概述 (Protocol Overview)

*   **通讯协议**: HTTP / 1.1
*   **请求方法**: `POST`
*   **数据格式**: `multipart/form-data` (表单上传)
*   **响应格式**: 二进制音频流 (Binary Audio Stream)

## 2. 接口定义 (Endpoint Definition)

工具会自动在用户输入的 API 地址后拼接 `/synthesize`。

*   **路径**: `/synthesize`
*   **完整 URL 示例**: `http://127.0.0.1:8000/synthesize`

### 📥 请求参数 (Request)

客户端将以 `multipart/form-data` 形式发送以下字段：

| 字段名 | 类型 | 必填 | 描述 |
| :--- | :--- | :--- | :--- |
| **text** | `String` | 是 | 需要合成的文本内容 (例如："前方左转") |
| **prompt_audio** | `File` | 是 | **参考音频文件** (WAV/MP3)。即工具界面中选中的“参考音频”，用于克隆音色。 |
| **infer_mode** | `String` | 否 | 固定值为 `"普通推理"` (保留字段，服务端可忽略) |

### 📤 响应内容 (Response)

*   **HTTP 状态码**: `200 OK`
*   **Content-Type**: `audio/wav` 或 `audio/mpeg`
*   **Body**: **直接返回音频文件的二进制数据** (不要返回 JSON)。

---

## 3. 健康检查 (Health Check)

工具在启动时会通过 `GET` 请求检测服务是否在线。

*   **请求方法**: `GET`
*   **请求地址**: 用户在软件界面输入的地址 (例如 `http://127.0.0.1:8000`)
*   **要求**: 服务端只需返回 HTTP 200 状态码即可（内容不限）。

---

## 4. 服务端适配示例 (Python FastAPI)

如果你正在开发后端，可以使用以下 Python 代码快速适配此工具：

```python
from fastapi import FastAPI, Form, UploadFile, File
from fastapi.responses import Response
import uvicorn

app = FastAPI()

# 1. 健康检查接口
@app.get("/")
async def health_check():
    return {"status": "ok"}

# 2. 核心合成接口
@app.post("/synthesize")
async def synthesize(
    text: str = Form(...),                # 接收文本
    prompt_audio: UploadFile = File(...), # 接收参考音频文件
    infer_mode: str = Form(default="普通推理") # 接收保留字段
):
    print(f"收到合成请求: {text}")
    
    # 读取参考音频的二进制数据
    ref_audio_bytes = await prompt_audio.read()
    
    # TODO: 在这里调用你的 TTS 推理逻辑
    # 假设 generated_audio 是推理生成的 WAV/MP3 二进制数据
    # generated_audio = my_tts_model.inference(text, ref_audio_bytes)
    
    # 模拟返回一个空的音频数据 (实际开发请替换为真实音频)
    generated_audio = b'\x00\x00' * 100 

    # 直接返回二进制流
    return Response(content=generated_audio, media_type="audio/wav")

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

---

## 5. 常见问题 (FAQ)

**Q1: 为什么我的接口收到了请求，但软件提示 "TTS 失败"?**
*   检查 HTTP 状态码是否为 200。
*   检查返回的内容是否直接是**音频二进制流**。有些 API 习惯返回 JSON（如 `{"audio": "base64..."}`），这**不兼容**本工具，必须直接返回文件流。

**Q2: 为什么我看不到 `text` 参数？**
*   请确保你的服务端能解析 `multipart/form-data`。不要尝试以 JSON Body (`application/json`) 的形式读取数据。

**Q3: 参考音频是传路径还是传文件？**
*   本工具会读取本地文件，并以**文件流**的形式上传给服务器。服务端不需要访问本地文件系统，直接从内存读取上传的文件即可。