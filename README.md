<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>SkylineZ1 AI</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #121212;
      color: #fff;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      margin: 0;
    }
    .chat-container {
      width: 420px;
      height: 640px;
      background: #1e1e1e;
      border-radius: 20px;
      box-shadow: 0 0 15px rgba(0,0,0,0.5);
      display: flex;
      flex-direction: column;
      overflow: hidden;
    }
    .header {
      background: #007bff;
      padding: 10px;
      text-align: center;
      font-size: 1.1em;
      font-weight: bold;
    }
    .sub-header {
      font-size: 0.8em;
      color: #ddd;
    }
    .options {
      display: flex;
      flex-wrap: wrap;
      gap: 6px;
      padding: 10px;
      background: #181818;
      justify-content: center;
    }
    .option-btn {
      padding: 6px 12px;
      border-radius: 20px;
      background: #2a2a2a;
      color: #fff;
      cursor: pointer;
      font-size: 0.8em;
      border: 1px solid #444;
      transition: 0.2s;
    }
    .option-btn:hover { background: #007bff; }
    .chat-box {
      flex: 1;
      padding: 15px;
      overflow-y: auto;
      display: flex;
      flex-direction: column;
    }
    .welcome {
      text-align: center;
      color: #bbb;
      margin: 10px 0;
      font-style: italic;
    }
    .chat-message {
      margin: 8px 0;
      padding: 10px 14px;
      border-radius: 20px;
      max-width: 75%;
      word-wrap: break-word;
      line-height: 1.4;
      white-space: pre-wrap;
    }
    .chat-time {
      font-size: 0.7em;
      color: #aaa;
      margin-top: 4px;
      text-align: right;
    }
    .user {
      background: #007bff;
      align-self: flex-end;
      border-bottom-right-radius: 6px;
    }
    .bot {
      background: #2a2a2a;
      align-self: flex-start;
      border-bottom-left-radius: 6px;
    }
    .input-box {
      display: flex;
      border-top: 1px solid #444;
      background: #1e1e1e;
      padding: 8px;
    }
    input {
      flex: 1;
      padding: 12px;
      border: none;
      outline: none;
      border-radius: 20px;
      background: #2a2a2a;
      color: #fff;
      margin-right: 6px;
    }
    button {
      width: 60px;
      background: #007bff;
      border: none;
      border-radius: 20px;
      color: white;
      cursor: pointer;
      font-size: 18px;
      transition: 0.2s;
    }
    button:hover { background: #0056b3; }
    .typing {
      display: flex;
      gap: 5px;
      margin: 10px;
      align-items: center;
    }
    .dot {
      width: 8px;
      height: 8px;
      background: #007bff;
      border-radius: 50%;
      animation: bounce 1.3s infinite;
    }
    .dot:nth-child(2) { animation-delay: 0.2s; }
    .dot:nth-child(3) { animation-delay: 0.4s; }
    @keyframes bounce {
      0%, 80%, 100% { transform: scale(0.8); }
      40% { transform: scale(1.2); }
    }
    .footer {
      font-size: 0.75em;
      color: #999;
      text-align: center;
      padding: 6px;
      border-top: 1px solid #333;
      background: #181818;
    }
  </style>
</head>
<body>
  <div class="chat-container">
    <div class="header">
      SkylineZ1 Model <br>
      <span class="sub-header">By Amit Bhakundra</span>
    </div>

    <div class="options">
      <div class="option-btn">Search Everything</div>
      <div class="option-btn">Today New</div>
      <div class="option-btn">New Food</div>
      <div class="option-btn">Summary</div>
      <div class="option-btn">Photo Upload</div>
    </div>

    <div class="chat-box" id="chat-box">
      <div class="welcome">✨ Welcome Curious ✨</div>
    </div>

    <div class="input-box">
      <input type="text" id="user-input" placeholder="Type a message...">
      <button onclick="sendMessage()">➤</button>
    </div>

    <div class="footer">
      Amit Bhakundra Corp ©2025® | Powered by SkylineZ1
    </div>
  </div>

  <script>
    const API_KEY = "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";
    const API_URL = "https://openrouter.ai/api/v1/chat/completions";

    async function sendMessage() {
      const input = document.getElementById("user-input");
      const message = input.value.trim();
      if (!message) return;

      appendMessage("user", message);
      input.value = "";

      const botMsg = appendMessage("bot", "");
      const typingEl = showTyping();

      try {
        const response = await fetch(API_URL, {
          method: "POST",
          headers: {
            "Authorization": `Bearer ${API_KEY}`,
            "Content-Type": "application/json"
          },
          body: JSON.stringify({
            model: "deepseek/deepseek-chat",
            messages: [{ role: "user", content: message }],
            stream: true
          })
        });

        const reader = response.body.getReader();
        const decoder = new TextDecoder("utf-8");

        typingEl.remove();

        let buffer = "";
        while (true) {
          const { done, value } = await reader.read();
          if (done) break;

          buffer += decoder.decode(value, { stream: true });
          const lines = buffer.split("\n");

          for (let i = 0; i < lines.length - 1; i++) {
            const line = lines[i].trim();
            if (!line.startsWith("data: ")) continue;
            const data = line.replace("data: ", "").trim();
            if (data === "[DONE]") return;

            try {
              const json = JSON.parse(data);
              const delta = json.choices?.[0]?.delta?.content;
              if (delta) botMsg.firstChild.nodeValue += delta;
            } catch {}
          }
          buffer = lines[lines.length - 1];
        }
      } catch (error) {
        typingEl.remove();
        botMsg.innerText = "Error: " + error.message;
      }
    }

    function appendMessage(sender, text) {
      const chatBox = document.getElementById("chat-box");
      const msg = document.createElement("div");
      msg.className = "chat-message " + sender;
      msg.appendChild(document.createTextNode(text));

      const time = document.createElement("div");
      time.className = "chat-time";
      time.innerText = new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'});

      msg.appendChild(time);
      chatBox.appendChild(msg);
      chatBox.scrollTop = chatBox.scrollHeight;
      return msg;
    }

    function showTyping() {
      const chatBox = document.getElementById("chat-box");
      const typing = document.createElement("div");
      typing.className = "typing";
      typing.innerHTML = '<div class="dot"></div><div class="dot"></div><div class="dot"></div>';
      chatBox.appendChild(typing);
      chatBox.scrollTop = chatBox.scrollHeight;
      return typing;
    }
  </script>
</body>
</html>
