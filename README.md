# Deploying Next.js on VPS

This guide provides step-by-step instructions to deploy a **Next.js frontend** on a VPS using **Nginx as a reverse proxy** and **SSL with Certbot** for HTTPS.

## 🚀 Steps to Deploy a Next.js Frontend on VPS

### Step 1: Connect to Your VPS
```sh
ssh root@your-server-ip
```
If using a non-root user:
```sh
ssh user@your-server-ip
```

### Step 2: Install Dependencies
Update your system and install required dependencies:
```sh
sudo apt update && sudo apt upgrade -y
sudo apt install curl git nginx certbot python3-certbot-nginx -y
```

### Step 3: Install Node.js & Package Manager
If Node.js is not installed:
```sh
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
```
Verify installation:
```sh
node -v
npm -v
```
(Use **NVM** if you need a specific Node.js version.)

### Step 4: Clone Your Next.js Repository
```sh
git clone <your-nextjs-repo-url> nextjs-app
cd nextjs-app
```

### Step 5: Install Dependencies
```sh
npm install
```
Or if using pnpm:
```sh
pnpm install
```

### Step 6: Configure `.env` File
If your Next.js project uses environment variables:
```sh
nano .env
```
Add your variables, then save (`Ctrl + O`, Enter, `Ctrl + X`).

### Step 7: Build the Next.js Application
```sh
npm run build
```

### Step 8: Start the Next.js App
```sh
npm run start
```
(This starts the app, but it won’t run 24x7, so we use PM2.)

### Step 9: Keep Next.js Running with PM2
Install PM2:
```sh
npm install -g pm2
```
Run Next.js with PM2:
```sh
pm2 start npm --name "nextjs-app" -- start
```
Ensure it starts on reboot:
```sh
pm2 save
pm2 startup
```

### Step 10: Configure Nginx as a Reverse Proxy
Navigate to Nginx config directory:
```sh
cd /etc/nginx/sites-available
```
Create a new Nginx config file:
```sh
nano nextjs.conf
```
Add this configuration:
```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
Save (`Ctrl + O`, Enter, `Ctrl + X`).

Enable the config:
```sh
ln -s /etc/nginx/sites-available/nextjs.conf /etc/nginx/sites-enabled/
```
Test Nginx:
```sh
sudo nginx -t
```
Restart Nginx:
```sh
sudo systemctl restart nginx
```

### Step 11: Secure with SSL (HTTPS)
Run:
```sh
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```
Follow the instructions. Certbot will automatically configure SSL.

To auto-renew SSL:
```sh
sudo certbot renew --dry-run
```

### Step 12: Deployment Complete 🎉
Your Next.js frontend should now be live at **https://yourdomain.com**.

---
## ✅ Updating the Frontend
Whenever you push updates to GitHub:
```sh
cd nextjs-app
git pull
npm install
npm run build
pm2 restart nextjs-app
sudo systemctl restart nginx
```

Now your **Next.js frontend** is deployed and runs **24x7** on your VPS! 🚀

