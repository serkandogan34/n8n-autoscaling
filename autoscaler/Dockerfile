# Use an official Python runtime as a parent image
FROM python:3.9-slim as builder

# Install system dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    curl \
    jq \
    && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Set the working directory in the container
WORKDIR /app

# Install Docker CLI and Docker Compose v2
RUN apt-get update && \
    apt-get install -y --no-install-recommends docker.io && \
    mkdir -p ~/.docker/cli-plugins/ && \
    curl -SL https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose && \
    chmod +x ~/.docker/cli-plugins/docker-compose && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Copy requirements first to leverage Docker cache
COPY autoscaler/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Final stage
FROM python:3.9-slim
WORKDIR /app

# Copy from builder
# We'll use a more robust copy approach to find the docker binary
RUN DOCKER_BINARY_PATH=$(which docker) && cp "$DOCKER_BINARY_PATH" /usr/bin/docker
COPY --from=builder /usr/bin/docker /usr/bin/docker
COPY --from=builder /root/.docker/cli-plugins/ /usr/local/lib/docker/cli-plugins/
COPY --from=builder /usr/local/lib/python3.9/site-packages /usr/local/lib/python3.9/site-packages

# Ensure docker compose plugin is in PATH
ENV PATH="/usr/local/lib/docker/cli-plugins:${PATH}"

# Copy application files
COPY autoscaler/autoscaler.py .
COPY docker-compose.yml .
COPY Dockerfile .

# Specify the command to run on container start
CMD ["python", "-u", "autoscaler.py"]
