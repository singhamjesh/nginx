# 🚀 Nginx on Ubuntu with Docker

This guide explains how to run **Ubuntu inside Docker**, install **Nginx**, and expose it on port `8080` so you can see the Nginx welcome page in your browser.  
It also covers how to **customize Nginx configuration** and serve your own HTML pages.

---

## 🖼 Architecture Diagram

![Nginx Docker Flow](https://raw.githubusercontent.com/amjesh/example-assets/main/nginx-docker-architecture.png)

**Flow:**
1. You run Docker Desktop on macOS/Windows/Linux.
2. Inside Docker, an **Ubuntu container** is started.
3. Nginx is installed inside the Ubuntu container.
4. Port `8080` on your machine is mapped to port `80` in the container.
5. You can access the Nginx server from your browser at `http://localhost:8080`.

---

## 🛠 Prerequisites
- Docker Desktop installed (on macOS, Windows, or Linux).
- Basic command line knowledge.

---

## 🔹 Steps

### 1. Run Ubuntu Container
```bash
docker run -it -p 8080:80 --name myubuntu ubuntu
````

* `-it` → interactive mode with terminal.
* `-p 8080:80` → maps host port `8080` → container port `80`.
* `--name myubuntu` → names the container `myubuntu`.
* `ubuntu` → base image from Docker Hub.

---

### 2. Verify Ubuntu

Inside the container:

```bash
uname -a
```

This confirms you are inside Ubuntu running in Docker.

---

### 3. Update Package Repository

```bash
apt-get update
```

---

### 4. Install Nginx

```bash
apt-get install nginx -y
```

Check version:

```bash
nginx -v
```

---

### 5. Start Nginx

Run Nginx in **foreground mode** so Docker doesn’t stop the container:

```bash
nginx -g "daemon off;"
```

Now open your browser:

```
http://localhost:8080
```

🎉 You should see the **Nginx Welcome Page**.

---

## 🔹 Nginx Useful Commands

* Check configuration syntax:

  ```bash
  nginx -t
  ```
* Reload config without restarting:

  ```bash
  nginx -s reload
  ```
* Default configuration directory:

  ```bash
  cd /etc/nginx
  ```

---

## 🔹 Custom Configuration & HTML

### 1. Add Your Own HTML

Default web root:

```bash
cd /var/www/html
```

Replace the default page:

```bash
echo "<h1>Hello from Custom Nginx 🚀</h1>" > index.html
```

Visit:

```
http://localhost:8080
```

Now you’ll see your custom page.

---

### 2. Edit Nginx Configuration

Main config file:

```bash
nano /etc/nginx/nginx.conf
```

Or edit virtual host config:

```bash
nano /etc/nginx/sites-available/default
```

After changes, reload Nginx:

```bash
nginx -s reload
```

---

### 3. Example: Reverse Proxy

Update `/etc/nginx/sites-available/default`:

```nginx
server {
    listen 80;

    location /api/ {
        proxy_pass http://your-backend-service:5000/;
    }

    location / {
        root /var/www/html;
        index index.html;
    }
}
```

Check syntax:

```bash
nginx -t
```

Reload:

```bash
nginx -s reload
```

---

## 🔹 Restarting the Container

If you exit the container, restart it with:

```bash
docker start -ai myubuntu
```

---

## 🔹 One-Command Setup (Optional)

Run everything in a single command:

```bash
docker run -d -p 8080:80 --name mynginx ubuntu bash -c "apt-get update && apt-get install -y nginx && echo '<h1>Hello from Docker Nginx 🚀</h1>' > /var/www/html/index.html && nginx -g 'daemon off;'"
```

* `-d` → detached mode (runs in background).
* Installs Nginx + adds a custom page + runs it directly.

---


## ✅ Done!

You now have **Ubuntu + Nginx running in Docker**, accessible at:

👉 [http://localhost:8080](http://localhost:8080)

You can customize configs under `/etc/nginx/` and HTML files under `/var/www/html/`.


---

# 📖 Nginx File & Directory Structure with Examples

This guide explains the purpose of each important Nginx file/directory and provides examples for better understanding.

---

## 📂 File & Directory Overview

### 1. `/etc/nginx/nginx.conf` → Main Config File
Controls global settings, worker processes, and includes other configs.

```nginx
user nginx;
worker_processes auto;

events {
    worker_connections 1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent"';

    access_log /var/log/nginx/access.log main;
    error_log  /var/log/nginx/error.log warn;

    include /etc/nginx/conf.d/*.conf;   # Load all configs from conf.d
}
````

---

### 2. `/etc/nginx/conf.d/` → Extra Configs

Files here are auto-loaded. Each `.conf` usually contains a **server block**.

**Example: `/etc/nginx/conf.d/myapp.conf`**

```nginx
server {
    listen 80;
    server_name myapp.local;

    root /var/www/myapp;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

---

### 3. `/etc/nginx/sites-available/` (Debian/Ubuntu only)

Holds **all possible sites**, but not active until symlinked.

**Example: `/etc/nginx/sites-available/example.com`**

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    root /var/www/example.com;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Enable it:

```bash
ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
```

---

### 4. `/etc/nginx/sites-enabled/` (Debian/Ubuntu only)

Contains **symlinks** to active sites.

```bash
/etc/nginx/sites-enabled/example.com -> ../sites-available/example.com
```

---

### 5. `/etc/nginx/modules-enabled/` → Dynamic Modules

Defines which extra modules to load.

**Example:**

```nginx
load_module modules/ngx_http_image_filter_module.so;
```

---

### 6. `/etc/nginx/mime.types` → File Type Mapping

Ensures correct `Content-Type` headers are sent.

**Example (snippet):**

```nginx
types {
    text/html   html htm;
    text/css    css;
    text/javascript js;
    image/jpeg  jpeg jpg;
    application/json json;
}
```

---

### 7. `/etc/nginx/snippets/` → Reusable Configs

Reusable blocks included inside other configs.

**Example: `/etc/nginx/snippets/ssl-params.conf`**

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers on;
ssl_ciphers HIGH:!aNULL:!MD5;
```

Include it:

```nginx
server {
    listen 443 ssl;
    include snippets/ssl-params.conf;
}
```

---

### 8. `/usr/sbin/nginx` → Nginx Executable

The actual binary. Useful commands:

```bash
nginx -t        # test config
nginx -s reload # reload service
```

---

### 9. `/var/log/nginx/` → Logs

* `access.log` → every request
* `error.log` → errors and issues

**Example Log Entry:**

```
192.168.1.1 - - [19/Sep/2025:05:30:22 +0530] "GET /index.html HTTP/1.1" 200 512 "-" "Mozilla/5.0"
```

---

### 10. `/var/www/` → Website Files

Default document root.

**Example: `/var/www/html/index.html`**

```html
<!DOCTYPE html>
<html>
<head><title>Welcome</title></head>
<body>
    <h1>Hello from Nginx!</h1>
</body>
</html>
```

---

### 11. `/lib/systemd/system/nginx.service` → Service File

Controls start/stop/restart with `systemctl`.

**Example:**

```ini
[Unit]
Description=A high performance web server and a reverse proxy server
After=network.target

[Service]
ExecStart=/usr/sbin/nginx -g 'daemon off;'
ExecReload=/usr/sbin/nginx -s reload
PIDFile=/run/nginx.pid

[Install]
WantedBy=multi-user.target
```

---

### 12. `/run/nginx.pid` → PID File

Stores the process ID of the main Nginx process.

**Check PID:**

```bash
cat /run/nginx.pid
```

---

## ✅ Summary

* **`nginx.conf`** → global settings
* **`conf.d/`** → active site configs
* **`sites-available` + `sites-enabled`** → enable/disable virtual hosts (Debian/Ubuntu)
* **`snippets/`** → reusable configs (SSL, PHP, etc.)
* **`mime.types`** → file type mappings
* **`logs/`** → access & error logs
* **`www/`** → website files
* **systemd files** → service management

---




