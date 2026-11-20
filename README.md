# Containerization of a Web Application Using Docker

This project demonstrates how to package and run a simple web application consistently across different environments by building a **Docker Image** from a **Dockerfile** and running it inside an isolated **Docker Container** with port mapping.

---

## Architecture Flow

```text
[index.html + Dockerfile] --(docker build)--> [Docker Image] --(docker run -p 8080:80)--> [Running Docker Container] --> [Browser]
```

---

## Important Files, Paths & Ports Reference

| Component | Path / Port | Purpose |
| :--- | :--- | :--- |
| **Project Directory** | `~/docker-webapp/` (or `C:\docker-webapp\`) | Working folder containing app files and `Dockerfile` |
| **Web Application File** | `./index.html` | Simple HTML web page to be containerized |
| **Build Instructions** | `./Dockerfile` | Defines the base image (`nginx` or `httpd`) and file copy steps |
| **Container Web Root (Nginx)** | `/usr/share/nginx/html/` | Default web directory inside the `nginx:alpine` container |
| **Container Web Root (Apache)** | `/usr/local/apache2/htdocs/` | Default web directory inside the `httpd:alpine` container |
| **Port Mapping (`-p 8080:80`)** | Host `8080` $\rightarrow$ Container `80` | Exposes the container's web server to `http://localhost:8080` |

---

## Step-by-Step Setup Guide

### Step 0: Install & Start Docker (If Not Already Installed)

#### On Ubuntu / AWS EC2:
```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
docker --version
```
*(If running on AWS EC2, make sure **Port 8080** is allowed in your Security Group inbound rules.)*

---

### Step 1: Develop a Simple Web Application

1. Create a project folder and navigate into it:
   ```bash
   mkdir docker-webapp && cd docker-webapp
   ```

2. Create an `index.html` file:
   ```bash
   cat << 'EOF' > index.html
   <!DOCTYPE html>
   <html>
   <head><title>DevOps Docker Lab</title></head>
   <body>
     <h1>Hello from Docker Container!</h1>
     <p>Containerized Web Application - Question 3</p>
   </body>
   </html>
   EOF
   ```

---

### Step 2: Create a `Dockerfile`

Create a file named `Dockerfile` (exact capitalization, no file extension) in the same directory:

```bash
cat << 'EOF' > Dockerfile
# 1. Use official lightweight Nginx web server as the base image
FROM nginx:alpine

# 2. Copy local index.html into the container's web root directory
COPY index.html /usr/share/nginx/html/index.html

# 3. Document that the container listens on port 80
EXPOSE 80
EOF
```

---

### Step 3: Build the Docker Image

Build a custom image tagged `my-webapp:v1` from the current directory (`.`):
```bash
docker build -t my-webapp:v1 .
```

Verify that the image was created:
```bash
docker images
```

---

### Step 4 & 5: Run the Docker Container & Configure Port Mapping

Start a detached (`-d`) container named `webapp-container` and map host port `8080` to container port `80` (`-p 8080:80`):
```bash
docker run -d --name webapp-container -p 8080:80 my-webapp:v1
```

Verify the container is up and running:
```bash
docker ps
```

---

### Step 6: Verify Application Accessibility Through a Browser

- **On Local Machine:** Open `http://localhost:8080`
- **On AWS EC2:** Open `http://<EC2_PUBLIC_IP>:8080`

You should see **"Hello from Docker Container!"** rendered in the browser.

---

### Step 7: Stop and Remove Containers (Cleanup Commands)

```bash
# 1. Stop the running container
docker stop webapp-container

# 2. Verify it is stopped (shows all containers including stopped ones)
docker ps -a

# 3. Remove the stopped container
docker rm webapp-container

# 4. (Optional) Remove the Docker image
docker rmi my-webapp:v1
```

---

## Lab Exam Demonstration Checklist

Follow this exact sequence when presenting to the examiner:

1. **Display `Dockerfile` & Web App:** Run `cat Dockerfile` and `cat index.html` to explain the `FROM`, `COPY`, and `EXPOSE` instructions.
2. **Demonstrate Image Creation:** Run `docker build -t my-webapp:v1 .` followed by `docker images` to show the newly built image.
3. **Run & Show Running Containers:** Run `docker run -d --name webapp-container -p 8080:80 my-webapp:v1` and `docker ps` to display the active container and port mapping (`0.0.0.0:8080->80/tcp`).
4. **Access App in Browser:** Open `http://localhost:8080` (or `http://<EC2_PUBLIC_IP>:8080`) to prove the containerized app is live.
5. **Stop & Remove Container:** Run `docker stop webapp-container` and `docker rm webapp-container`, then verify with `docker ps -a` that the container is removed.
