# ==========================
# Build Stage
# ==========================
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build-env

# Install only what’s needed for the build
RUN dnf --setopt=install_weak_deps=False install -q -y \
    maven \
    java-21-amazon-corretto-headless \
    which \
    tar \
    gzip \
    && \
    dnf clean all

ARG JAR_PATH

VOLUME /tmp
WORKDIR /

# Copy Maven wrapper and config
COPY .mvn .mvn
COPY mvnw .
RUN chmod +x mvnw   # ✅ make mvnw executable
COPY pom.xml .

# Pre-fetch dependencies for better caching
RUN ./mvnw dependency:go-offline -B -q

# Copy source and build JAR
COPY ./src ./src
RUN ./mvnw -DskipTests package -q && \
    mv /target/orders-0.0.1-SNAPSHOT.jar /app.jar

# ==========================
# Package Stage
# ==========================
FROM public.ecr.aws/amazonlinux/amazonlinux:2023

# Install only runtime dependencies
RUN dnf --setopt=install_weak_deps=False install -q -y \
    java-21-amazon-corretto-headless \
    shadow-utils \
    && \
    dnf clean all

# Use curl-full to enable "telnet://" scheme
RUN dnf -q -y swap libcurl-minimal libcurl-full \
    && dnf -q -y swap curl-minimal curl-full

# Create non-root user
ENV APPUSER=appuser
ENV APPUID=1000
ENV APPGID=1000

RUN useradd \
    --home "/app" \
    --create-home \
    --user-group \
    --uid "$APPUID" \
    "$APPUSER"

ENV JAVA_TOOL_OPTIONS=
ENV SPRING_PROFILES_ACTIVE=prod

WORKDIR /app
USER appuser

# Copy the app JAR from build stage
COPY --chown=appuser:appuser --from=build-env /app.jar .

EXPOSE 8080

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar /app/app.jar"]
