# 📘 **Docker Reference Guide: Hosting a Static HTML File Using Nginx**

## 📝 **Overview**

This guide teaches beginners how a **Dockerfile works** by creating a simple Docker image that serves a static HTML file using **Nginx**.

---

## 📂 **Project Structure**

```
project-folder/
│── Dockerfile
│── index.html
```

---

## 🧱 **Dockerfile (Fully Explained)**

```Dockerfile
# ------------------------------------------------------------
# 1. Use the official Nginx image as the base
# ------------------------------------------------------------
# Nginx is already installed and configured to serve files from
# /usr/share/nginx/html, so no extra installation is needed.
# ------------------------------------------------------------
FROM nginx:latest

# ------------------------------------------------------------
# 2. Set the working directory inside the container
# ------------------------------------------------------------
# Any COPY or RUN commands that follow will execute in this path.
# This is the folder Nginx uses to serve static HTML files.
# ------------------------------------------------------------
WORKDIR /usr/share/nginx/html

# ------------------------------------------------------------
# 3. Copy your static HTML file into the container
# ------------------------------------------------------------
# The left side is the file in your local system.
# The right side is the location inside the container.
# ------------------------------------------------------------
COPY index.html .

# ------------------------------------------------------------
# 4. Expose port 80
# ------------------------------------------------------------
# This declares that the application inside the container listens
# on port 80. It does not publish the port—docker run will do that.
# ------------------------------------------------------------
EXPOSE 80

# ------------------------------------------------------------
# 5. Run Nginx in the foreground
# ------------------------------------------------------------
# Docker expects the main process to stay in the foreground.
# "daemon off" keeps Nginx running without backgrounding itself.
# ------------------------------------------------------------
CMD ["nginx", "-g", "daemon off;"]
```

---

## 🌐 **Example HTML File (`index.html`)**

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Hello Docker!</title>
  </head>
  <body>
    <h1>Welcome to Docker + Nginx</h1>
    <p>This page is being served from inside a Docker container.</p>
  </body>
</html>
```

---

## 🛠️ **Build the Docker Image**

Run this command inside your project folder:

```bash
docker build -t my-nginx-site .
```

---

## ▶️ **Run the Docker Container**

```bash
docker run -d -p 8080:80 my-nginx-site
```

Visit this URL in your browser:

👉 [http://localhost:8080](http://localhost:8080)

---

## 🧠 **What You Learned**

* How to write a basic Dockerfile
* How layers are created from each instruction
* How Nginx can serve static files inside Docker
* Difference between `EXPOSE` and `docker run -p`
* How the container starts and stays alive using `CMD`

---

## 🎯 **Next Steps**

You can extend this example by:

* Adding CSS and JS files
* Creating a custom Nginx config
* Serving multiple HTML pages
* Packaging a small frontend website

---

If you want, I can convert this into a **downloadable .md file**, or expand it into a full **Docker basics handbook**.
