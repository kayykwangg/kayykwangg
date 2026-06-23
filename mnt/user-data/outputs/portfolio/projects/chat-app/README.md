# Real-time Chat App

WebSocket-based multi-room chat built with Django Channels and React.

## Features

- **Multi-room chat** — create and join rooms by URL slug
- **User presence** — online/offline status, typing indicators
- **Message history** — last 50 messages loaded on join
- **Django Channels** with Redis as the channel layer
- **React frontend** with optimistic UI updates

## Stack

| Layer | Tech |
|---|---|
| Backend | Django 5 + Django Channels 4 |
| Channel layer | Redis 7 |
| Frontend | React + TypeScript |
| Auth | Django session auth |
| Styling | Tailwind CSS |

## Quick start

```bash
# Backend
cd backend
pip install -r requirements.txt
python manage.py migrate
python manage.py runserver

# In another terminal — Redis
docker run -p 6379:6379 redis:7-alpine

# Frontend
cd frontend
npm install && npm run dev
```

## How it works

Django Channels upgrades the HTTP connection to a WebSocket at `/ws/chat/<room_name>/`. Each connected client is added to a Channel Layer group named after the room. When a message arrives, the consumer broadcasts it to all group members via Redis pub/sub — this means it scales horizontally without sticky sessions.

```
Client A ──WebSocket──► Consumer ──► Redis Group ──► Consumer ──WebSocket──► Client B
                                                  └──WebSocket──► Client C
```

## WebSocket consumer (excerpt)

```python
class ChatConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        self.room_name = self.scope["url_route"]["kwargs"]["room_name"]
        self.room_group_name = f"chat_{self.room_name}"
        await self.channel_layer.group_add(self.room_group_name, self.channel_name)
        await self.accept()
        await self.send_history()

    async def receive(self, text_data):
        data = json.loads(text_data)
        await self.channel_layer.group_send(
            self.room_group_name,
            {"type": "chat.message", "message": data["message"], "user": self.scope["user"].username},
        )

    async def chat_message(self, event):
        await self.send(text_data=json.dumps({"message": event["message"], "user": event["user"]}))
```
