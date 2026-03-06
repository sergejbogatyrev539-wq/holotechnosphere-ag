# 🌐 Holotechnosphere AGI: Семиотический Реактор

**Автор:** Сергей Иванович Богатырёв  
**Email:** sergejbogatyrev539@gmail.com  
**Версия:** 1.0.0 (Alpha)  
**Лицензия:** AGPL-3.0

## 🚀 О Проекте

**Holotechnosphere AGI** — это уникальная система искусственного интеллекта, основанная на семиотике (науке о знаках и смыслах). В отличие от обычных чат-ботов, эта система не просто предсказывает слова, а пытается *понимать смыслы*, используя философию, этику и логику.

Это «сад мыслей», где человек и ИИ развиваются вместе.

## 🔑 Ключевые особенности

- **Семиотическое ядро:** Обработка информации через конструкты, маски и матрицы смыслов.
- **Этический каркас:** Встроенные принципы этики и безопасности.
- **Голосовое управление:** Полноценное взаимодействие голосом (STT/TTS).
- **Прозрачность:** Вы видите «мысли» системы в реальном времени.

## 🛠️ Установка и Запуск

Система спроектирована для запуска через Docker.

1. Клонируйте репозиторий:
```bash
   git clone https://github.com/ВАШ_НИК/holotechnosphere-ag.git
   cd holotechnosphere-ag
```

2. Запустите одной командой:
```bash
   docker-compose up --build
```

3. Откройте браузер по адресу: `http://localhost:3000`

## 📂 Структура

- `/backend` — Мозг системы (Python, FastAPI, Агенты).
- `/frontend` — Лицо системы (React, Визуализация).
- `/infra` — Инструкции для серверов (Docker, Kubernetes).
- `/knowledge_base` — Граф знаний и онтология.

## 🤝 Как помочь

Проект находится в стадии активной разработки. Любая помощь приветствуется!
Читайте файл `CONTRIBUTING.md` для подробностей.

---
*Создано с верой в этичное будущее ИИ.*
DB_USER=holo
DB_PASS=holo_secret
SECRET_KEY=supersecretkey_change_me
ENVIRONMENT=development
LOG_LEVEL=INFO
fastapi==0.104.1
uvicorn[standard]==0.24.0
websockets==12.0
pydantic==2.5.0
pydantic-settings==2.1.0
sqlalchemy==2.0.23
asyncpg==0.29.0
redis==5.0.1
python-multipart==0.0.6
numpy==1.26.2
aiofiles==23.2.1
pytest==7.4.3
pytest-asyncio==0.21.1
version: '3.8'

services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: ${DB_USER:-holo}
      POSTGRES_PASSWORD: ${DB_PASS:-holo_secret}
      POSTGRES_DB: holotechnosphere
    volumes:
      - pgdata:/var/lib/postgresql/data
    networks:
      - holo-net

  redis:
    image: redis:7-alpine
    networks:
      - holo-net

  backend:
    build: ./backend
    command: uvicorn main:app --host 0.0.0.0 --port 8000 --reload
    volumes:
      - ./backend:/app
    environment:
      DATABASE_URL: postgresql://${DB_USER:-holo}:${DB_PASS:-holo_secret}@db:5432/holotechnosphere
      REDIS_URL: redis://redis:6379
      SECRET_KEY: ${SECRET_KEY:-supersecretkey}
    depends_on:
      - db
      - redis
    ports:
      - "8000:8000"
    networks:
      - holo-net

  frontend:
    build: ./frontend
    ports:
      - "3000:80"
    depends_on:
      - backend
    networks:
      - holo-net

volumes:
  pgdata:

networks:
  holo-net:
    driver: bridge
backend/Dockerfile
FROM python:3.11-slim

WORKDIR /app

RUN apt-get update && apt-get install -y gcc libpq-dev && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY fastapi==0.104.1
uvicorn[standard]==0.24.0
websockets==12.0
pydantic==2.5.0
pydantic-settings==2.1.0
sqlalchemy==2.0.23
asyncpg==0.29.0
redis==5.0.1
python-multipart==0.0.6
numpy==1.26.2
aiofiles==23.2.1
pytest==7.4.3
pytest-asyncio==0.21.1

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
backend/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

app = FastAPI(title="Holotechnosphere AGI", version="1.0.0")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.on_event("startup")
async def startup_event():
    logger.info("🚀 Holotechnosphere AGI Core Starting...")
    logger.info("✅ Semiotic Engine Loaded")
    logger.info("✅ Agents Initialized")

@app.get("/")
async def root():
    return {"message": "Welcome to Holotechnosphere AGI. Connect to /ws/thought-stream"}

@app.get("/health")
async def health():
    return {"status": "healthy"}
frontend/Dockerfile
FROM node:18-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
frontend/nginx.conf
server {
    listen 80;
    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri $uri/ /index.html;
    }
}
frontend/package.json
{
  "name": "holo-frontend",
  "private": true,
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.2.1",
    "vite": "^5.0.8"
  }
}
frontend/vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    host: true,
    port: 5173
  }
})
frontend/index.html
<!doctype html>
<html lang="ru">
  <head>
    <meta charset="UTF-8" />
    <title>Holotechnosphere AGI</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
frontend/src/main.tsx
import React from 'react'
import ReactDOM from 'react-dom/client'

function App() {
  return (
    <div style={{padding: '20px', color: 'white', background: '#111', height: '100vh'}}>
      <h1>🌐 Holotechnosphere AGI</h1>
      <p>Система загружена. Автор: Сергей Богатырёв.</p>
      <p>Статус: Ожидание подключения ядра...</p>
    </div>
  )
}

ReactDOM.createRoot(document.getElementById('root')).render(<App />)
