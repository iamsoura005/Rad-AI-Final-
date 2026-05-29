# Deployment Guide

## Deploying RadiologyAI to Vercel

This project has a **React frontend** (Vite) and a **FastAPI backend**. For Vercel deployment:

### Option 1: Frontend on Vercel + Backend Elsewhere (Recommended)

#### 1. Backend Deployment (Render, Railway, Heroku, or AWS)

Choose a platform that supports Python/FastAPI:

**Render.com** (Easy & Free tier):
1. Push your code to GitHub
2. Go to [render.com](https://render.com) → New → Web Service
3. Connect your GitHub repo
4. Set **Build Command**: `pip install -r backend/requirements.txt`
5. Set **Start Command**: `uvicorn backend.main:app --host 0.0.0.0 --port $PORT`
6. Add environment variable: `GROQ_API_KEY=your_actual_key`
7. Deploy
8. Copy the backend URL (e.g., `https://your-app.onrender.com`)

**Railway.app** (Easy & Free tier):
1. Connect GitHub repo
2. Create service → Create service from Git
3. Select backend directory
4. Add environment variable: `GROQ_API_KEY=your_actual_key`
5. Railway auto-detects Python + deploys
6. Copy the backend URL

#### 2. Frontend Deployment (Vercel)

1. Go to [vercel.com](https://vercel.com) → New Project
2. Import your GitHub repo
3. **Framework**: Vite (auto-detected)
4. **Root Directory**: Set to `frontend`
5. **Environment Variables** → Add:
   - `VITE_API_URL=https://your-backend-url.onrender.com` (replace with actual backend URL)
6. Deploy

#### 3. Update Frontend for Production

After deploying the backend, update `frontend/src/App.jsx` to use the environment variable:

```javascript
const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:8000';

async function uploadImage(file) {
  const formData = new FormData();
  formData.append('file', file);
  
  const response = await fetch(`${API_URL}/analyze`, {
    method: 'POST',
    body: formData
  });
  // ...
}
```

---

### Option 2: Monorepo Deployment (Advanced - Using Vercel with Backend as Serverless)

Requires refactoring backend to serverless functions (Python API Routes on Vercel). Not recommended for complex streaming apps like this.

---

## Environment Variables Needed

### Backend (Render/Railway/etc)
- `GROQ_API_KEY` - Your Groq API key from [console.groq.com](https://console.groq.com)

### Frontend (Vercel)
- `VITE_API_URL` - Your deployed backend URL

---

## Testing Before Deployment

1. **Local Backend** (on your machine):
   ```bash
   cd backend
   pip install -r requirements.txt
   $env:GROQ_API_KEY="your_key"  # PowerShell
   uvicorn main:app --reload --port 8000
   ```

2. **Local Frontend** (pointing to localhost backend):
   ```bash
   cd frontend
   npm install
   npm run dev
   ```

3. **Point frontend to deployed backend**:
   - Edit `.env.local` in frontend:
     ```
     VITE_API_URL=https://your-backend-url.onrender.com
     ```
   - Rebuild: `npm run build`

---

## Troubleshooting

**Frontend can't reach backend?**
- Check `VITE_API_URL` is correct in Vercel env variables
- Backend must have CORS enabled (already configured in `backend/main.py`)

**API key not working?**
- Verify `GROQ_API_KEY` is set in backend env variables
- Test key at [console.groq.com](https://console.groq.com)

**Build fails on Vercel?**
- Ensure `frontend` is the root directory or set it in project settings
- Check Node.js version (should be 18+)

---

## Quick Deployment Checklist

- [ ] Backend deployed to Render/Railway with `GROQ_API_KEY` env var
- [ ] Backend URL copied
- [ ] Frontend deployed to Vercel with `VITE_API_URL` env var set
- [ ] Frontend code uses `import.meta.env.VITE_API_URL` for API calls
- [ ] Tested upload and analysis flow end-to-end
