# Real-Time WebSocket Chat App Deployment  
## DevOps Engineering Assignment

---

## 1. Project Overview

This project is a **real-time WebSocket chat application** deployed using:

- **FastAPI** (backend)
- **NGINX** (reverse proxy + frontend serving)
- **Docker**
- **Docker Compose**
- **GitHub Actions CI/CD**
- **AWS EC2**

The repository provided for this assignment contained a **deliberately broken deployment setup**.  
The goal of this project was to:

- Debug the infrastructure issues
- Fix Docker, Docker Compose, and NGINX configuration
- Ensure WebSocket communication works correctly
- Deploy the application on a cloud VM
- Automate deployment using GitHub Actions

---

## 2. Architecture Diagram

### High-Level Architecture

```text
+----------------------+
|   User Browser       |
+----------------------+
           |
           v
+----------------------+
|  Public IP :80       |
|  AWS EC2 Instance    |
+----------------------+
           |
           v
+----------------------+
|  NGINX Container     |
|  Reverse Proxy       |
|  Serves Frontend     |
+----------------------+
           |
           v
+----------------------+
|  FastAPI Backend     |
|  WebSocket Server    |
+----------------------+
```
## 3. How Docker Containers Are Set Up

The application is deployed as a multi-container application using Docker Compose.

Containers Used
1. Backend Container
Built using the custom Dockerfile
Runs the FastAPI application with Uvicorn
Exposes port 8000 internally
Handles WebSocket communication
2. NGINX Container
Uses the nginx:alpine image
Serves frontend files
Acts as a reverse proxy
Exposes port 80 publicly
Dockerfile Setup

The backend container:

Uses python:3.11-slim
Installs dependencies from requirements.txt
Copies backend application code
Copies frontend assets required by backend
Starts Uvicorn server
## 4. How Docker Networking Works

Docker Compose automatically creates a shared bridge network for all services.

Networking Behavior
The backend service is accessible inside Docker using:
```
http://backend:8000
```
The nginx service communicates with the backend using the Docker Compose service name backend

Why This Is Important

Inside a container:

localhost means that same container only
Containers must communicate using service names

So NGINX must connect to:
```
http://backend:8000
```
instead of:
```
http://localhost:8000
```
## 5. How NGINX Reverse Proxy Works

NGINX is used as the entry point for the application.

NGINX Responsibilities
Serve frontend files
Reverse proxy WebSocket requests to backend
Expose the application on port 80
Frontend Flow

When a user opens:
```
http://<your-public-ip>
```
NGINX serves the frontend from:
```
/usr/share/nginx/html
```
WebSocket Flow

When the frontend initiates a WebSocket connection to:
```
/ws
```
NGINX forwards it to the backend container:
```
http://backend:8000/ws
```
## 6. How WebSocket Works Through NGINX

The application uses WebSocket for real-time chat communication.

Why WebSocket Needs Special Handling

WebSocket starts as an HTTP request and then upgrades to a persistent bidirectional connection.

To support this, NGINX must include these headers:
```
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
```
Why This Matters

Without these headers:

WebSocket handshake fails
Real-time chat will not work properly
Validation Performed

The application was tested by:

Opening the app in multiple browser tabs
Sending messages between tabs
Verifying real-time message broadcasting works correctly
## 7. How CI/CD Pipeline Works

A GitHub Actions CI/CD pipeline was created to automate deployment.

Workflow File
```
.github/workflows/deploy.yml
```
Pipeline Flow:
On every push to the main branch:

GitHub Actions workflow is triggered
Connects to EC2 server using SSH
Pulls the latest code from GitHub
Rebuilds Docker containers
Restarts the application automatically
Deployment Commands Used
```
docker compose down
docker compose up -d --build
```
GitHub Secrets Used

The following GitHub Secrets were configured:
```
SERVER_HOST
SERVER_USER
SERVER_SSH_KEY
```
These secrets allow secure automated deployment without exposing credentials.

## 8. What Issues I Found and How I Fixed Them

The repository provided in the assignment contained several deployment misconfigurations.

Issue 1: Backend bound to localhost
Problem

The backend was running with:
```
127.0.0.1
```
This made the FastAPI application inaccessible from other containers.

Fix

Changed backend host binding to:
```
0.0.0.0
```
This made the backend accessible within the Docker network.

Issue 2: Frontend files were missing in backend image
Problem

The backend expected frontend files under:
```
/frontend/index.html
```
But the frontend directory was not copied into the image.

Fix

Added this to the Dockerfile:
```
COPY frontend /frontend
```
This ensured the backend could access required frontend files.

Issue 3: Frontend was not mounted into NGINX
Problem

The frontend directory was not mounted into the NGINX container, so NGINX could not serve the HTML files.

Fix

Added this volume mount in docker-compose.yml:
```
- ./frontend:/usr/share/nginx/html:ro
```
This allowed NGINX to serve the frontend correctly.

Issue 4: NGINX was proxying WebSocket traffic to localhost
Problem

NGINX was configured with:
```
proxy_pass http://localhost:8000/ws;
```
Inside the NGINX container, localhost refers to the NGINX container itself, not the backend container.

Fix

Changed it to:

proxy_pass http://backend:8000/ws;

This correctly routed WebSocket traffic to the backend container.

Issue 5: WebSocket upgrade headers were disabled
Problem

The required WebSocket upgrade headers were commented out in nginx.conf.

Fix

Enabled these headers:
```
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
```
This allowed WebSocket connections to upgrade properly.

## 9. Steps to Deploy the Project
Step 1: Clone the Repository
```
git clone <your-github-repo-link>
cd devops
Step 2: Start the Application
docker-compose up -d --build
```
Step 3: Verify Running Containers
```
docker ps
```
Step 4: Access the Application

Open in browser:
```
http://<your-public-ip>
```
 Step 5: Test WebSocket Functionality
Open the app in multiple tabs
Send chat messages
Verify real-time communication works correctly
## 10. Live Public IP

Application URL:
```
http://<your-public-ip>
```
Replace <your-public-ip> with your actual EC2 public IP.

## 11. Required Project Files Included

This repository contains the required files:

Dockerfile
docker-compose.yml
nginx.conf
.github/workflows/deploy.yml
README.md
## 12. Cloud Deployment Details

This project was deployed on:

AWS EC2
Required EC2 Configuration
Port 22 open for SSH
Port 80 open for HTTP
Docker installed
Docker Compose installed
Git installed
## 13. Container Restart Behavior

The containers are configured with:
```
restart: always
```
This ensures:

Containers restart automatically if they crash
Services recover after server restart
## 14. Validation Performed

The following validations were completed successfully:

Docker containers build correctly
Frontend loads successfully
Backend is reachable through Docker network
NGINX serves frontend correctly
WebSocket works through NGINX
Multi-user chat works across multiple browser tabs
Application is accessible via public IP
GitHub Actions redeploys the application automatically
## 15. Submission Deliverables

This repository includes all required submission items:

GitHub Repository
Live Public IP
Architecture Diagram
README Documentation
## 16. Conclusion

This project successfully demonstrates:

Debugging broken deployment configurations
Docker containerization
Docker Compose networking
NGINX reverse proxy configuration
WebSocket proxy handling
Cloud deployment on AWS EC2
CI/CD automation using GitHub Actions

---

## Replace these before pushing:
- `<your-github-repo-link>` → your GitHub repo URL
- `<your-public-ip>` → your EC2 public IP

If you want, next I can give you the **architecture diagram as a clean PNG-style version** you can put in your repo.
