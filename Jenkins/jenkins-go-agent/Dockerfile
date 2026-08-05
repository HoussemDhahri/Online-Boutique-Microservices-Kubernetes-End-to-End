FROM golang:1.24.5

USER root

# ==========================================
# Install common dependencies + Java
# ==========================================
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        git \
        curl \
        unzip \
        ca-certificates \
        gnupg \
        openjdk-17-jdk && \
    rm -rf /var/lib/apt/lists/*

# ==========================================
# Environment
# ==========================================
ENV JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
ENV PATH="${JAVA_HOME}/bin:/usr/local/go/bin:/go/bin:/usr/local/bin:${PATH}"
ENV CGO_ENABLED=0

# Go
ENV GOPATH=/go
ENV GOMODCACHE=/go/pkg/mod
ENV GOCACHE=/go/cache

# Home
ENV HOME=/go

# Trivy
ENV TRIVY_CACHE_DIR=/go/.cache/trivy

# ==========================================
# Prepare directories
# ==========================================
RUN mkdir -p \
    ${GOMODCACHE} \
    ${GOCACHE} \
    ${TRIVY_CACHE_DIR} \
    /go/bin

# ==========================================
# Install Docker CLI
# ==========================================
RUN curl -fsSL \
    https://download.docker.com/linux/static/stable/x86_64/docker-27.3.1.tgz \
    -o docker.tgz && \
    tar xzf docker.tgz \
        --strip-components=1 \
        -C /usr/local/bin docker/docker && \
    rm docker.tgz

# ==========================================
# Install Trivy
# ==========================================
RUN curl -L \
    https://github.com/aquasecurity/trivy/releases/download/v0.72.0/trivy_0.72.0_Linux-64bit.tar.gz \
    -o trivy.tar.gz && \
    tar -xzf trivy.tar.gz trivy && \
    mv trivy /usr/local/bin/trivy && \
    chmod +x /usr/local/bin/trivy && \
    rm trivy.tar.gz

# ==========================================
# Install kubectl
# ==========================================
RUN curl -LO \
    https://dl.k8s.io/release/v1.35.1/bin/linux/amd64/kubectl && \
    install \
        -o root \
        -g root \
        -m 0755 \
        kubectl \
        /usr/local/bin/kubectl && \
    rm kubectl

# ==========================================
# Install kustomize
# ==========================================
RUN curl -L \
    https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2Fv5.7.1/kustomize_v5.7.1_linux_amd64.tar.gz \
    -o kustomize.tar.gz && \
    tar -xzf kustomize.tar.gz && \
    mv kustomize /usr/local/bin/kustomize && \
    chmod +x /usr/local/bin/kustomize && \
    rm kustomize.tar.gz

# ==========================================
# Install Go tools
# ==========================================
RUN go install golang.org/x/tools/go/analysis/passes/shadow/cmd/shadow@v0.38.0 && \
    curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | \
    sh -s -- -b /usr/local/bin v2.5.0

# ==========================================
# Fix permissions for ANY uid:gid
# ==========================================
RUN chmod -R a+rwX /go

# ==========================================
# Install SonarScanner CLI
# ==========================================
RUN curl -L \
    https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-7.1.0.4889-linux-x64.zip \
    -o sonar-scanner.zip && \
    unzip -q sonar-scanner.zip -d /opt && \
    mv /opt/sonar-scanner-* /opt/sonar-scanner && \
    ln -s /opt/sonar-scanner/bin/sonar-scanner /usr/local/bin/sonar-scanner && \
    rm sonar-scanner.zip

# ==========================================
# Validation
# ==========================================
RUN java -version && \
    go version && \
    git --version && \
    docker --version && \
    trivy --version && \
    kubectl version --client && \
    kustomize version && \
    sonar-scanner --version && \
    golangci-lint --version

# ==========================================
# Runtime
# ==========================================
WORKDIR /go