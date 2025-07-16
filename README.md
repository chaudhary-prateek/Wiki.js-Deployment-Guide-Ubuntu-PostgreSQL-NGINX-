# Wiki.js-Deployment-Guide-Ubuntu-PostgreSQL-NGINX-

````
## ✅ Prerequisites

- Ubuntu 22.04+ server (cloud/VPS)
- Public IP or domain (optional but recommended)
- Root or sudo access

---

## 🧱 Step 1: Update System

```bash
sudo apt update && sudo apt upgrade -y
````

---

## 🐘 Step 2: Install PostgreSQL and Create DB

```bash
sudo apt install postgresql postgresql-contrib -y
```

Create the database and user:

```bash
sudo -u postgres psql
```

```sql
CREATE USER wikiuser WITH ENCRYPTED PASSWORD 'StrongNewPassword123!';
CREATE DATABASE wikijs OWNER wikiuser;
\c wikijs
ALTER SCHEMA public OWNER TO wikiuser;
\q
```

---

## ⚙️ Step 3: Install Node.js and Git

```bash
sudo apt install curl git -y
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install nodejs -y
```

---

## 📥 Step 4: Download and Configure Wiki.js

```bash
cd ~
git clone https://github.com/Requarks/wiki.git wikijs
cd wikijs
```

Install dependencies:

```bash
npm install
```

Copy and edit config:

```bash
cp config.sample.yml config.yml
nano config.yml
```

Edit the following:

```yaml
port: 3001

db:
  type: postgres
  host: localhost
  port: 5432
  user: wikiuser
  pass: StrongNewPassword123!
  db: wikijs
```

---

## 🔃 Step 5: Test Wiki.js

```bash
node server
```

Should show:

```
Database Connection Successful [ OK ]
HTTP Server on port 3001 [ OK ]
```

---

## 🌐 Step 6: Configure NGINX Reverse Proxy

Install NGINX:

```bash
sudo apt install nginx -y
```

Remove default site:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

Create new config:

```bash
sudo nano /etc/nginx/sites-available/wikijs
```

Paste:

```nginx
server {
    listen 80;
    server_name your-server-ip-or-domain;

    location / {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable and restart:

```bash
sudo ln -s /etc/nginx/sites-available/wikijs /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

## 🚀 Step 7: Create systemd Service

```bash
sudo nano /etc/systemd/system/wikijs.service
```

Paste:

```ini
[Unit]
Description=Wiki.js
After=network.target

[Service]
Type=simple
User=your-linux-username
WorkingDirectory=/home/your-linux-username/wikijs
ExecStart=/usr/bin/node server
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable wikijs
sudo systemctl start wikijs
```

---

## 📘 Step 8: Access Wiki.js

Open in your browser:

```
http://<your-server-ip>/
```

Follow the setup wizard:

* Create Admin user
* Name your site
* Done!

---

## 🔐 Optional: Add SSL (Let’s Encrypt)

If you have a domain:

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d wiki.example.com
```

Certbot will configure HTTPS and auto-renew.

---

## ✅ Summary

| Item       | Value                          |
| ---------- | ------------------------------ |
| App Port   | 3001 (internal)                |
| NGINX Port | 80 (external)                  |
| DB Name    | `wikijs`                       |
| DB User    | `wikiuser`                     |
| Auto Start | systemd enabled                |
| Web Access | `http://<server-ip>` or domain |

---

> 🛠 Maintained by Prateek Chaudhary – DevOps & Cloud Engineer
