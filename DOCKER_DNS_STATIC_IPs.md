
---

# 📘 Static Internal IP for Containers in Docker

**(Beginner Study Material | Windows + Docker Desktop)**

---

## 🧠 What You Will Learn

By the end of this guide, you will understand:

* What **container internal IPs** are
* Why **static internal IPs** are sometimes required
* Why Docker **does NOT recommend static IPs** in most cases
* How Docker networking works (bridge network)
* How to **assign a static internal IP** to containers
* A **minimal practical demo** using Nginx
* How to **verify and test connectivity**

---

## 🧩 Prerequisites

* Windows 10 / 11
* Docker Desktop installed and running
* Basic command-line knowledge (Command Prompt / PowerShell)

Verify Docker installation:

```bash
docker --version
```

Expected output (example):

```text
Docker version 26.x.x, build xxxx
```

---

## 🔍 What is an Internal IP in Docker?

* Every container runs inside a **Docker network**
* Docker assigns an **internal IP** to each container
* This IP is:

  * Used for **container-to-container communication**
  * NOT directly accessible from your host unless port is exposed

Example internal IP:

```text
172.18.0.2
```

⚠️ **Important:**
By default, Docker assigns IPs **dynamically**, meaning:

* IP can change if container restarts
* IP can change if container is recreated

---

## ❓ Why Do We Need Static Internal IPs?

You might need static internal IPs when:

* Legacy applications require **hardcoded IPs**
* Practicing **networking concepts**
* Testing firewall rules
* Learning Docker networking in depth

🚨 **Real-world best practice:**
Docker recommends using **service/container names (DNS)** instead of static IPs.

---

## 🌐 How Docker Networking Works (Bridge Network)

![Image](https://blogs.cisco.com/gcs/ciscoblogs/1/2022/08/docker-bridge-1-768x478.jpeg?utm_source=chatgpt.com)

![Image](https://labs.iximiuz.com/content/files/challenges/reproduce-docker-bridge-network/__static__/bridge.png?utm_source=chatgpt.com)

![Image](https://www.claudiokuenzler.com/graph/news/900-container-cni-communication.png?utm_source=chatgpt.com)

* Docker creates a **bridge network**
* Each container:

  * Gets an IP from the network subnet
  * Can talk to other containers on the same network

---

## 🧪 Lab 1: Create a Custom Docker Network with Subnet

### 🔹 Step 1: Create a Custom Bridge Network

We will explicitly define a subnet.

```bash
docker network create ^
  --driver bridge ^
  --subnet 172.18.0.0/16 ^
  my_static_net
```

✅ Explanation:

* `--driver bridge` → default Docker network type
* `--subnet` → defines IP range
* `my_static_net` → network name

Verify network:

```bash
docker network ls
```

Inspect network details:

```bash
docker network inspect my_static_net
```

---

## 🧪 Lab 2: Run a Container with Static Internal IP

### 🔹 Step 2: Run Nginx with Static IP

```bash
docker run -d ^
  --name nginx-static ^
  --network my_static_net ^
  --ip 172.18.0.10 ^
  nginx
```

✅ Explanation:

* `--network my_static_net` → attach to custom network
* `--ip 172.18.0.10` → static internal IP
* `nginx` → container image

---

### 🔹 Step 3: Verify Assigned IP

```bash
docker inspect nginx-static
```

Look for:

```json
"IPAddress": "172.18.0.10"
```

🎉 Your container now has a **static internal IP**.

---

## 🧪 Lab 3: Add Another Container with Static IP

```bash
docker run -d ^
  --name nginx-static-2 ^
  --network my_static_net ^
  --ip 172.18.0.11 ^
  nginx
```

Now you have:

| Container Name | Internal IP |
| -------------- | ----------- |
| nginx-static   | 172.18.0.10 |
| nginx-static-2 | 172.18.0.11 |

---

## 🔄 Test Container-to-Container Communication

### 🔹 Step 4: Exec into One Container

```bash
docker exec -it nginx-static sh
```

Install curl (temporary):

```bash
apt update && apt install -y curl
```

Test connectivity:

```bash
curl http://172.18.0.11
```

✅ You should see Nginx default HTML page.

---

## 🧠 Important Notes (VERY IMPORTANT)

### ❌ Docker Does NOT Guarantee Static IP Safety

Static IPs can break if:

* Network is deleted
* Container is recreated
* IP conflict happens

---

## ✅ Recommended Alternative (BEST PRACTICE)

Instead of IPs, use **container names**:

```bash
curl http://nginx-static-2
```

Why this works:

* Docker provides **built-in DNS**
* Container names automatically resolve to IPs
* Much safer and production-ready

---

## 🧪 Bonus: Static IP Using Docker Compose (Minimal Example)

```yaml
version: "3.9"

networks:
  my_static_net:
    driver: bridge
    ipam:
      config:
        - subnet: 172.18.0.0/16

services:
  web1:
    image: nginx
    networks:
      my_static_net:
        ipv4_address: 172.18.0.10

  web2:
    image: nginx
    networks:
      my_static_net:
        ipv4_address: 172.18.0.11
```

Run:

```bash
docker compose up -d
```

---

## 🧹 Cleanup (Very Important)

```bash
docker rm -f nginx-static nginx-static-2
docker network rm my_static_net
```

---

## 📌 Key Takeaways

* Docker assigns **dynamic IPs by default**
* Static IPs require:

  * Custom bridge network
  * Defined subnet
* Static IPs are:

  * ❌ Not recommended for production
  * ✅ Useful for learning & legacy apps
* **Container DNS names are preferred**

---

## 🏁 Summary

| Concept               | Recommendation           |
| --------------------- | ------------------------ |
| Static IP             | ❌ Avoid in production    |
| Container name DNS    | ✅ Best practice          |
| Custom bridge network | ✅ Required for static IP |
| Docker Compose        | ✅ Cleaner setup          |

---

Just tell me 👍
