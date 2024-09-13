# Docker Image Layers, Container Execution, and Temporary Modifications: A Practical Guide

Docker’s ability to efficiently build, reuse, and layer images plays a crucial role in modern software development. In this guide, we’ll walk through the concept of Docker image layers, container execution, and the temporary nature of changes within containers. The process showcased here uses Dockerfiles to build images, run containers, and modify file systems inside running containers — all while demonstrating how Docker handles changes and user management inside containers.

[Checkout the GitHub Repo](https://github.com/sharmarvellous/testing)

---

## Step 1: Understanding the Dockerfile Creation Process

The Dockerfile (`dockerfile6`) contains the instructions used to create a Docker image. In this case, the image uses Ubuntu as the base, installs Apache2, fetches web content from a GitHub repository, and copies web files into Apache's default directory.

### Key Steps in `dockerfile6`:

- **Base Image**:
  - `FROM ubuntu`: Sets Ubuntu as the base image for the Docker container.

- **Installing Apache and Unzip**:
  - `RUN apt update && apt install apache2 unzip -y`: Updates the package manager and installs Apache2 (a web server) and Unzip (to handle zipped files).

- **Fetching External Files**:
  - `ADD https://github.com/sharmarvellous/testing/archive/refs/heads/main.zip /var/www/html/code.zip`: Downloads a ZIP file from GitHub and places it into the `/var/www/html/` directory of the container.

- **Unzipping and Moving Files**:
  - `RUN unzip code.zip && mv /var/www/html/testing-main/* /var/www/html/`: Unzips the downloaded file and moves the extracted content to Apache’s default directory, making the files available for the web server.

- **Starting Apache**:
  - `CMD ["apache2ctl", "-D", "FOREGROUND"]`: This command starts Apache in the foreground, ensuring that the container continues running after starting the web server.

---

## Step 2: Building the Docker Image

After defining the Dockerfile, the next step is to build the Docker image using the following command:

```bash
docker build -t testing:v0 -f dockerfile6 .
```

**Explanation:**

*docker build:* This command builds a Docker image by reading the instructions in the Dockerfile.

`-t testing:v0`: Tags the image as testing:v0.

`-f dockerfile6`: Specifies that the image should be built using dockerfile6.

**Layering and Caching:**

- Docker creates a new layer for each instruction in the Dockerfile.

- Cached layers are reused if no changes have been made, speeding up the build process.

---

## Step 3: Running the Docker Container

Once the image is built, you can run it as a container:
```
docker run -d -p 9090:80 --name testc testing:v0
```

**Explanation:**
- `docker run`: This command starts a new container.
- `-d`: Runs the container in detached mode (in the background).
- `-p 9090:80`: Maps port 9090 on your local machine to port 80 inside the container, where Apache is running.
- `--name testc`: Assigns the container a specific name (testc) for easier identification.
- `testing:v0`: Specifies the image to use for the container.

**Outcome:**

The container starts running in the background, and Apache is serving the content fetched from GitHub. You can access the website at http://localhost:9090.

---

## Step 4: Inspecting and Modifying the Container’s File System

After the container is running, you can access its file system to inspect or make temporary changes:

# Inspecting the Container:

```
docker exec -it testc bash
```

# Explanation:

`docker exec -it testc bash`: Opens an interactive bash shell inside the running container, allowing you to inspect and modify the file system.

# Temporary Changes in the Container:

Once inside the container, you can list the files in the `/var/www/html/` directory using:

```ls /var/www/html/```

The output shows the files that were extracted from the ZIP file and placed into Apache’s directory:

```README.md index.html scripts.js styles.css testing-main```

If you modify or remove any files inside the container, these changes are temporary. Docker containers have a read-write layer on top of the image’s read-only layers. Changes made inside a running container will not affect the underlying image and will be lost once the container is stopped or removed.

---

## Step 5: Key Concepts — Image Layers and Temporary Modifications

1. **Layered Image Structure**: Docker Image Layers: Each command in the Dockerfile adds a new layer to the Docker image. These layers are cached, which allows Docker to reuse them during subsequent builds if no changes have occurred in those steps.

2. **Efficiency**: Docker’s layering mechanism makes images lightweight and efficient, as unchanged layers can be reused across multiple builds, avoiding unnecessary reprocessing.

3. **Temporary Nature of Containers**: Read-Write Layer: When a Docker container runs, it adds a read-write layer on top of the image’s read-only layers. This allows you to modify files and make changes within the container without altering the underlying image.

4. **Temporary Changes**: Any changes made in the container (such as unzipping files, modifying content, or adding new files) only persist for the life of the container. Once the container is stopped or deleted, all changes are lost.
