# High-Availability-Web
High Availability Web Architecture with Docker &amp; NGINX Load Balancing

Great 👍 you’ve already done the hardest part — understanding the architecture.
Now I’ll guide you **step-by-step like a lab manual** so you can confidently build, test, explain, and demo this HA project.

We’ll go in this order:

1. Environment Setup
2. Create Project Structure
3. Create Backend Servers (web1 & web2)
4. Configure NGINX Load Balancer
5. Create docker-compose.yml
6. Run the Project
7. Test Load Balancing
8. Simulate Failure (HA Demo)
9. Scale Backends
10. Explain It Confidently in Viva / Interview

---

# ✅ STEP 1 — Prerequisites

Make sure your VM has:

* Docker installed
* Docker Compose installed

Check:

```bash
docker --version
docker-compose --version
```

If both work → good to go.

---

# ✅ STEP 2 — Create Project Structure

Create a main project folder:

```bash
mkdir ha-demo
cd ha-demo
```

Now create folders:

```bash
mkdir web1 web2 nginx
```

Your structure should look like:

```
ha-demo/
 ├── web1/
 ├── web2/
 ├── nginx/
 └── docker-compose.yml
```

---

# ✅ STEP 3 — Create Backend Servers

Inside **web1**:

```bash
cd web1
nano index.html
```

Add:

```html
<h1>This is Web Server 1</h1>
```

Save and exit.

Now go to **web2**:

```bash
cd ../web2
nano index.html
```

Add:

```html
<h1>This is Web Server 2</h1>
```

Now both servers serve different content.

---

# ✅ STEP 4 — Configure NGINX Load Balancer

Go to nginx folder:

```bash
cd ../nginx
nano nginx.conf
```

Add:

```nginx
events {}

http {
    upstream backend {
        server web1:80;
        server web2:80;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend;
        }
    }
}
```

Save.

This config tells NGINX:

* Create backend pool
* Send traffic to web1 and web2
* Use default round-robin

---

# ✅ STEP 5 — Create docker-compose.yml

Go back to main directory:

```bash
cd ..
nano docker-compose.yml
```

Paste this:

```yaml
version: '3'

services:
  web1:
    image: nginx:alpine
    container_name: web1
    volumes:
      - ./web1:/usr/share/nginx/html:ro
    networks:
      - ha-network

  web2:
    image: nginx:alpine
    container_name: web2
    volumes:
      - ./web2:/usr/share/nginx/html:ro
    networks:
      - ha-network

  nginx-lb:
    image: nginx:alpine
    container_name: nginx-lb
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - web1
      - web2
    networks:
      - ha-network

networks:
  ha-network:
    driver: bridge
```

---

# ✅ STEP 6 — Start Everything

Run:

```bash
docker-compose up -d
```

Check:

```bash
docker ps
```

You should see:

* web1
* web2
* nginx-lb

---

# ✅ STEP 7 — Test Load Balancing

Open browser:

```
http://<your-vm-ip>
```

Refresh multiple times.

You should see alternating:

* This is Web Server 1
* This is Web Server 2

That means Round Robin is working 🎯

---

# ✅ STEP 8 — Simulate Failure (Important HA Demo)

Now simulate failure:

```bash
docker stop web1
```

Refresh browser again.

You should still get:

```
This is Web Server 2
```

System still works.

Explain this in demo:

> "Even if one backend fails, the load balancer redirects traffic to available servers. This provides service-level high availability."

Now restart:

```bash
docker start web1
```

---

# ✅ STEP 9 — Scale Backend (Advanced Demo)

Run:

```bash
docker-compose up -d --scale web1=3
```

Now check:

```bash
docker ps
```

You’ll see:

* web1_1
* web1_2
* web1_3
* web2
* nginx-lb

