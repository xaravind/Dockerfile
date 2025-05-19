# Dockerfile Instructions

---

## FROM

The `FROM` instruction should be the **first instruction** in a Dockerfile. It specifies the **base image** for your Docker image.

```dockerfile
FROM image-name:tag
```

---

## RUN

The `RUN` instruction is used to **execute commands** in the container **at build time** (i.e., while creating the image). It's commonly used to install and configure packages.

```dockerfile
RUN <<EOF
apt-get update
apt-get install -y curl
EOF
```

---

## CMD

The `CMD` instruction is used to **specify the default command** to run **when the container starts**.

```dockerfile
CMD ["executable", "param1", "param2"]
```

### Example

```dockerfile
CMD ["nginx", "-g", "daemon off;"]  # Starts nginx in the foreground
```

---

## RUN vs CMD

| Instruction | Executes At       | Purpose                              |
| ----------- | ----------------- | ------------------------------------ |
| `RUN`       | Build time        | Install software, set up environment |
| `CMD`       | Container runtime | Default action when container starts |

---

## LABEL

Used to **add metadata** to images using **key-value pairs**. Useful for filtering images or adding descriptive information like author, version, etc.

```dockerfile
LABEL author="name" \
      version="1.0" \
      description="frontend-app"
```

### Filter by label:

```bash
docker images -f label=version=1.0
```

---

## EXPOSE

Informs Docker that the container will **listen on the specified network port** at runtime. This is **only informational**—it doesn't publish the port.

```dockerfile
EXPOSE 80
```

To inspect exposed ports:

```bash
docker inspect <image-id or image-name>
```

---

## ENV

The `ENV` instruction sets **environment variables** inside the container. These variables persist when the container is started from the image.

```dockerfile
ENV name="aravind" \
    role="devops engineer" \
    exp="3*"
```

### Inspect ENV variables:

```bash
docker inspect <container-id>
```

### Override at runtime:

```bash
docker run --env name="john" image-name
```

---

## COPY vs ADD

Both are used to copy files from your build context into the image. However, `ADD` offers **additional functionality**:

| Instruction | Basic Copy | Remote URL Download | Automatic Extraction (.tar) |
| ----------- | ---------- | ------------------- | --------------------------- |
| `COPY`      | Yes        | No                  | No                          |
| `ADD`       | Yes        | Yes                 | Yes                         |

---

## CMD vs ENTRYPOINT

### CMD

Can be **overridden** at runtime.

```dockerfile
CMD ["ping", "-c5", "google.com"]
```

### ENTRYPOINT

Cannot be overridden. If overridden, additional args are **appended**, not replaced.

```dockerfile
ENTRYPOINT ["ping", "-c5", "google.com"]
```

### Example Issue:

```bash
docker run -it entry:2.0 ping -c5 yahoo.com
```

Output:

```bash
ping: ping: Name or service not known
```

Final command becomes:

```bash
"ping -c5 google.com ping -c5 yahoo.com"
```

---

## Combine CMD & ENTRYPOINT

This is useful when you want a fixed command (`ENTRYPOINT`) but allow arguments to be overridden via `CMD`.

```dockerfile
FROM almalinux:9
ENTRYPOINT ["ping", "-c5"]
CMD ["google.com"]
```

### Runtime:

```bash
docker run entry:3.0 yahoo.com
docker run entry:3.0 gmail.com
```

---

## USER

Avoid running containers as `root`. Use `USER` to switch to a non-root user.

```dockerfile
FROM almalinux:9
RUN adduser docker
USER docker
CMD ["sleep", "100"]
```

### Verify:

```bash
docker exec -it <container-id> bash
# You should see: [docker@container-id /]$
```

---

## WORKDIR

Sets the **working directory** inside the container. Subsequent commands (like `RUN`, `CMD`) will be run from this directory.

```dockerfile
FROM almalinux:9
RUN mkdir /app
WORKDIR /app
RUN pwd
CMD ["sleep", "100"]
```

---

## ARG

`ARG` defines a variable **only available at build time**.

```dockerfile
ARG version
FROM almalinux:${version}
```

### Usage with default:

```dockerfile
ARG version
FROM almalinux:${version:-8}
```

### Pass value at build time:

```bash
docker build -t arg:1.0 --build-arg version=8 .
```

### Access `ARG` in container via `ENV`:

```dockerfile
ARG ENV=PROD
ENV ENV=${ENV}
```

| Feature | Accessible During Build | Accessible During Runtime |
| ------- | ----------------------- | ------------------------- |
| `ARG`   | Yes                     | No                        |
| `ENV`   | Yes                     | Yes                       |

---

## ONBUILD

Adds a **trigger instruction** to the image, which will run when the image is used **as a base image** in another Dockerfile.

Useful for enforcing best practices (e.g., requiring files in child builds).

```dockerfile
FROM almalinux:9
RUN dnf install nginx -y
RUN rm -rf /usr/share/nginx/html/index.html
ONBUILD COPY index.html /usr/share/nginx/html/index.html
CMD ["nginx", "-g", "daemon off;"]
```

---
## Pushing Images

### To Docker Hub

```bash
docker login              # Authenticate with Docker Hub
docker tag image-name username/image-name:version
docker push username/image-name:version
```

### To Nexus

```bash
docker tag image-name label.<nexus-url>/username/image-name:v1
docker push label.<nexus-url>/username/image-name:v1
```

You can push images to:

* Docker Hub
* Nexus
* Amazon ECR
* JFrog Artifactory
  
---


#  Deploying a 3-Tier Architecture with Docker and Docker Compose

Time to roll up our sleeves, it’s time to get into some **REAL WORK** — to deploy a fully containerized **3-tier architecture** using **Dockerfile** and **docker-compose.yml**, and bring all containers up in one shot. Let’s go! 💪🔥

---

##  Step 1: Create a VM (EC2)

I launched an **AWS EC2 instance** (`t3.medium`) and configured the **security group** to allow `ALL TCP` traffic for testing purposes.
Then SSH’d into the instance and switched to root:

```bash
sudo -i
```

---

##  Step 2: Install Dependencies

Install Git and Docker:

```bash
yum install git -y
yum install docker -y
```

Install Docker Compose plugin:

```bash
mkdir -p ~/.docker/cli-plugins/
curl -SL https://github.com/docker/compose/releases/download/v2.27.1/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
chmod +x ~/.docker/cli-plugins/docker-compose
```

That’s all we need! No extra build or packaging tools are required — we’ll leverage **multi-stage Dockerfiles** for optimal builds! 

---

##  Step 3: Clone the Code

```bash
git clone -b <branch> https://github.com/backend.git
git clone -b <branch> https://github.com/frontend.git
```

---

## 🌐 Step 4: Frontend Setup

Navigate to the `frontend` folder and create:

* `Dockerfile`
* `nginx.conf`

```bash
[root@ip-172-31-85-16 frontend]#ls Dockerfile nginx.conf
Dockerfile nginx.conf
```

### Dockerfile (Frontend)

```Dockerfile
FROM node:lts-alpine3.20 AS build

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

FROM nginx:alpine

COPY --from=build /app/dist/ /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
```

### nginx.conf

```nginx
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        rewrite ^/api/(.*)$ /$1 break;
        proxy_pass http://backend:8080;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|svg|ico|woff|woff2|ttf|otf|eot)$ {
        expires 30d;
        access_log off;
        add_header Cache-Control "public";
    }

    error_page 404 /index.html;
    error_page 500 502 503 504 /50x.html;

    location = /50x.html {
        root /usr/share/nginx/html;
    }
}
```

> 🔄 **I updated this file to use `backend` as the hostname for the API proxy — Docker DNS magic!**


When you define a custom network in your docker-compose.yml (e.g., with driver: bridge), 

* Each service can **refer to the others by their service names as hostnames**.
* For example, the service named `backend` can be reached simply by using `http://backend` inside any other container on the same Docker network.

---


### Build the Frontend Image

```bash
docker build -t frontend:1.0 .
```

---

##  Step 5: Backend Setup

Navigate to the `backend` folder and create the `Dockerfile`.

```bash
[root@ip-172-31-85-16 backend]#ls Dockerfile
Dockerfile
```
### Dockerfile (Backend)

```Dockerfile
FROM maven:3.9.9-eclipse-temurin-21-alpine AS build

WORKDIR /app
COPY . .
RUN mvn package -DskipTests

FROM openjdk:21-slim

WORKDIR /app

RUN groupadd -r ajabackend && useradd -r -g ajabackend ajabackend
COPY --from=build /app/target/*.jar /app/app.jar
RUN chown -R ajabackend:ajabackend /app

USER ajabackend

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

> ✅ Following best practices: minimal base image, fewer layers, and running as **non-root** user for enhanced security!

---

> ✅ Following best practices: minimal base image, fewer layers, and running as **non-root** user for enhanced security!

### 1. **Minimal base image**

* Using a **minimal base image** (e.g., `alpine`, `slim`, or lightweight JREs) means your image includes only the essential components needed to run your application.
* This reduces:

  * The overall image size, making downloads and deployments faster.
  * The attack surface — fewer packages means fewer vulnerabilities.
* For example, using `openjdk:17-jre-slim` or `eclipse-temurin:17-jre-alpine` keeps the runtime environment small and efficient.



### 2. **Fewer layers**

* Each `RUN`, `COPY`, or other Dockerfile instruction adds a new image layer.
* Consolidating commands into fewer instructions minimizes the number of layers.
* Fewer layers mean:

  * Smaller images.
  * Faster build times.
  * Easier caching and image management.
* For example, combining user creation and permissions adjustments in a single `RUN` command avoids extra layers.


### 3. **Running as a non-root user**

* Running the container’s main process as a **non-root user** (e.g., `ajabackend` user you created) significantly enhances security.
* Benefits include:

  * Limiting potential damage if the container is compromised.
  * Preventing accidental or malicious access to sensitive system files or the host.
* This follows the principle of least privilege, which is a core security best practice.
* It’s much safer than running as the default root user inside containers.



### Overall Benefits:

* **Smaller, more secure, and faster images**
* **Improved security posture** in production environments
* **Better maintainability** and consistency across builds and deployments

---


### Build the Backend Image

```bash
docker build -t backend:1.0 .
```

Clean up unnecessary artifacts to save space:

```bash
docker system prune -a
```

```bash
[root@ip-172-31-85-16 backend]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED             SIZE
frontend     1.0       1db9f5b7436b   23 minutes ago      50MB
backend      1.0       402b2e85ab8a   About an hour ago   619MB
mysql        8.0       00a697b8380c   4 weeks ago         772MB
```


---

##  Why Docker DNS & Custom Network is Awesome

* Services like `backend`, `mysql`, and `frontend` are automatically discoverable via DNS within the Docker custom network.
* No need to hardcode IPs — services resolve each other by **service name**.
* This makes the system **portable**, **clean**, and **cloud-native**.

---

## 🧰 Step 6: Docker Compose - Hero Time! 💥

```bash
[root@ip-172-31-85-16 ~]# ls
backend  frontend  docker-compose.yml
```
create `docker-compose.yml`:

```yaml
volumes:
  mysql_data:

networks:
  app_net:
    driver: bridge

services:
  mysql:
    image: mysql:8.0
    container_name: mysql
    ports:
      - "3307:3306"
    environment:
      MYSQL_ROOT_PASSWORD: admin@123
      MYSQL_DATABASE: sample
      MYSQL_USER: admin
      MYSQL_PASSWORD: Admin@1234!
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - app_net
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-uadmin", "-pAdmin@1234!"]
      interval: 10s
      timeout: 5s
      retries: 5

  backend:
    image: backend:1.0
    container_name: backend
    depends_on:
      mysql:
        condition: service_healthy
    ports:
      - "8080:8080"
    environment:
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/sample
      SPRING_DATASOURCE_USERNAME: admin
      SPRING_DATASOURCE_PASSWORD: Admin@1234!
      SERVER_ADDRESS: 0.0.0.0
    networks:
      - app_net

  frontend:
    image: frontend:1.0
    container_name: frontend
    depends_on:
      - backend
    ports:
      - "80:80"
    networks:
      - app_net
```

> ✅ **Custom network**, **volume for data persistence**, **inter-service dependency with health checks** — all done right!


I’ve set up a **custom network** to isolate services and enable them to communicate using their service names as hostnames. I’ve used a **named volume** to ensure database data persists across container restarts and upgrades. I’ve also added **health checks with `depends_on`** so dependent services start only after their dependencies are healthy, improving reliability. This setup follows Docker best practices for a solid and maintainable environment.


> 🔐 While this setup uses hardcoded credentials for demo purposes, Docker Secrets or `.env` files are best practices for production. We can also integrate Jenkins credentials management to securely handle passwords and sensitive information during CI/CD pipelines, ensuring secrets are never exposed in plain text. Additionally, there are many other best practices available to safely store and manage secrets.
---

## 🚀 Step 7: Launch All Containers

```bash
docker compose up -d
```

🎉 Boom! Within seconds, all containers are up and running:

```bash
[root@ip-172-31-85-16 frontend]# docker compose up -d
[+] Running 4/4
 ✔ Network root_app_net  Created                                                                                                                     0.2s
 ✔ Container mysql       Healthy                                                                                                                    10.9s
 ✔ Container backend     Started                                                                                                                    11.3s
 ✔ Container frontend    Started  
```

---

## 🔥 Testing the Setup

I used **Thunder Client** (Visual Studio Code extension) to test the `POST /register` API:

```
POST http://<frontend-ip>/api/register
```

Once registered, I accessed the **frontend UI** in the browser, logged in with the newly created user, and it worked flawlessly! ✨

* Frontend → hits `/api` → proxies to backend
* Backend → connects to MySQL using Docker DNS (`mysql`)
* Data → gets validated and stored 🔗

---

## 🌟 Conclusion & Key Wins

✅ Deployed a full **3-tier app** (frontend + backend + DB) in under **5 minutes**!
✅ All containers run in **one isolated custom network** — zero public IP hassle.
✅ Used **Docker best practices**: multi-stage builds, non-root users, persistent volumes, and health checks.
✅ Easily scalable, portable, and production-ready foundation.
✅ No hardcoded IPs — just clean service discovery via Docker DNS.
✅ All configuration centralized in one `docker-compose.yml` file.

**So cool, right?!**
This is the power of Docker + Compose — build, ship, and run anywhere! 🐳🔥


next phase is  **Kubernetes** for container orchestration! 
🌟 Imagine our app automatically scaling up or down, healing itself when things go wrong, and running with zero downtime — like magic! 🔄✨ With powerful features like load balancing, rolling updates, and fault tolerance, K8s takes everything to the next level.🧠

---







