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
  












   
