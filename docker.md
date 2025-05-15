**Q:** What is the difference between CMD and ENTRYPOINT?

**A:** 
CMD sets default arguments that can be overridden at runtime, while ENTRYPOINT defines a fixed executable. If both are present, CMD supplies parameters to ENTRYPOINT. This is useful for tools like ping, where I fix the command but allow override of the target host.

**Q:** Why would you use ONBUILD?

**A:** 
ONBUILD is used to define triggers in base images that execute when a downstream image is built. For example, if I create a base image for Node.js apps, I might add ONBUILD COPY . /app so that any inheriting Dockerfile must provide app code.


### 🔹 `FROM`

**Q:** What does the `FROM` instruction do in a Dockerfile?

**A:** It sets the base image for the build. It must be the first instruction unless using `ARG` to define the base image tag.

---

### 🔹 `RUN`

**Q:** When is the `RUN` instruction executed?

**A:** At build time. It's used to install packages and configure the image environment.

**Q:** Can I use `cd` in a `RUN` command to change directories?

**A:** No. Each `RUN` starts in a new shell. Use `WORKDIR` instead.

---

### 🔹 `CMD`

**Q:** What is the purpose of `CMD`?

**A:** It defines the default command to run when the container starts. It can be overridden at runtime.

---

### 🔹 `ENTRYPOINT`

**Q:** How is `ENTRYPOINT` different from `CMD`?

**A:** `ENTRYPOINT` defines a fixed command; arguments passed at runtime are appended. Use it when the main process should not be overridden.

---

### 🔹 `CMD` vs `ENTRYPOINT`

**Q:** Can you combine `CMD` and `ENTRYPOINT`?

**A:** Yes. `ENTRYPOINT` defines the executable; `CMD` supplies default arguments. Useful for override-friendly behavior.

```dockerfile
ENTRYPOINT ["ping", "-c5"]
CMD ["google.com"]
```

---

### 🔹 `COPY` vs `ADD`

**Q:** When should I use `COPY` over `ADD`?

**A:** Prefer `COPY`—it’s simpler and predictable. Use `ADD` only when you need automatic tar extraction or downloading from URLs.

---

### 🔹 `ENV`

**Q:** What does `ENV` do?

**A:** It sets environment variables available during both build and container runtime.

---

### 🔹 `ARG`

**Q:** What is the difference between `ARG` and `ENV`?

**A:** `ARG` is available only at build time, `ENV` persists into the container.

**Q:** How do I use an `ARG` to pass a base image version?

**A:**

```dockerfile
ARG version
FROM almalinux:${version:-8}
```

---

### 🔹 `LABEL`

**Q:** What is the use of `LABEL`?

**A:** It adds metadata to an image (e.g., author, version). Useful for filtering and documentation.

---

### 🔹 `EXPOSE`

**Q:** Does `EXPOSE` publish a port?

**A:** No. It documents the intended port to use. Use `-p` flag during `docker run` to publish.

---

### 🔹 `USER`

**Q:** Why use the `USER` instruction?

**A:** To run the container as a non-root user, improving security.

---

### 🔹 `WORKDIR`

**Q:** What does `WORKDIR` do?

**A:** Sets the working directory inside the image. Unlike `RUN cd`, it persists across Dockerfile instructions.

---

### 🔹 `ONBUILD`

**Q:** What is `ONBUILD` used for?

**A:** It sets a trigger to run commands in child images. Ideal for reusable base images that enforce build structure.

---

### 🔹 Image Pushing

**Q:** How do you push an image to Docker Hub or Nexus?

**A:**

```bash
docker login
docker tag image username/repo:tag
docker push username/repo:tag
```

---

