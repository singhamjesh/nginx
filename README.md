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


