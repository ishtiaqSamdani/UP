# Docker

---

## 1. Optimizing Docker Images

* **Multi-Stage Builds:** Use multiple `FROM` instructions to separate build-time dependencies from the final runtime image.

    ```dockerfile
    # Stage 1: Node environment to build the React app
    FROM node:22-alpine AS build
    WORKDIR /app
    COPY . .
    RUN npm install
    RUN npm run build
    
    # Stage 2: Serve the built app via Nginx
    FROM nginx:alpine
    COPY nginx.conf /etc/nginx/conf.d/default.conf
    COPY --from=build /app/build /usr/share/nginx/html/
    EXPOSE 80
    CMD ["nginx", "-g", "daemon off;"]

    ```

* **Caching Layers:** Order instructions from least to most frequently changing to leverage Docker's build cache effectively.

    ```dockerfile
    FROM node:18-alpine
    WORKDIR /app

    # 1. Copy dependency manifest (changes less often)
    COPY package*.json ./

    # 2. Install dependencies (layer is cached if manifest doesn't change)
    RUN npm install

    # 3. Copy source code (changes most often)
    COPY . .

    CMD ["node", "index.js"]
    ```

* **.dockerignore:** Exclude files and directories from the build context to keep images small and secure.

    **.dockerignore file example:**
    ```
    .git
    node_modules
    npm-debug.log
    Dockerfile
    .dockerignore
    ```

* **Lean Base Images:** Start with the smallest possible base image that meets your needs, such as `alpine` or `distroless`.

* **Minimizing Layers:** Chain related `RUN` commands together using `&&` to reduce the number of image layers.

    ```dockerfile
    # Good: One layer
    RUN apt-get update && apt-get install -y curl

    # Bad: Two layers
    # RUN apt-get update
    # RUN apt-get install -y curl
    ```

---

## 2. Common Dockerfile Instructions

These are the fundamental building blocks of any Dockerfile.

* **`FROM`**: Specifies the base image for your build.
    * `FROM ubuntu:22.04`

* **`RUN`**: Executes a command in a new layer to install software or configure the image.
    * `RUN pip install -r requirements.txt`

* **`COPY`**: Copies files or directories from your host into the container's filesystem.
    * `COPY . /app`

* **`CMD`**: Provides the default command to execute when a container starts, which can be overridden.
    * `CMD ["python", "app.py"]`

* **`EXPOSE`**: Documents which network ports the container listens on at runtime.
    * `EXPOSE 8080`

---

## 3. `CMD` vs. `ENTRYPOINT`

While they both define what command runs when a container starts, they have different purposes.

* **`CMD`**: Sets a default, overridable command and/or parameters for a container.

* **`ENTRYPOINT`**: Configures a container to run as a specific executable, making it less easily overridden.

**How They Work Together:**
Use `ENTRYPOINT` for the executable and `CMD` for the default parameters.

**Example:**

```dockerfile
ENTRYPOINT ["/usr/bin/git"]
CMD ["--help"]
```

* `docker run <image>` executes `/usr/bin/git --help`.
* `docker run <image> status` executes `/usr/bin/git status`.
