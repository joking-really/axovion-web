# Axovion.io Deployment Guide

## Recommended Hosting Architecture

For a production-ready, cost-effective, and reliable deployment, I recommend **Option 1** (Platform-as-a-Service) for ease and speed, or **Option 3** (VPS) for maximum cost savings long-term.

---

## Option 1: Platform-as-a-Service (Recommended for Beginners)

**Best for:** Quick deployment, zero server management, automatic scaling

### Stack
- **Frontend:** Vercel (Free tier)
- **Backend:** Railway.app or Render.com (Free tier)
- **Database:** MongoDB Atlas (Free M0 cluster)
- **Custom Domain:** Cloudflare (Free DNS + CDN)

### Why This Stack?
- **Vercel:** Optimized for React, automatic deployments from Git, global CDN, free SSL
- **Railway/Render:** Simple container deployment, automatic HTTPS, environment variables management
- **MongoDB Atlas:** Managed database with backups, monitoring, and free tier
- **Estimated Cost:** $0-10/month (depending on traffic)

---

## Option 2: Cloud-Native (Best for Scale)

**Best for:** High traffic, microservices architecture, enterprise

### Stack
- **Frontend:** AWS S3 + CloudFront or Cloudflare Pages
- **Backend:** AWS ECS/Fargate or Google Cloud Run
- **Database:** MongoDB Atlas or AWS DocumentDB
- **Load Balancer:** AWS ALB or Cloudflare

### Why This Stack?
- Pay-as-you-go pricing
- Enterprise-grade reliability
- Advanced security features
- **Estimated Cost:** $20-100/month

---

## Option 3: VPS/Cloud Server (Most Cost-Effective)

**Best for:** Full control, predictable pricing, long-term savings

### Stack
- **Server:** Hetzner Cloud (€5.35/month) or DigitalOcean ($6/month)
- **Web Server:** Nginx (reverse proxy + static file serving)
- **Process Manager:** PM2 (for backend)
- **Database:** MongoDB (self-hosted on same server)
- **SSL:** Let's Encrypt (free)

### Why This Stack?
- **Cheapest option** for moderate traffic
- Full control over the environment
- Can host everything on one server
- **Estimated Cost:** $6-10/month

---

## Option 4: Static + Serverless (Ultra-Cheap)

**Best for:** Low traffic, hobby projects, proof of concepts

### Stack
- **Frontend:** Cloudflare Pages (Free, unlimited bandwidth)
- **Backend:** Cloudflare Workers or Vercel Serverless Functions
- **Database:** MongoDB Atlas (Free tier)

### Why This Stack?
- Nearly free for low traffic
- Edge computing for low latency
- **Estimated Cost:** $0-5/month

---

# Step-by-Step Deployment Guides

---

## Guide A: Deploy on Vercel + Railway (Recommended)

### Prerequisites
- GitHub account
- Domain name (optional but recommended)

### Step 1: Prepare Your Code

#### 1.1 Update Frontend API URL
In `frontend/.env.production`:
```env
REACT_APP_API_URL=https://your-backend-url.railway.app/api
REACT_APP_SITE_URL=https://your-frontend-url.vercel.app
```

#### 1.2 Update Backend CORS
In `backend/.env`:
```env
CORS_ORIGINS=https://your-frontend-url.vercel.app,https://www.yourdomain.com
```

#### 1.3 Create `vercel.json` in frontend/
```json
{
  "routes": [
    {
      "src": "/[^.]+",
      "dest": "/index.html",
      "status": 200
    }
  ]
}
```

#### 1.4 Create `railway.toml` in backend/
```toml
[build]
builder = "nixpacks"

[deploy]
startCommand = "uvicorn server:app --host 0.0.0.0 --port $PORT"
healthcheckPath = "/api/health"
healthcheckTimeout = 100
```

#### 1.5 Create `requirements.txt` (update if needed)
Ensure all dependencies are listed:
```txt
fastapi==0.115.12
uvicorn==0.34.0
pydantic==2.11.4
motor==3.7.0
pymongo==4.17.0
httpx==0.28.1
bcrypt==4.1.3
pyjwt==2.12.1
passlib==1.7.4
python-dotenv==1.2.2
email-validator==2.3.0
python-jose==3.5.0
python-multipart==0.0.28
requests==2.34.1
pandas==2.2.0
numpy==1.26.0
boto3==1.43.7
requests-oauthlib==2.0.0
cryptography==42.0.8
tzdata==2024.2
jq==1.11.0
```

### Step 2: Deploy Frontend to Vercel

1. Push your code to GitHub
2. Go to [vercel.com](https://vercel.com)
3. Click "Add New Project"
4. Import your GitHub repository
5. Set root directory to `frontend`
6. Add environment variables from `.env.production`
7. Click Deploy
8. Note down your Vercel URL (e.g., `https://axovion.vercel.app`)

### Step 3: Deploy Backend to Railway

1. Go to [railway.app](https://railway.app)
2. Click "New Project"
3. Select "Deploy from GitHub repo"
4. Import your repository
5. Set root directory to `backend`
6. Add environment variables in Railway Dashboard:
   - Click on your service → Variables → Add all from `.env`
7. Railway will auto-detect Python and deploy
8. Note down your Railway URL (e.g., `https://axovion-api.up.railway.app`)

### Step 4: Update Environment Variables

1. Go back to Vercel → Project Settings → Environment Variables
2. Update `REACT_APP_API_URL` to your Railway URL + `/api`
3. Redeploy frontend (Vercel will auto-redeploy)

### Step 5: Set Up MongoDB Atlas

1. Go to [mongodb.com/atlas](https://mongodb.com/atlas)
2. Create free cluster (M0)
3. Create a database user
4. Whitelist IPs (or allow from anywhere for Railway)
5. Get connection string:
   ```
   mongodb+srv://username:password@cluster.mongodb.net/axovion?retryWrites=true&w=majority
   ```
6. Add this to Railway environment variables as `MONGO_URL`

### Step 6: Connect Custom Domain (Optional)

#### On Vercel:
1. Project Settings → Domains
2. Add your domain
3. Follow DNS instructions

#### On Railway:
1. Service Settings → Domains
2. Add custom domain
3. Configure DNS CNAME record

#### On Cloudflare (Recommended for DNS):
1. Add your domain to Cloudflare
2. Update nameservers at your registrar
3. Add CNAME records:
   - `www` → `cname.vercel-dns.com`
   - `api` → your Railway domain
4. Enable "Full (Strict)" SSL mode

### Step 7: Verify Deployment

Test these endpoints:
- Frontend: `https://yourdomain.com`
- API: `https://api.yourdomain.com/api/health`
- Chat: Test the chatbot on your site

---

## Guide B: Deploy on Hetzner VPS (Most Cost-Effective)

### Prerequisites
- Hetzner Cloud account
- Domain name
- SSH key pair

### Step 1: Create Server

1. Go to [console.hetzner.cloud](https://console.hetzner.cloud)
2. Create new project
3. Add server:
   - Location: Closest to your users (e.g., US East, EU Central)
   - Image: Ubuntu 22.04
   - Type: CPX11 (2 vCPU, 4 GB RAM) - €5.35/month
   - Add your SSH key
4. Note the IP address

### Step 2: Initial Server Setup

SSH into your server:
```bash
ssh root@YOUR_SERVER_IP
```

Create a non-root user:
```bash
adduser axovion
usermod -aG sudo axovion
su - axovion
```

### Step 3: Install Dependencies

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install required packages
sudo apt install -y nginx git python3 python3-pip python3-venv nodejs npm mongodb

# Install PM2 globally
sudo npm install -g pm2

# Start MongoDB
sudo systemctl start mongod
sudo systemctl enable mongod

# Install certbot for SSL
sudo apt install -y certbot python3-certbot-nginx
```

### Step 4: Clone and Setup Application

```bash
cd /home/axovion
git clone YOUR_GITHUB_REPO_URL axovion
cd axovion
```

#### Setup Backend:
```bash
cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

Create `.env` file:
```bash
nano .env
```
Add your environment variables (same as local but with production values).

#### Setup Frontend:
```bash
cd ../frontend
npm install
npm run build
```

### Step 5: Configure Nginx

Create Nginx config:
```bash
sudo nano /etc/nginx/sites-available/axovion
```

Add this configuration:
```nginx
# Frontend - Static files
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    
    location / {
        root /home/axovion/axovion/frontend/build;
        try_files $uri /index.html;
        
        # Gzip compression
        gzip on;
        gzip_types text/plain text/css application/json application/javascript text/xml;
    }
    
    # API Proxy
    location /api/ {
        proxy_pass http://localhost:8000/api/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable the site:
```bash
sudo ln -s /etc/nginx/sites-available/axovion /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### Step 6: Setup PM2 for Backend

```bash
cd /home/axovion/axovion/backend
source venv/bin/activate
```

Create PM2 ecosystem file:
```bash
nano ecosystem.config.js
```

```javascript
module.exports = {
  apps: [{
    name: 'axovion-api',
    script: 'venv/bin/python',
    args: '-m uvicorn server:app --host 0.0.0.0 --port 8000',
    cwd: '/home/axovion/axovion/backend',
    instances: 1,
    autorestart: true,
    watch: false,
    max_memory_restart: '1G',
    env: {
      NODE_ENV: 'production'
    }
  }]
};
```

Start with PM2:
```bash
pm2 start ecosystem.config.js
pm2 save
pm2 startup systemd
```

### Step 7: Setup SSL with Let's Encrypt

```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

Follow prompts. Choose to redirect HTTP to HTTPS.

### Step 8: Configure Firewall

```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

### Step 9: Setup MongoDB (Production)

Create admin user:
```bash
mongosh
```

```javascript
use admin
db.createUser({
  user: "admin",
  pwd: "YOUR_STRONG_PASSWORD",
  roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
})
```

Enable authentication in `/etc/mongod.conf`:
```yaml
security:
  authorization: enabled
```

Restart MongoDB:
```bash
sudo systemctl restart mongod
```

Update backend `.env`:
```env
MONGO_URL=mongodb://admin:YOUR_PASSWORD@localhost:27017/axovion?authSource=admin
```

### Step 10: Setup Automated Deployments

Create deploy script:
```bash
nano /home/axovion/deploy.sh
```

```bash
#!/bin/bash
set -e

cd /home/axovion/axovion

# Pull latest code
git pull origin main

# Update backend
cd backend
source venv/bin/activate
pip install -r requirements.txt

# Update frontend
cd ../frontend
npm install
npm run build

# Restart backend
pm2 restart axovion-api

# Reload nginx
sudo nginx -s reload

echo "Deployment completed!"
```

Make executable:
```bash
chmod +x /home/axovion/deploy.sh
```

### Step 11: Setup GitHub Actions (CI/CD)

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to Production

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Deploy to server
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.SERVER_IP }}
        username: axovion
        key: ${{ secrets.SSH_PRIVATE_KEY }}
        script: |
          cd /home/axovion/axovion
          git pull origin main
          cd backend && source venv/bin/activate && pip install -r requirements.txt
          cd ../frontend && npm install && npm run build
          pm2 restart axovion-api
          sudo nginx -s reload
```

Add secrets in GitHub repository settings.

---

## Guide C: Deploy on Render (Alternative)

### Frontend (Static Site)
1. Go to [render.com](https://render.com)
2. New → Static Site
3. Connect GitHub repo
4. Set:
   - Root Directory: `frontend`
   - Build Command: `npm run build`
   - Publish Directory: `build`
5. Add environment variables
6. Deploy

### Backend (Web Service)
1. New → Web Service
2. Connect same repo
3. Set:
   - Root Directory: `backend`
   - Environment: Python 3
   - Build Command: `pip install -r requirements.txt`
   - Start Command: `uvicorn server:app --host 0.0.0.0 --port $PORT`
4. Add environment variables
5. Deploy

---

## Security Checklist

### Essential Security Measures

1. **Environment Variables**
   - Never commit `.env` files
   - Use strong, unique secrets in production
   - Rotate API keys regularly

2. **HTTPS Only**
   - Force HTTPS redirects
   - Use HSTS headers
   - Keep SSL certificates updated

3. **API Security**
   - Rate limiting on all endpoints
   - Input validation and sanitization
   - CORS properly configured
   - JWT secret should be 32+ random characters

4. **Database Security**
   - Enable MongoDB authentication
   - Use strong passwords
   - Limit network access (whitelist IPs)
   - Enable backups

5. **Server Security**
   - Keep OS and packages updated
   - Use firewall (UFW/iptables)
   - Disable root login via SSH
   - Use SSH keys only (no passwords)
   - Setup fail2ban

6. **Monitoring**
   - Setup log rotation
   - Monitor server resources
   - Setup alerts for downtime
   - Use PM2 for process monitoring

---

## Performance Optimization

### Frontend
1. **Enable Gzip/Brotli compression** (Nginx/Cloudflare)
2. **Use CDN** for static assets (Cloudflare, AWS CloudFront)
3. **Optimize images** (WebP format, lazy loading)
4. **Code splitting** (already configured in CRA)
5. **Cache headers** for static assets

### Backend
1. **Enable response compression**
2. **Database indexing** (ensure MongoDB indexes are created)
3. **Connection pooling** (Motor/PyMongo handles this)
4. **Caching layer** (Redis for session/API caching)
5. **Async operations** (already using async/await)

### Database
1. **Proper indexing** on frequently queried fields
2. **Regular backups** (MongoDB Atlas does this automatically)
3. **Connection limits** (monitor active connections)

---

## Monitoring & Maintenance

### Recommended Tools (Free Tiers)
1. **Uptime Monitoring:** UptimeRobot (free, 5-min intervals)
2. **Error Tracking:** Sentry (free tier)
3. **Analytics:** Google Analytics 4 (free)
4. **Server Monitoring:** Netdata or htop
5. **Logs:** PM2 logs or journalctl

### Regular Maintenance Tasks
- Weekly: Check server updates
- Monthly: Review logs for errors
- Monthly: Rotate API keys
- Quarterly: Security audit
- Quarterly: Performance review

---

## Cost Comparison (Monthly Estimates)

| Platform | Frontend | Backend | Database | Total |
|----------|----------|---------|----------|-------|
| **Vercel + Railway + Atlas** | Free | Free | Free | **$0** |
| **Vercel + Railway + Atlas** (Paid) | Free | $5 | Free | **$5** |
| **Hetzner VPS** | Included | Included | Included | **$6** |
| **DigitalOcean** | Included | Included | Included | **$6** |
| **AWS (minimal)** | $1 | $5 | $10 | **$16** |

*Note: Free tiers have limits. For production with traffic, expect $5-20/month.*

---

## Quick Start: My Recommendation

**For immediate deployment with minimal cost:**

1. **Use Option 1 (Vercel + Railway)**
   - Fastest setup (30 minutes)
   - Free tier handles moderate traffic
   - Automatic HTTPS and CDN
   - Easy to scale

2. **For cost savings long-term:**
   - Switch to Option 3 (Hetzner VPS) after validating your product
   - Single $6/month server handles everything
   - More predictable pricing

3. **Migration path:**
   - Start with PaaS (Option 1)
   - Move to VPS (Option 3) when ready
   - Both use the same codebase

---

## Troubleshooting

### Common Issues

**1. CORS Errors**
- Ensure `CORS_ORIGINS` includes your frontend URL exactly
- Check for `http` vs `https`
- Include `www` if using www subdomain

**2. MongoDB Connection Failed**
- Check IP whitelist in MongoDB Atlas
- Verify connection string format
- Ensure database user has correct permissions

**3. Environment Variables Not Loading**
- Restart the server after adding env vars
- Check variable names match exactly
- Verify no quotes around values (unless needed)

**4. Frontend Not Connecting to Backend**
- Check `REACT_APP_API_URL` is set correctly
- Verify backend URL is accessible
- Check browser console for errors

**5. Slow Response Times**
- Enable gzip compression
- Check server location vs user location
- Optimize database queries
- Consider adding Redis caching

---

## Support & Resources

- **Vercel Docs:** [vercel.com/docs](https://vercel.com/docs)
- **Railway Docs:** [docs.railway.app](https://docs.railway.app)
- **Render Docs:** [render.com/docs](https://render.com/docs)
- **MongoDB Atlas:** [docs.mongodb.com/atlas](https://docs.mongodb.com/atlas)
- **Nginx Docs:** [nginx.org/en/docs](https://nginx.org/en/docs)
- **PM2 Docs:** [pm2.keymetrics.io](https://pm2.keymetrics.io)

---

## Next Steps

1. Choose your deployment option
2. Follow the corresponding guide
3. Test thoroughly before sharing
4. Setup monitoring
5. Share your live Axovion.io app!

**Questions?** Check the troubleshooting section or review the specific platform documentation.
