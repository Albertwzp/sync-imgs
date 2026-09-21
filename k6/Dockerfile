# =========================
# Stage 1: Build k6
# =========================
FROM grafana/xk6:latest AS builder

RUN xk6 build latest \
    --with github.com/grafana/xk6-kubernetes \
    --with github.com/grafana/xk6-exec \
    --with github.com/grafana/xk6-dns

# =========================
# Stage 2: Runtime
# =========================
FROM alpine:3.24.1@sha256:28bd5fe8b56d1bd048e5babf5b10710ebe0bae67db86916198a6eec434943f8b as release

USER root

COPY --from=builder /xk6/k6 /usr/local/bin/k6

RUN apk add --no-cache ca-certificates curl \
    && curl -fsSLo /usr/local/bin/kubectl \
      "https://dl.k8s.io/release/$(curl -fsSL https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" \
    && chmod +x /usr/local/bin/kubectl \
    && curl -fsSLo /usr/local/bin/jq \
      "https://github.com/jqlang/jq/releases/latest/download/jq-linux-amd64" \
    && chmod +x /usr/local/bin/jq \
    && curl -fsSLo /usr/local/bin/yq \
      "https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64" \
    && chmod +x /usr/local/bin/yq \
    && kubectl version --client \
    && jq --version \
    && yq --version
RUN adduser -D -u 12345 -g 12345 k6
RUN chmod +x /usr/local/bin/k6

USER 12345
WORKDIR /home/k6

ENTRYPOINT ["k6"]