# realtime_chatapp

 Full-stack realtime group chat(MERN + Socket.IO).

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
- readme.md

## Requirements
- Node.js >=16, npm
- MongoDB (local or hosted)

## Quick start (macOS)

**Backend**
1. `cd mern-chat-backend`
2. `npm install`
3. Create `.env`:
   ```
   MONGO_URL=mongodb://localhost:27017/chatdb
   JWT_SECRET=supersecret
   PORT=5000
   ```
4. Start: `npm start`

**Frontend**
1. `cd mern-chat-frontend`
2. `npm install`
3. Create `.env`:
   ```
   VITE_API_URL=http://localhost:5000
   ```
4. Start dev server: `npm run dev`

## REST endpoints

- `POST /api/users/register` — `{ username, email, password }` → `{ user, token }`
- `POST /api/users/login` — `{ email, password }` → `{ user, token }`
- `GET /api/groups` — list all groups
- `POST /api/groups` — `{ name, description }` → created group *(admin only)*
- `POST /api/groups/:groupId/join` — join a group
- `POST /api/groups/:groupId/leave` — leave a group
- `POST /api/messages` — `{ groupId, content }` → saved message
- `GET /api/messages/:groupId` — list all messages for a group

Protected routes require:
```
Authorization: Bearer <token>
```

## Socket events

**Handshake:** client passes user object via `auth: { user }`, server attaches it to the socket.

**client → server:**
- `"join room"` `groupId`
- `"leave room"` `groupId`
- `"new message"` `{ ...message, groupId }` (persist via REST first, then emit)
- `"typing"` `{ groupId, username }`
- `"stop typing"` `{ groupId }`

**server → clients:**
- `"message received"` `message`
- `"users in room"` `[...users]` — full list sent on join
- `"user left"` `userId`
- `"notification"` `{ type, message, user }` — e.g. `type: "USER_JOINED"`
- `"user typing"` `{ username }`
- `"user stop typing"` `{ username }`

## Data model summary
- User: `{ username, email, password(hashed), isAdmin, timestamps }`
- Group: `{ name, description, admin(ref), members[ref], isPublic, timestamps }`
- Message: `{ sender(ref), group(ref), content, readBy[], timestamps }`

## Security & scalability notes
- Do NOT trust client-sent user objects in socket handshake in production. Send a token and verify on the server instead.
- Presence is in-memory — not suitable for multi-instance deployment. Use Redis adapter or a central store to scale.
- Validate group membership server-side for both REST and socket actions.
- Sanitize and limit message content; add rate limiting in production.

## Development tips
- Use nodemon for backend dev restarts.
- Use browser React devtools and the network tab to debug REST/socket traffic.
- Add Redis + socket.io-redis adapter to enable horizontal scaling and shared rooms/presence.
