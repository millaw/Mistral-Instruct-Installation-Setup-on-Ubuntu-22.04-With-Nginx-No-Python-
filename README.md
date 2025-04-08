# 🧠 Mistral:Instruct Installation & Setup on Ubuntu 22.04 (With Nginx, No Python)

This guide will walk you through setting up [`mistral:instruct`](https://ollama.com/library/mistral) using [Ollama](https://ollama.com/) on **Ubuntu 22.04**, with **Nginx** as a reverse proxy. No Python or other unnecessary languages involved.

---

## 🧰 Prerequisites

- Ubuntu 22.04 (fresh or existing)
- 16 GB RAM (recommended for Mistral)
- Root/sudo privileges
- Internet access

---

## 🔧 Step 1: Update System

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 📦 Step 2: Install Required Packages

```bash
sudo apt install curl gnupg2 ca-certificates lsb-release nginx -y
```

---

## ⚙️ Step 3: Install Ollama (for Mistral LLM)

```bash
curl -fsSL https://ollama.com/install.sh | sh
```

```bash
sudo systemctl enable ollama
sudo systemctl start ollama
```

Verify it's running:

```bash
ollama --version
```

---

## 📥 Step 4: Download the Mistral:Instruct Model

```bash
ollama run mistral:instruct
```

Test it:

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "mistral:instruct",
  "prompt": "Explain the concept of gravity.",
  "stream": false
}'
```

---

## 🌐 Step 5: Set Up Nginx Reverse Proxy

```bash
sudo nano /etc/nginx/sites-available/ollama
```

Paste this:

```nginx
server {
    listen 80;
    server_name YOUR_DOMAIN_OR_IP;

    location / {
        proxy_pass http://localhost:11434/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable the config and reload:

```bash
sudo ln -s /etc/nginx/sites-available/ollama /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🔒 Optional: Secure with HTTPS

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx
```

---

## 🚀 Use it via Web or API

```bash
curl http://YOUR_DOMAIN_OR_IP/api/generate -d '{
  "model": "mistral:instruct",
  "prompt": "What is Ubuntu?",
  "stream": false
}'
```

---

## 🛠️ Troubleshooting

| Error | Cause | Fix |
|------|-------|-----|
| `ollama: command not found` | Ollama not in PATH | Run: `source ~/.bashrc` or re-login |
| `Failed to connect to localhost port 11434` | Ollama not running | Run: `sudo systemctl start ollama` |
| `404 Not Found` | Bad Nginx config | Reload Nginx and check config |
| SSL fails | DNS or ports | Ensure DNS points to server and ports are open |

---

## 🔄 Keeping Mistral:Instruct and Ollama Up-to-Date

### 📥 Update the Model

```bash
ollama pull mistral:instruct
```

### 🛠️ Update Ollama Engine

```bash
curl -fsSL https://ollama.com/install.sh | sh
sudo systemctl restart ollama
```

### 🔍 Version Checks

```bash
ollama list
ollama --version
```

### ⏰ Automate Daily Model Updates

```bash
crontab -e
```

Add this:

```bash
0 3 * * * /usr/local/bin/ollama pull mistral:instruct > /var/log/ollama_update.log 2>&1
```

---

## ✅ Summary

| Component | Status |
|----------|--------|
| Ubuntu 22.04 | ✅ |
| Ollama Installed | ✅ |
| Mistral Model | ✅ |
| Nginx Configured | ✅ |
| Exposed to Public | ✅ |
| Auto Updates | ✅ |

---


---

## 🆙 Optional: Upgrade to a 13B Model (Advanced)

> ⚠️ You need **at least 32 GB RAM** to run 13B models like `llama2:13b` or `mistral:13b`.

### 📥 Step 1: Pull a 13B Model

To download `llama2:13b`:

```bash
ollama pull llama2:13b
```

To download `mistral:13b` (if available):

```bash
ollama pull mistral:13b
```

> This may take **15+ GB of disk space** and several minutes to download depending on your internet speed.

### 🚀 Step 2: Run the Model

```bash
ollama run llama2:13b
```

Or using the API:

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "llama2:13b",
  "prompt": "Describe how neural networks work.",
  "stream": false
}'
```

---

### 🚨 Common Issues & Fixes

| Issue | Cause | Fix |
|-------|-------|-----|
| `cannot allocate memory` | Not enough RAM | You need 32+ GB, add swap (not ideal), or use a smaller model |
| `Killed` message | OOM (Out of Memory) | Try `mistral:7b` instead |
| System freeze or crash | Swap overuse | Do not run on systems with < 32GB |

---

### 💡 Tips

- Use SSD for better performance
- Do not run multiple large models at the same time
- For development or testing, use `mistral:7b` or `llama2:7b`

---

