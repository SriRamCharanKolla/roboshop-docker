# ==============================================================================
# LEGACY / UNOPTIMIZED BUILD (FOR COMPARISON & REFERENCE)
# ==============================================================================
# Issues with unoptimized approach:
# 1. Used heavy Debian/Ubuntu base image (~1GB+).
# 2. Runs as default root user (security risk).
# 3. Build dependencies, package managers, and caches remain in the final image.
#
# FROM node:20
# # this creates /app and move there
# WORKDIR /app
# COPY package.json .
# COPY *.js .
# ENV MONGO="true" \
#     REDIS_URL="redis://redis:6379" \
#     MONGO_URL="mongodb://mongodb:27017/users"
# EXPOSE 8080
# RUN npm install
# CMD ["node", "server.js"]

# ==============================================================================
# STAGE 1: BUILD STAGE (Dependency Resolution & Compilation)
# ==============================================================================
# - Uses lightweight Alpine Linux base (~170MB vs ~1GB standard image).
# - Named 'builder' so compiled artifacts/modules can be referenced in Stage 2.
# - Temporary build tools and npm caches will NOT be included in the final runtime image.
FROM node:20.20.2-alpine3.23 AS builder

# Set working directory inside the build container
WORKDIR /app

# Copy package.json first to leverage Docker layer caching (npm install runs only if package.json changes)
COPY package.json .

# Copy application source code files into the builder container
COPY *.js .

# Install production dependencies (saved to /app/node_modules inside builder)
RUN npm install


# ==============================================================================
# STAGE 2: PRODUCTION / RUNTIME STAGE (Final Minimal & Secure Image)
# ==============================================================================
# - Fresh, clean Alpine container with zero build-time bloat.
FROM node:20.20.2-alpine3.23

# Set runtime working directory
WORKDIR /app

# Expose port 8080 to document that User service listens on this port
EXPOSE 8080

# Multi-stage copy:
# - '--from=builder': Copies files directly from Stage 1 ('builder').
# - First '/app': Source directory path inside the 'builder' container (includes code + node_modules).
# - Second '/app': Destination directory path in this final runtime container.
# Why? Only the necessary runtime files are carried over, keeping the image tiny and fast to download.
COPY --from=builder /app /app

# Note on Environment Variables:
# In Kubernetes, environment variables (MONGO, MONGO_URL, REDIS_URL) are decoupled and injected
# via ConfigMaps (e.g. envFrom: - configMapRef: name: user) rather than hardcoded in the Docker image.
# ENV MONGO="true" \
#     REDIS_URL="redis://redis:6379" \
#     MONGO_URL="mongodb://mongodb:27017/users"

# Security Best Practice:
# 1. Create a system group (-S) and system user (-S) named 'roboshop'
# 2. Add 'roboshop' user to 'roboshop' group (-G roboshop)
RUN addgroup -S roboshop && adduser -S roboshop -G roboshop

# Grant ownership of /app directory and all copied files to the non-root user 'roboshop'
RUN chown -R roboshop:roboshop /app

# Switch from root to non-root user 'roboshop' to prevent container breakout and privilege escalation
USER roboshop

# Default argument passed to ENTRYPOINT
CMD ["server.js"]

# Main binary executable to start the application
ENTRYPOINT [ "node" ]

# Note: ENTRYPOINT ("node") + CMD ("server.js") evaluates to: "node server.js"
# Using ENTRYPOINT allows overriding command arguments via CLI if needed.