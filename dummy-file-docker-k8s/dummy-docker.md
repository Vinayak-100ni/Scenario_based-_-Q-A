## Stage 1 - Build dependencies
## Stage 2 - Runtime

```

FROM node:22-alpine AS builder

WORKDIR /app
COPY frontend/package*.json ./frontend/
RUN cd frontend && npm install
COPY frontend ./frontend
RUN cd frontend && npm run build

FROM node:22-alpine

WORKDIR /app
COPY backend/package*.json ./
RUN npm install --production
COPY backend .
COPY --from=builder /app/frontend/dist ./public

EXPOSE 5000

CMD ["node", "server.js"]
```
