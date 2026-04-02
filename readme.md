# realtime_chatapp

Lightweight full-stack realtime group chat demo (MERN + Socket.IO).

## Summary
Single-repo demo demonstrating a layered REST + WebSocket architecture:
- Frontend: React (Vite) — UI, REST client, socket.io-client
- Backend: Node.js + Express + Socket.IO — REST API, socket handler, JWT auth
- Database: MongoDB (Mongoose)
- Realtime: Socket.IO rooms for group chat, typing, presence (in-memory)

## Key features
- User register / login (JWT + bcrypt)
- Create / join / leave groups
- Persisted group messages + realtime broadcast
- Per-room online users list and typing indicator
- Simple, single-README repo for local development

## Repo layout
- mern-chat-backend/ — backend (server.js, socket.js, routes, models)
- mern-chat-frontend/ — frontend (Vite + React)
- README.md 

## Requirements
- Node.js >=16, npm
- MongoDB (local or hosted)

## Quick start (macOS)
Backend
1. cd mern-chat-backend
2. npm install
3. Create `.env`:
   MONGO_URI=mongodb://localhost:27017/chatdb
   JWT_SECRET=supersecret PORT=5000
4. Start: `npm start`

Frontend
1. cd mern-chat-frontend
2. npm install
3. Create Vite `.env`:VITE_API_URL=http://localhost:5000

4. Start dev server: `npm run dev`

## Important REST endpoints (summary)
- POST /api/users/register — { username, email, password } -> { user, token }
- POST /api/users/login — { email, password } -> { user, token }
- GET /api/groups
- POST /api/groups — { name, description }
- GET /api/groups/:id
- POST /api/groups/:id/join
- POST /api/groups/:id/leave
- POST /api/messages — { groupId, content } -> saved message
- GET /api/messages?groupId=xxx&page=1&limit=50

Protected routes require:
Authorization: Bearer <token>

## Recommended Socket events
- Handshake: client sends token via auth: { token }, server verifies JWT and attaches user
- client -> server:
  - "join room" { groupId }
  - "leave room" { groupId }
  - "new message" { message } (client can persist via REST then emit)
  - "typing" { groupId, isTyping }
- server -> clients:
  - "message received" { message }
  - "users in room" { groupId, users: [...] }
  - "user joined" { user, groupId }
  - "user left" { user, groupId }
  - "typing" { userId, groupId, isTyping }

## Data model summary
- User: { username, email, password(hashed), isAdmin, timestamps }
- Group: { name, description, admin(ref), members[ref], isPublic, timestamps }
- Message: { sender(ref), group(ref), content, attachments?, readBy[], timestamps }

Index messages by { group, createdAt } for paging.

## Security & scalability notes
- Do NOT trust client-sent user objects in socket handshake. Send token and verify on server.
- Presence is in-memory — not suitable for multi-instance deployment. Use Redis adapter or central store to scale.
- Validate group membership server-side for REST and socket actions.
- Sanitize and limit message content; add rate limiting in production.

## Development tips
- Use nodemon for backend dev restarts.
- Use browser React devtools and network tab to debug REST/socket traffic.
- Add Redis + socket.io-redis adapter to enable horizontal scaling and shared rooms/presence.
