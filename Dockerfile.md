# Dockerfile Instructions Guide

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


