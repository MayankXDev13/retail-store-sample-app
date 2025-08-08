# =========================
# Build Stage
# =========================
FROM public.ecr.aws/amazonlinux/amazonlinux:2023 AS build-env

# Install required packages for build
RUN dnf --setopt=install_weak_deps=False install -q -y \
    maven \
    java-21-amazon-corretto-headless \
    which \
    tar \
    gzip \
    && \
    dnf clean all

VOLUME /tmp
WORKDIR /

# Copy Maven wrapper files
COPY .mvn .mvn
COPY mvnw .
RUN chmod +x mvnw   # Ensure mvnw is executable

# Copy project metadata
COPY pom.xml .

# Download dependencies offline
RUN ./mvnw dependency:go-offline -B -q

# Copy project source code
COPY ./src ./src

# Build application JAR
RUN ./mvnw -DskipTests package -q && \
    mv /target/ui-0.0.1-SNAPSHOT.jar /app.jar

# =========================
# Package Stage
# =========================
FROM public.ecr.aws/amazonlinux/amazonlinux:2023

# Install runtime dependencies
RUN dnf --setopt=install_weak_deps=False install -q -y \
    java-21-amazon-corretto-headless \
    shadow-utils \
    && \
    dnf clean all

# Replace minimal curl with full curl for telnet support
RUN dnf -q -y swap libcurl-minimal libcurl-full \
    && dnf -q -y swap curl-minimal curl-full

# Create application user
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

# Copy license files and application JAR from build stage
COPY ./ATTRIBUTION.md ./LICENSES.md
COPY --chown=appuser:appuser --from=build-env /app.jar .

# Expose application port
EXPOSE 8080

# Run application
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar /app/app.jar"]
