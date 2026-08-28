# ==============================================================================
# LEGACY / UNOPTIMIZED BUILD (FOR COMPARISON & REFERENCE)
# ==============================================================================
# Issues with unoptimized approach:
# 1. Used full Maven SDK image (~800MB - 1GB+) containing compiler tools in production.
# 2. Runs as default root user (security risk).
# 3. Maven local repository cache (~/.m2) and build artifacts retained in the final image.
#
# FROM maven:3.9.14
# EXPOSE 8080
# WORKDIR /app
# COPY pom.xml .
# COPY src ./src
# RUN mvn clean package
# RUN mv target/shipping-1.0.jar shipping.jar
# ENV CART_ENDPOINT="cart:8080" \
#     DB_HOST="mysql"
# CMD ["java", "-jar", "shipping.jar"]

# ==============================================================================
# STAGE 1: BUILD STAGE (Maven Compilation & JAR Packaging)
# ==============================================================================
# - Uses lightweight Alpine-based Maven with OpenJDK 8 (project requirement).
# - Named 'builder' so generated JAR can be copied to the final runtime stage.
# - Maven dependencies and build cache (~/.m2) will be discarded after this stage.
FROM maven:3.9-eclipse-temurin-8-alpine AS builder

# Set working directory inside the build container
WORKDIR /app

# Copy Maven pom.xml first to take advantage of Docker layer caching
COPY pom.xml .

# Download dependencies (cached unless pom.xml changes)
RUN mvn dependency:go-offline -B || true

# Copy application Java source code files
COPY src ./src

# Compile and package Spring Boot application into executable JAR
RUN mvn clean package -DskipTests && \
    mv target/shipping-1.0.jar shipping.jar


# ==============================================================================
# STAGE 2: PRODUCTION / RUNTIME STAGE (Minimal JRE Alpine Container)
# ==============================================================================
# - Fresh, ultra-lightweight Alpine container containing ONLY Java 8 JRE (~80-100MB).
# - Zero build tools, zero Maven SDK, zero temporary caches.
FROM eclipse-temurin:8-jre-alpine

# Set runtime working directory
WORKDIR /app

# Expose port 8080 to document the port on which Shipping service listens
EXPOSE 8080

# Multi-stage copy:
# - '--from=builder': Copies ONLY the pre-built shipping.jar from Stage 1.
# - Leaves behind all Maven compiler dependencies, saving ~700MB+ in image size.
COPY --from=builder /app/shipping.jar /app/shipping.jar

# Note on Environment Variables:
# In Kubernetes, environment variables (CART_ENDPOINT, DB_HOST) are decoupled and injected
# dynamically via ConfigMaps (e.g., envFrom: - configMapRef: name: shipping) rather than hardcoded.
# ENV CART_ENDPOINT="cart:8080" \
#     DB_HOST="mysql"

# Security Best Practice:
# 1. Create a system group (-S) and system user (-S) named 'roboshop'
# 2. Add 'roboshop' user to 'roboshop' group (-G roboshop)
# 3. Grant ownership of /app directory and the JAR file to the non-root user 'roboshop'
RUN addgroup -S roboshop && adduser -S roboshop -G roboshop && \
    chown -R roboshop:roboshop /app

# Switch from root to non-root user 'roboshop' to prevent container breakout & privilege escalation
USER roboshop

# Default argument passed to the ENTRYPOINT (the JAR file name)
CMD ["shipping.jar"]

# Main binary executable run by the container
ENTRYPOINT ["java", "-jar"]

# Note: ENTRYPOINT ["java", "-jar"] + CMD ["shipping.jar"] evaluates to:
# "java -jar shipping.jar"
# This allows passing custom JVM memory options or arguments via CLI if needed.

