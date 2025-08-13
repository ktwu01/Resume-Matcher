# Server Deployment Guide

## Prerequisites
- Ubuntu/CentOS server with sudo access
- Domain name (optional but recommended)
- Python 3.12+, Node.js 18+, nginx

## Quick Deploy Steps

### 1. Clone & Setup
```bash
git clone <your-repo>
cd ai-resume-refiner
chmod +x setup.sh
./setup.sh  # Skip if Ollama fails - we use API
```

### 2. Environment Files
**Backend** (`apps/backend/.env`):
```bash
SYNC_DATABASE_URL=sqlite:///./app.db
ASYNC_DATABASE_URL=sqlite+aiosqlite:///./app.db
SESSION_SECRET_KEY=your-secure-random-key-here
LLM_PROVIDER=llama_index.llms.openai_like.OpenAILike
LLM_API_KEY=your-anthropic-api-key
LLM_BASE_URL=https://anyrouter.top/v1
LL_MODEL=claude-3-5-haiku-20241022
EMBEDDING_PROVIDER=openai
EMBEDDING_API_KEY=your-openai-key-for-embeddings
EMBEDDING_MODEL=text-embedding-3-small
```

**Frontend** (`apps/frontend/.env`):
```bash
NEXT_PUBLIC_API_URL=https://your-domain.com
```

### 3. Build & Start
```bash
# Build
npm run build

# Production start (with PM2)
npm install -g pm2
pm2 start "cd apps/backend && ~/.local/bin/uv run uvicorn app.main:app --host 0.0.0.0 --port 8000" --name backend
pm2 start "cd apps/frontend && npm start" --name frontend
pm2 save
pm2 startup
```

### 4. Nginx Reverse Proxy
```nginx
server {
    listen 80;
    server_name your-domain.com;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    location /api/ {
        proxy_pass http://localhost:8000/api/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
    
    location /ping {
        proxy_pass http://localhost:8000/ping;
    }
}
```

### 5. SSL (Optional)
```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

## Key Points
- No Ollama needed - uses API-based LLM
- Backend runs on :8000, Frontend on :3000
- Database is SQLite (consider PostgreSQL for production)
- Update `NEXT_PUBLIC_API_URL` to your domain
- Generate secure `SESSION_SECRET_KEY`
- Monitor with `pm2 logs`

## Quick Health Check
```bash
curl https://your-domain.com/ping
# Should return: {"message":"pong","database":"reachable"}
```
