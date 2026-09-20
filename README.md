const http = require('http');
const fs = require('fs');
const path = require('path');
const WebSocket = require('ws');

const PORT = 3000;

// HTTP Sunucusu (Sayfaları sunmak için)
const server = http.createServer((req, res) => {
    let filePath = path.join(__dirname, 'public', req.url === '/' ? 'index.html' : req.url);
    const extname = String(path.extname(filePath)).toLowerCase();
    const mimeTypes = {
        '.html': 'text/html',
        '.js': 'text/javascript',
        '.css': 'text/css',
    };

    fs.readFile(filePath, (error, content) => {
        if (error) {
            res.writeHead(404, { 'Content-Type': 'text/html' });
            res.end('<h1>404 Sayfa Bulunamadi</h1>', 'utf-8');
        } else {
            res.writeHead(200, { 'Content-Type': mimeTypes[extname] || 'application/octet-stream' });
            res.end(content, 'utf-8');
        }
    });
});

// WebSocket Sunucusu (Anlık mesajlaşma için)
const wss = new WebSocket.Server({ server });

wss.on('connection', (ws) => {
    console.log('Yeni bir cihaz/istemci bağlandı!');

    ws.on('message', (message) => {
        // Gelen mesafeyi bağlı olan HERkese (telefona ve bilgisayara) anında gönder
        wss.clients.forEach((client) => {
            if (client.readyState === WebSocket.OPEN) {
                client.send(message.toString());
            }
        });
    });

    ws.on('close', () => {
        console.log('Bir cihaz bağlantıyı kesti.');
    });
});

server.listen(PORT, '0.0.0.0', () => {
    console.log(`WebSocket Sohbet Sunucusu aktif: http://localhost:${PORT}`);
}); <!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Canli Chat Odasi</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body {
            font-family: Arial, sans-serif;
            background-color: #0b132b;
            color: #ffffff;
            display: flex;
            flex-direction: column;
            height: 100vh;
            height: 100dvh;
        }
        header {
            background-color: #1c2541;
            padding: 12px 16px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-bottom: 1px solid #3a506b;
        }
        #chat-messages {
            flex: 1;
            overflow-y: auto;
            padding: 12px;
            display: flex;
            flex-direction: column;
            gap: 8px;
        }
        .message {
            background-color: #1c2541;
            padding: 10px 14px;
            border-radius: 8px;
            max-width: 85%;
            word-wrap: break-word;
        }
        .message span {
            font-weight: bold;
            color: #4ea8de;
            display: block;
            margin-bottom: 2px;
            font-size: 0.85rem;
        }
        #chat-form {
            display: flex;
            padding: 10px;
            background-color: #1c2541;
            border-top: 1px solid #3a506b;
        }
        #message-input {
            flex: 1;
            padding: 12px;
            border: 1px solid #3a506b;
            border-radius: 6px;
            background-color: #0b132b;
            color: #fff;
            font-size: 1rem;
            outline: none;
        }
        #send-btn {
            background-color: #4ea8de;
            color: #0b132b;
            border: none;
            padding: 0 20px;
            margin-left: 8px;
            border-radius: 6px;
            font-weight: bold;
            cursor: pointer;
            font-size: 1rem;
        }
    </style>
</head>
<body>

    <header>
        <h2>Chat Odasi</h2>
        <span id="user-label">Baglaniyor...</span>
    </header>

    <div id="chat-messages">
        <div class="message"><span>Sistem</span>Sohbete hos geldin!</div>
    </div>

    <form id="chat-form">
        <input type="text" id="message-input" placeholder="Mesaj yaz..." autocomplete="off" required>
        <button type="submit" id="send-btn">Gonder</button>
    </form>

    <script>
        const username = prompt("Lutfen adinizi girin:", "Kullanici_" + Math.floor(Math.random() * 1000)) || "Misafir";
        document.getElementById('user-label').innerText = username;

        // WebSocket bağlantısı (Sayfanın açıldığı adrese otomatik bağlanır)
        const protocol = window.location.protocol === 'https:' ? 'wss://' : 'ws://';
        const ws = new WebSocket(protocol + window.location.host);

        const form = document.getElementById('chat-form');
        const input = document.getElementById('message-input');
        const messagesDiv = document.getElementById('chat-messages');

        ws.onmessage = (event) => {
            const data = JSON.parse(event.data);
            const div = document.createElement('div');
            div.className = 'message';
            div.innerHTML = `<span>${data.user}</span>${data.text}`;
            messagesDiv.appendChild(div);
            messagesDiv.scrollTop = messagesDiv.scrollHeight;
        };

        form.addEventListener('submit', (e) => {
            e.preventDefault();
            if (input.value.trim() && ws.readyState === WebSocket.OPEN) {
                const messageData = {
                    user: username,
                    text: input.value.trim()
                };
                ws.send(JSON.stringify(messageData));
                input.value = '';
            }
        });
    </script>
</body>
</html>
