# Deploy a Node.js App on Azure VM with SSL (Auto-Renew) & DuckDNS 🚀

This guide helps you deploy a **Node.js application** on an **Azure Virtual Machine (VM)** with **Let's Encrypt SSL (Auto-Renewal)** and **DuckDNS (Free Dynamic DNS)**.

## 🚀 Steps to Deploy a Node.js App on Azure VM

### ✅ Step 1: Create & Setup an Azure VM
1. Go to **Azure Portal** → [https://portal.azure.com/](https://portal.azure.com/)
2. **Create a New Virtual Machine**
   - **OS:** Ubuntu 22.04 LTS
   - **Size:** B1s (for small apps)
   - **Public IP:** Assign a static IP
   - **Inbound Ports:** Allow **SSH (22), HTTP (80), HTTPS (443)**
3. **Connect to your VM via SSH:**
   ```sh
   ssh username@your-public-ip
   ```

### ✅ Step 2: Install Required Packages
```sh
sudo apt update && sudo apt upgrade -y
sudo apt install -y nginx nodejs npm certbot python3-certbot-nginx
```

### ✅ Step 3: Setup Your Node.js App
1. **Create a new folder for the app:**
   ```sh
   mkdir -p ~/myapp && cd ~/myapp
   ```
2. **Clone or create a Node.js app:**
   ```sh
   git clone https://github.com/heroku/node-js-sample.git .
   npm install
   ```
3. **Run your app:**
   ```sh
   node index.js
   ```
   It should say:
   ```sh
   Node app is running at localhost:5000
   ```
4. **Install PM2 to keep it running in the background:**
   ```sh
   sudo npm install -g pm2
   pm2 start index.js --name myapp
   pm2 startup
   pm2 save
   ```

### ✅ Step 4: Set Up DuckDNS
1. Go to **DuckDNS** → [https://www.duckdns.org/](https://www.duckdns.org/)
2. **Add a subdomain** (e.g., `yourname.duckdns.org`)
3. **Get your DuckDNS Token**
4. **Set up a cron job to auto-update your public IP:**
   ```sh
   mkdir -p ~/duckdns && cd ~/duckdns
   nano duck.sh
   ```
   Add this inside:
   ```sh
   echo url="https://www.duckdns.org/update?domains=yourname&token=YOUR_DUCKDNS_TOKEN&ip=" | curl -k -o ~/duckdns/duck.log -K -
   ```
5. **Make it executable & set up cron job:**
   ```sh
   chmod +x duck.sh
   crontab -e
   ```
   Add this line at the bottom:
   ```sh
   */5 * * * * ~/duckdns/duck.sh >/dev/null 2>&1
   ```

### ✅ Step 5: Set Up Nginx as a Reverse Proxy
1. **Edit Nginx config:**
   ```sh
   sudo nano /etc/nginx/sites-available/default
   ```
   Replace content with:
   ```nginx
   server {
       listen 80;
       server_name yourname.duckdns.org;

       location / {
           proxy_pass http://localhost:5000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }
   }
   ```
2. **Restart Nginx:**
   ```sh
   sudo nginx -t
   sudo systemctl restart nginx
   ```

### ✅ Step 6: Open Firewall Ports
```sh
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

### ✅ Step 7: Install SSL Certificate (Let's Encrypt)
1. **Run Certbot to Generate SSL Certificate:**
   ```sh
   sudo certbot --nginx -d yourname.duckdns.org
   ```
   Select **option 2** to redirect all traffic to HTTPS.
2. **Enable Auto-Renewal:**
   ```sh
   sudo certbot renew --dry-run
   ```

### ✅ Step 8: Keep Everything Running on Reboot
```sh
pm2 startup
pm2 save
```

## 🎉 Done! Your Node.js App is Live with SSL on Azure!
✅ **App URL:** [https://yourname.duckdns.org/](https://yourname.duckdns.org/)

---

## 🚀 Common Issues & Fixes

### 🔥 Changes Not Reflecting in Browser?
1. **Restart PM2:**
   ```sh
   pm2 restart myapp
   ```
2. **Clear Nginx Cache (Optional):**
   ```sh
   sudo systemctl restart nginx
   ```
3. **Hard Refresh in Browser:**
   - **Windows/Linux:** Press `CTRL + SHIFT + R`
   - **Mac:** Press `CMD + SHIFT + R`
4. **Check If Changes Are Live:**
   ```sh
   curl http://localhost:5000
   ```
   If the updated content is shown here but not in the browser, it’s a caching issue.

✅ Now check [https://yourname.duckdns.org/](https://yourname.duckdns.org/) again! 🚀🔥

---

### 🚀 Now, Your Secure Node.js App is Running on Azure! 🎉

Feel free to share this guide with anyone who needs it! 🚀😎
