# ==============================================================================
#                      🚀 챗봇 실행을 위한 최종 통합 코드 (오타 수정) 🚀
# ==============================================================================

# ------------------------------------------------------------------------------
# 1. main.py 파이썬 서버 코드 파일 생성
# ------------------------------------------------------------------------------
%%writefile main.py

import os
import requests
from fastapi import FastAPI, HTTPException
from fastapi.responses import HTMLResponse
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
from typing import List, Dict, Any

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"], allow_credentials=True,
    allow_methods=["*"], allow_headers=["*"],
)

class ChatRequest(BaseModel): message: str
class ChatResponse(BaseModel): content: str; sources: List[str]; raw: Dict[str, Any]

# ===================================================================
#           🚨 중요: 아래 3개의 변수에 본인의 실제 값을 채워주세요!
# ===================================================================
# ==============================================================================
#                      🚀 챗봇 실행을 위한 최종 통합 코드 (오타 수정) 🚀
# ==============================================================================

# ------------------------------------------------------------------------------
# 1. main.py 파이썬 서버 코드 파일 생성
# ------------------------------------------------------------------------------
%%writefile main.py

import os
import requests
from fastapi import FastAPI, HTTPException
from fastapi.responses import HTMLResponse
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
from typing import List, Dict, Any

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"], allow_credentials=True,
    allow_methods=["*"], allow_headers=["*"],
)

class ChatRequest(BaseModel): message: str
class ChatResponse(BaseModel): content: str; sources: List[str]; raw: Dict[str, Any]

# ===================================================================
#           🚨 중요: 아래 3개의 변수에 본인의 실제 값을 채워주세요!
# ===================================================================
# ==============================================================================
#                      🚀 챗봇 실행을 위한 최종 통합 코드 (오타 수정) 🚀
# ==============================================================================

# ------------------------------------------------------------------------------
# 1. main.py 파이썬 서버 코드 파일 생성
# ------------------------------------------------------------------------------
%%writefile main.py

import os
import requests
from fastapi import FastAPI, HTTPException
from fastapi.responses import HTMLResponse
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
from typing import List, Dict, Any

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"], allow_credentials=True,
    allow_methods=["*"], allow_headers=["*"],
)

class ChatRequest(BaseModel): message: str
class ChatResponse(BaseModel): content: str; sources: List[str]; raw: Dict[str, Any]

# ===================================================================
#           🚨 중요: 아래 3개의 변수에 본인의 실제 값을 채워주세요!
# ===================================================================
# 1. Clova Studio API 키
CLOVA_API_KEY = "nv-2a8de24........."

# 2. API Gateway에서 발급받은 '호출 URL(Invoke URL)'
CLOVA_API_ENDPOINT = "https://kr-pub-gateway.rag.naverncp.com/api/v1/svc/서비스ID/conversation"

# 3. 나의 RAG 서비스 ID
RAG_SERVICE_ID = "68ad0c2........."
# ===================================================================

def ask_clova(user_message: str) -> ChatResponse:
    messages = [{"role": "user", "content": user_message}]
    request_body = {"messages": messages, "serviceId": RAG_SERVICE_ID}
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {CLOVA_API_KEY}"
    }
    try:
        response = requests.post(CLOVA_API_ENDPOINT, headers=headers, json=request_body, timeout=60)
        response.raise_for_status()
        data = response.json()
        content = "죄송합니다, 답변을 찾을 수 없습니다."
        sources = []
        if "data" in data and "results" in data["data"]:
            for result in data["data"]["results"]:
                if result.get("blockId") == "GENERATOR":
                    content = result.get("value", content)
                    break
            if "references" in data["data"]:
                for ref in data["data"]["references"]:
                    if ref.get("name") == "RETRIEVAL" and "data" in ref:
                        for item in ref["data"]:
                            sources.append(item.get("text", ""))
        return ChatResponse(content=content, sources=sources, raw=data)
    except requests.exceptions.RequestException as e:
        error_details = e.response.text if e.response else str(e)
        raise HTTPException(status_code=500, detail=f"CLOVA RAG API 오류: {error_details}")

@app.post("/api/chat", response_model=ChatResponse)
async def chat(req: ChatRequest):
    return ask_clova(req.message)

@app.get("/", response_class=HTMLResponse)
async def read_root():
    try:
        with open("index.html", "r", encoding="utf-8") as f:
            return HTMLResponse(content=f.read(), status_code=200)
    except FileNotFoundError:
        return HTMLResponse(content="<h1>index.html 파일을 찾을 수 없습니다.</h1>", status_code=404)

# ------------------------------------------------------------------------------
# 2. index.html 프론트엔드 코드 파일 생성
# ------------------------------------------------------------------------------
%%writefile index.html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8"><meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>부산시 여권발급 AI 상담</title>
  <style>
    body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; margin: 0; background: #f4f6f9; display: flex; justify-content: center; align-items: center; height: 100vh; }
    .chatbot-container { width: 400px; height: 90vh; max-height: 700px; background: white; border-radius: 16px; box-shadow: 0 10px 30px rgba(0,0,0,0.1); display: flex; flex-direction: column; }
    .chatbot-header { background: #007bff; color: white; padding: 16px; text-align: center; border-radius: 16px 16px 0 0; }
    .chat-messages { flex: 1; padding: 16px; overflow-y: auto; }
    .input-area { padding: 16px; border-top: 1px solid #e5e5e5; display: flex; gap: 8px; }
    #messageInput { flex: 1; padding: 10px; border: 1px solid #ccc; border-radius: 20px; font-size: 14px; }
    #sendButton { padding: 10px 16px; background: #007bff; color: white; border: none; border-radius: 20px; cursor: pointer; font-weight: bold; }
    .message { margin-bottom: 12px; display: flex; }
    .bot .message-bubble { background: #e9e9eb; color: #333; padding: 10px 14px; border-radius: 18px; max-width: 80%; word-wrap: break-word; }
    .user { justify-content: flex-end; }
    .user .message-bubble { background: #007bff; color: white; padding: 10px 14px; border-radius: 18px; max-width: 80%; word-wrap: break-word; }
  </style>
</head>
<body>
  <div class="chatbot-container">
    <div class="chatbot-header"><h3>부산시 여권발급 AI 상담</h3></div>
    <div class="chat-messages" id="chatMessages">
      <div class="message bot"><div class="message-bubble">안녕하세요! 여권 발급에 대해 무엇이 궁금하신가요?</div></div>
    </div>
    <div class="input-area">
      <input type="text" id="messageInput" placeholder="질문을 입력하세요..." onkeypress="if(event.key === 'Enter') sendMessage()">
      <button id="sendButton" onclick="sendMessage()">전송</button>
    </div>
  </div>
  <script>
    const chatMessages = document.getElementById('chatMessages');
    const messageInput = document.getElementById('messageInput');
    async function sendMessage() {
      const message = messageInput.value.trim();
      if (!message) return;
      addMessage(message, true);
      messageInput.value = '';
      try {
        const response = await fetch('/api/chat', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ message: message })
        });
        if (!response.ok) {
          const err = await response.json();
          throw new Error(err.detail || '서버 오류');
        }
        const data = await response.json();
        addMessage(data.content, false);
      } catch (error) {
        addMessage(`오류: ${error.message}`, false);
      }
    }
    function addMessage(text, isUser) {
      const msgDiv = document.createElement('div');
      msgDiv.className = isUser ? 'message user' : 'message bot';
      const bubbleDiv = document.createElement('div');
      bubbleDiv.className = 'message-bubble';
      bubbleDiv.textContent = text;
      msgDiv.appendChild(bubbleDiv);
      chatMessages.appendChild(msgDiv);
      chatMessages.scrollTop = chatMessages.scrollHeight;
    }
  </script>
</body>
</html>

# ------------------------------------------------------------------------------
# 3. 필요한 라이브러리 설치
# ------------------------------------------------------------------------------
print("⏳ 필요한 라이브러리를 설치합니다...")
!pip install -q "fastapi[all]" uvicorn requests pydantic

# ------------------------------------------------------------------------------
# 4. 백그라운드에서 서버 실행 및 URL 출력 (오타 수정됨)
# ------------------------------------------------------------------------------
import uvicorn
import threading
import time
from google.colab.output import eval_js

print("\n🚀 서버를 실행합니다...")

def run_app():
  # "main:py" -> "main:app" 으로 오타 수정
  uvicorn.run("main:app", host="0.0.0.0", port=8000)

thread = threading.Thread(target=run_app, daemon=True)
thread.start()
time.sleep(5)

print("\n\n🎉 서버가 준비되었습니다! 아래 URL로 접속하세요:")
print(eval_js("google.colab.kernel.proxyPort(8000)"))
# ===================================================================

def ask_clova(user_message: str) -> ChatResponse:
    messages = [{"role": "user", "content": user_message}]
    request_body = {"messages": messages, "serviceId": RAG_SERVICE_ID}
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {CLOVA_API_KEY}"
    }
    try:
        response = requests.post(CLOVA_API_ENDPOINT, headers=headers, json=request_body, timeout=60)
        response.raise_for_status()
        data = response.json()
        content = "죄송합니다, 답변을 찾을 수 없습니다."
        sources = []
        if "data" in data and "results" in data["data"]:
            for result in data["data"]["results"]:
                if result.get("blockId") == "GENERATOR":
                    content = result.get("value", content)
                    break
            if "references" in data["data"]:
                for ref in data["data"]["references"]:
                    if ref.get("name") == "RETRIEVAL" and "data" in ref:
                        for item in ref["data"]:
                            sources.append(item.get("text", ""))
        return ChatResponse(content=content, sources=sources, raw=data)
    except requests.exceptions.RequestException as e:
        error_details = e.response.text if e.response else str(e)
        raise HTTPException(status_code=500, detail=f"CLOVA RAG API 오류: {error_details}")

@app.post("/api/chat", response_model=ChatResponse)
async def chat(req: ChatRequest):
    return ask_clova(req.message)

@app.get("/", response_class=HTMLResponse)
async def read_root():
    try:
        with open("index.html", "r", encoding="utf-8") as f:
            return HTMLResponse(content=f.read(), status_code=200)
    except FileNotFoundError:
        return HTMLResponse(content="<h1>index.html 파일을 찾을 수 없습니다.</h1>", status_code=404)

# ------------------------------------------------------------------------------
# 2. index.html 프론트엔드 코드 파일 생성
# ------------------------------------------------------------------------------
%%writefile index.html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8"><meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>부산시 여권발급 AI 상담</title>
  <style>
    body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; margin: 0; background: #f4f6f9; display: flex; justify-content: center; align-items: center; height: 100vh; }
    .chatbot-container { width: 400px; height: 90vh; max-height: 700px; background: white; border-radius: 16px; box-shadow: 0 10px 30px rgba(0,0,0,0.1); display: flex; flex-direction: column; }
    .chatbot-header { background: #007bff; color: white; padding: 16px; text-align: center; border-radius: 16px 16px 0 0; }
    .chat-messages { flex: 1; padding: 16px; overflow-y: auto; }
    .input-area { padding: 16px; border-top: 1px solid #e5e5e5; display: flex; gap: 8px; }
    #messageInput { flex: 1; padding: 10px; border: 1px solid #ccc; border-radius: 20px; font-size: 14px; }
    #sendButton { padding: 10px 16px; background: #007bff; color: white; border: none; border-radius: 20px; cursor: pointer; font-weight: bold; }
    .message { margin-bottom: 12px; display: flex; }
    .bot .message-bubble { background: #e9e9eb; color: #333; padding: 10px 14px; border-radius: 18px; max-width: 80%; word-wrap: break-word; }
    .user { justify-content: flex-end; }
    .user .message-bubble { background: #007bff; color: white; padding: 10px 14px; border-radius: 18px; max-width: 80%; word-wrap: break-word; }
  </style>
</head>
<body>
  <div class="chatbot-container">
    <div class="chatbot-header"><h3>부산시 여권발급 AI 상담</h3></div>
    <div class="chat-messages" id="chatMessages">
      <div class="message bot"><div class="message-bubble">안녕하세요! 여권 발급에 대해 무엇이 궁금하신가요?</div></div>
    </div>
    <div class="input-area">
      <input type="text" id="messageInput" placeholder="질문을 입력하세요..." onkeypress="if(event.key === 'Enter') sendMessage()">
      <button id="sendButton" onclick="sendMessage()">전송</button>
    </div>
  </div>
  <script>
    const chatMessages = document.getElementById('chatMessages');
    const messageInput = document.getElementById('messageInput');
    async function sendMessage() {
      const message = messageInput.value.trim();
      if (!message) return;
      addMessage(message, true);
      messageInput.value = '';
      try {
        const response = await fetch('/api/chat', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ message: message })
        });
        if (!response.ok) {
          const err = await response.json();
          throw new Error(err.detail || '서버 오류');
        }
        const data = await response.json();
        addMessage(data.content, false);
      } catch (error) {
        addMessage(`오류: ${error.message}`, false);
      }
    }
    function addMessage(text, isUser) {
      const msgDiv = document.createElement('div');
      msgDiv.className = isUser ? 'message user' : 'message bot';
      const bubbleDiv = document.createElement('div');
      bubbleDiv.className = 'message-bubble';
      bubbleDiv.textContent = text;
      msgDiv.appendChild(bubbleDiv);
      chatMessages.appendChild(msgDiv);
      chatMessages.scrollTop = chatMessages.scrollHeight;
    }
  </script>
</body>
</html>

# ------------------------------------------------------------------------------
# 3. 필요한 라이브러리 설치
# ------------------------------------------------------------------------------
print("⏳ 필요한 라이브러리를 설치합니다...")
!pip install -q "fastapi[all]" uvicorn requests pydantic

# ------------------------------------------------------------------------------
# 4. 백그라운드에서 서버 실행 및 URL 출력 (오타 수정됨)
# ------------------------------------------------------------------------------
import uvicorn
import threading
import time
from google.colab.output import eval_js

print("\n🚀 서버를 실행합니다...")

def run_app():
  # "main:py" -> "main:app" 으로 오타 수정
  uvicorn.run("main:app", host="0.0.0.0", port=8000)

thread = threading.Thread(target=run_app, daemon=True)
thread.start()
time.sleep(5)

print("\n\n🎉 서버가 준비되었습니다! 아래 URL로 접속하세요:")
print(eval_js("google.colab.kernel.proxyPort(8000)"))
# ===================================================================

def ask_clova(user_message: str) -> ChatResponse:
    messages = [{"role": "user", "content": user_message}]
    request_body = {"messages": messages, "serviceId": RAG_SERVICE_ID}
    headers = {
        "Content-Type": "application/json",
        "Authorization": f"Bearer {CLOVA_API_KEY}"
    }
    try:
        response = requests.post(CLOVA_API_ENDPOINT, headers=headers, json=request_body, timeout=60)
        response.raise_for_status()
        data = response.json()
        content = "죄송합니다, 답변을 찾을 수 없습니다."
        sources = []
        if "data" in data and "results" in data["data"]:
            for result in data["data"]["results"]:
                if result.get("blockId") == "GENERATOR":
                    content = result.get("value", content)
                    break
            if "references" in data["data"]:
                for ref in data["data"]["references"]:
                    if ref.get("name") == "RETRIEVAL" and "data" in ref:
                        for item in ref["data"]:
                            sources.append(item.get("text", ""))
        return ChatResponse(content=content, sources=sources, raw=data)
    except requests.exceptions.RequestException as e:
        error_details = e.response.text if e.response else str(e)
        raise HTTPException(status_code=500, detail=f"CLOVA RAG API 오류: {error_details}")

@app.post("/api/chat", response_model=ChatResponse)
async def chat(req: ChatRequest):
    return ask_clova(req.message)

@app.get("/", response_class=HTMLResponse)
async def read_root():
    try:
        with open("index.html", "r", encoding="utf-8") as f:
            return HTMLResponse(content=f.read(), status_code=200)
    except FileNotFoundError:
        return HTMLResponse(content="<h1>index.html 파일을 찾을 수 없습니다.</h1>", status_code=404)

# ------------------------------------------------------------------------------
# 2. index.html 프론트엔드 코드 파일 생성
# ------------------------------------------------------------------------------
%%writefile index.html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8"><meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>부산시 여권발급 AI 상담</title>
  <style>
    body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; margin: 0; background: #f4f6f9; display: flex; justify-content: center; align-items: center; height: 100vh; }
    .chatbot-container { width: 400px; height: 90vh; max-height: 700px; background: white; border-radius: 16px; box-shadow: 0 10px 30px rgba(0,0,0,0.1); display: flex; flex-direction: column; }
    .chatbot-header { background: #007bff; color: white; padding: 16px; text-align: center; border-radius: 16px 16px 0 0; }
    .chat-messages { flex: 1; padding: 16px; overflow-y: auto; }
    .input-area { padding: 16px; border-top: 1px solid #e5e5e5; display: flex; gap: 8px; }
    #messageInput { flex: 1; padding: 10px; border: 1px solid #ccc; border-radius: 20px; font-size: 14px; }
    #sendButton { padding: 10px 16px; background: #007bff; color: white; border: none; border-radius: 20px; cursor: pointer; font-weight: bold; }
    .message { margin-bottom: 12px; display: flex; }
    .bot .message-bubble { background: #e9e9eb; color: #333; padding: 10px 14px; border-radius: 18px; max-width: 80%; word-wrap: break-word; }
    .user { justify-content: flex-end; }
    .user .message-bubble { background: #007bff; color: white; padding: 10px 14px; border-radius: 18px; max-width: 80%; word-wrap: break-word; }
  </style>
</head>
<body>
  <div class="chatbot-container">
    <div class="chatbot-header"><h3>부산시 여권발급 AI 상담</h3></div>
    <div class="chat-messages" id="chatMessages">
      <div class="message bot"><div class="message-bubble">안녕하세요! 여권 발급에 대해 무엇이 궁금하신가요?</div></div>
    </div>
    <div class="input-area">
      <input type="text" id="messageInput" placeholder="질문을 입력하세요..." onkeypress="if(event.key === 'Enter') sendMessage()">
      <button id="sendButton" onclick="sendMessage()">전송</button>
    </div>
  </div>
  <script>
    const chatMessages = document.getElementById('chatMessages');
    const messageInput = document.getElementById('messageInput');
    async function sendMessage() {
      const message = messageInput.value.trim();
      if (!message) return;
      addMessage(message, true);
      messageInput.value = '';
      try {
        const response = await fetch('/api/chat', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ message: message })
        });
        if (!response.ok) {
          const err = await response.json();
          throw new Error(err.detail || '서버 오류');
        }
        const data = await response.json();
        addMessage(data.content, false);
      } catch (error) {
        addMessage(`오류: ${error.message}`, false);
      }
    }
    function addMessage(text, isUser) {
      const msgDiv = document.createElement('div');
      msgDiv.className = isUser ? 'message user' : 'message bot';
      const bubbleDiv = document.createElement('div');
      bubbleDiv.className = 'message-bubble';
      bubbleDiv.textContent = text;
      msgDiv.appendChild(bubbleDiv);
      chatMessages.appendChild(msgDiv);
      chatMessages.scrollTop = chatMessages.scrollHeight;
    }
  </script>
</body>
</html>

# ------------------------------------------------------------------------------
# 3. 필요한 라이브러리 설치
# ------------------------------------------------------------------------------
print("⏳ 필요한 라이브러리를 설치합니다...")
!pip install -q "fastapi[all]" uvicorn requests pydantic

# ------------------------------------------------------------------------------
# 4. 백그라운드에서 서버 실행 및 URL 출력 (오타 수정됨)
# ------------------------------------------------------------------------------
import uvicorn
import threading
import time
from google.colab.output import eval_js

print("\n🚀 서버를 실행합니다...")

def run_app():
  # "main:py" -> "main:app" 으로 오타 수정
  uvicorn.run("main:app", host="0.0.0.0", port=8000)

thread = threading.Thread(target=run_app, daemon=True)
thread.start()
time.sleep(5)

print("\n\n🎉 서버가 준비되었습니다! 아래 URL로 접속하세요:")
print(eval_js("google.colab.kernel.proxyPort(8000)"))
