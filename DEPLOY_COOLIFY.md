# Deploying Car Agent on Coolify

This guide outlines how to deploy the Car Affordability Agent using Coolify and connect it to your frontend.

## 1. Prepare the Repository

The Car Agent is part of your mono-repo at `/Car-Agent/`. Coolify will build from this subdirectory.

**Prerequisites:**
- Gemini API Key (get at: https://aistudio.google.com/app/apikey)
- GitHub repository pushed to `main` branch
- Coolify instance running at `coolify.longtermtrends.net`

---

## 2. Coolify Configuration

### 2.1. Create New Application
1. **Login** to [coolify.longtermtrends.net](https://coolify.longtermtrends.net)
2. **Create New Resource:** Application → **Public Repository**
3. **Repository Settings:**
   - **URL:** `https://github.com/silvanfrank/longtermtrends2`
   - **Branch:** `main`
   - **Build Pack:** **Dockerfile**

### 2.2. Build Settings
Navigate to **General** tab:

**Base Directory:**
```
/Car-Agent
```

**Dockerfile Location:**
```
/Car-Agent/Dockerfile
```

**Port Exposes:**
```
8000
```

**Port Mappings:**
```
8010:8000
```
*(Note: Port 8010 is suggested to avoid conflict if running locally, but internally Coolify manages routing via Traefik so the exposed port is handled by the domain)*

### 2.3. Environment Variables
Go to **Environment Variables** tab and add:

| Key | Value | Description |
|-----|-------|-------------|
| `GEMINI_API_KEY` | `your_actual_api_key` | Google Gemini API key |
| `PYTHONUNBUFFERED` | `1` | Prevents Python output buffering |

### 2.4. Domain Configuration
**Domain Settings:**
- **Protocol:** `https`
- **Domain:** `car.longtermtrends.com`
- **Wildcard:** No

**DNS Setup (Required):**
Add an A record in your DNS provider:
```
car.longtermtrends.com → [Your Coolify Server IP]
```

### 2.5. Deploy
1. Click **Deploy**
2. Watch the build logs for errors
3. Wait for health check to pass (`/health` endpoint)

---

## 3. Verification

### 3.1. Health Check
```bash
curl https://car.longtermtrends.com/health
```

### 3.2. API Documentation
```
https://car.longtermtrends.com/docs
```

---

## 4. Frontend Integration

### 4.1. Update CORS Settings
Ensure `execution/api.py` includes `https://longtermtrends.net` in `ALLOWED_ORIGINS`.

### 4.2. Verify Chatbox Configuration
In `longtermtrends/community/templates/community/car-agent.html`, ensure the API URL matches your deployment:

```javascript
const apiUrl = "https://car.longtermtrends.com/chat";
```

---

## 5. Monitoring & Logs

### 5.1. Persistent Volume for Logs
To keep conversation logs across restarts:

1. Go to **Storages** tab in Coolify.
2. Add Volume:
   - **Name:** `car-agent-logs`
   - **Source:** `/var/lib/docker/volumes/car-agent-logs/_data`
   - **Destination:** `/app/logs`
3. **Redeploy**.

### 5.2. Download Logs
```bash
scp -i ~/.ssh/id_rsa root@46.224.23.170:/var/lib/docker/volumes/car-agent-logs/_data/transcripts.jsonl ~/Downloads/car_transcripts.jsonl
```

---

## 6. Deployment Status Template

```
🚗 Car Agent Deployment
━━━━━━━━━━━━━━━━━━━━━━━━
✅ Build: Success
✅ Health Check: Passing
✅ Domain: car.longtermtrends.com
✅ SSL: Active
✅ CORS: Configured
━━━━━━━━━━━━━━━━━━━━━━━━
Ready for Production: YES
```
