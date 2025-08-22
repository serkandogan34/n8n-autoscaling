# ---- builder ----
FROM python:3.9-slim AS builder

RUN apt-get update && apt-get install -y --no-install-recommends \
    curl jq docker.io && \
    mkdir -p ~/.docker/cli-plugins/ && \
    curl -SL https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 \
      -o ~/.docker/cli-plugins/docker-compose && \
    chmod +x ~/.docker/cli-plugins/docker-compose && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY autoscaler/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# ---- final ----
FROM python:3.9-slim
WORKDIR /app

# Final stage'de docker-cli ve compose plugin’i kur
RUN apt-get update && apt-get install -y --no-install-recommends docker.io curl && \
    mkdir -p /usr/local/lib/docker/cli-plugins && \
    curl -SL https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 \
      -o /usr/local/lib/docker/cli-plugins/docker-compose && \
    chmod +x /usr/local/lib/docker/cli-plugins/docker-compose && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Python bağımlılıklarını builder'dan al
COPY --from=builder /usr/local/lib/python3.9/site-packages /usr/local/lib/python3.9/site-packages

# Uygulama dosyaları
COPY autoscaler/autoscaler.py .
COPY docker-compose.yml .
# (Dockerfile’ı kopyalamanız şart değil; istiyorsanız bırakın, zararı yok.)

ENV PATH="/usr/local/lib/docker/cli-plugins:${PATH}"

CMD ["python", "-u", "autoscaler.py"]
