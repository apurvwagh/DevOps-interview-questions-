1. What are CMD and ENTRYPOINT? What is the difference?

1) Both CMD and ENTRYPOINT define what runs when a Docker container starts, but they have different purposes. ENTRYPOINT defines the main executable of the container, while CMD provides default arguments or a default command.

2) The important difference is how they behave when we provide a command while running the container. CMD can be completely overridden, whereas ENTRYPOINT normally remains the executable and the supplied command becomes its arguments.

3) For production images, I generally use ENTRYPOINT when the container represents a specific executable or application, and CMD for default arguments that users may want to override.

I prefer the exec form, such as ENTRYPOINT ["python", "app.py"], rather than shell form, because it provides better signal handling and process behavior.”

ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8080"]

ENTRYPOINT defines what the container is; CMD defines the default command or arguments.”

===================================================================

2. Explain Docker Multi-stage Builds.

1) A Docker multi-stage build allows me to use multiple stages in a single Dockerfile. I can use one stage to compile or build the application and a separate final stage containing only the runtime dependencies and compiled artifact.

2) The major advantage is that build tools, source code, package managers, and temporary files don’t need to be included in the final production image. This significantly reduces image size and attack surface.

3) For example, for a Go application, I can use a Go builder image to compile the binary and then copy only the binary into a minimal runtime image. The final image doesn’t contain the Go compiler or source code.

4) The same approach works for Java, Node.js, Python applications with appropriate dependency handling, and frontend applications where we build static assets in one stage and serve them from Nginx in another.”

# Stage 1 - Build
FROM golang:1.24 AS builder

WORKDIR /app
COPY . .

RUN go build -o myapp .


# Stage 2 - Runtime
FROM alpine:latest

COPY --from=builder /app/myapp /myapp

ENTRYPOINT ["/myapp"]

Benefits

* Smaller image
* Faster image pull
* Faster Pod startup
* Reduced attack surface
* Fewer unnecessary packages
* Cleaner production image

Strong interview line:

“I separate build-time dependencies from runtime dependencies so the production image contains only what the application actually needs.”

===================================================================

3. What are Dockerfile best practices?

1) My Dockerfile best practices focus on reducing image size, improving build performance, security, and reproducibility.

2) First, I use a minimal and appropriate base image and pin important versions instead of blindly using latest. I use multi-stage builds to remove build-time dependencies from the final image.

3) I also structure the Dockerfile to maximize layer caching. For example, I copy dependency files and install dependencies before copying frequently changing application source code.

4) I use .dockerignore to prevent unnecessary files such as .git, local build artifacts, logs, and temporary files from being sent to the Docker build context.

5) From a security perspective, I avoid running the application as root, don’t put secrets or credentials in the Dockerfile, scan images for vulnerabilities, and keep base images updated.

6)Finally, I use exec-form ENTRYPOINT/CMD, define appropriate health checks where useful, and keep one main responsibility per container.”

Dockerfile
   │
   ├── Minimal base image
   
   ├── Multi-stage build
   
   ├── .dockerignore
   
   ├── Layer caching
   
   ├── Non-root user
   
   ├── No secrets
   
   ├── Pin versions
   
   ├── Vulnerability scanning
   
   ├── Exec-form ENTRYPOINT
   
   └── Small runtime image

   My goal is to make the image small, reproducible, secure, and optimized for Docker layer caching.”

   ===================================================================
  
4. Your Docker image is very large, builds are slow, and Pods take a long time to start. How would you optimize it?

1) I would treat this as three related problems: image size, build time, and container startup/pull time. First, I would inspect the image layers using tools such as docker history and image scanning tools to identify which layers are consuming the most space.

2) Then I would use a smaller base image where appropriate and implement multi-stage builds so compilers, source code, build tools, and unnecessary dependencies don’t enter the final image. I would also use .dockerignore to reduce the build context.

3) For build performance, I would optimize Docker layer ordering so stable dependency installation happens before frequently changing source code. This allows Docker or BuildKit to reuse cached layers. I would also use BuildKit cache mounts and registry-backed build cache in CI/CD where appropriate.

4) For slow Kubernetes Pod startup, I would reduce the final image size because Kubernetes nodes need to pull the image before the container can start. I would also use a reliable container registry close to the cluster, such as ECR for EKS, and avoid unnecessary image pulls by using appropriate image pull policies and node/image caching strategies.

5) Finally, I would measure the improvement rather than assuming it worked: image size, build duration, image-pull duration, container startup time, and deployment time should all be monitored.”

Large Image
     ↓
docker history / image analysis
     ↓
Remove unnecessary layers
     ↓
Multi-stage Build
     ↓
Minimal Base Image
     ↓
.dockerignore
     ↓
Optimize Layer Cache
     ↓
BuildKit Cache
     ↓
Smaller Image
     ↓
Faster Pull
     ↓
Faster Pod Startup


===================================================================











   
