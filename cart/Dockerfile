# ==============================================================================
# LEGACY / UNOPTIMIZED BUILD (FOR COMPARISON & REFERENCE)
# ==============================================================================
# Issues with unoptimized approach:
# 1. Used heavy Debian/Ubuntu base image (~1GB+).
# 2. Runs as default root user (security risk).
# 3. Everything compiled and retained in a single layer.
#
# FROM node:20
# # this creates /app and move there
# WORKDIR /app
# COPY package.json .
# COPY *.js .
# RUN npm install
# ENV REDIS_HOST="redis" \
#     CATALOGUE_HOST="catalogue" \
#     CATALOGUE_PORT="8080"
# CMD ["node", "server.js"]

# ==============================================================================
# STAGE 1: BUILD STAGE (Dependency Resolution & Compilation)
# ==============================================================================
# - Uses lightweight Alpine Linux base (~170MB vs ~1GB standard image).
# - Named 'builder' so compiled artifacts/modules can be copied to subsequent stages.
# - Temporary build tools/cache will NOT be present in the final runtime image.
FROM node:20.20.2-alpine3.23 AS builder

# Set working directory inside the container for build tasks
WORKDIR /app

# Copy dependency definition file first to leverage Docker layer caching
COPY package.json .

# Copy application source code
COPY *.js .

# Install dependencies (packages will be saved to /app/node_modules)
RUN npm install


# ==============================================================================
# STAGE 2: PRODUCTION / RUNTIME STAGE (Final Minimal & Secure Image)
# ==============================================================================
# - Fresh, minimal Alpine container with zero build-time bloat/tools.
FROM node:20.20.2-alpine3.23

# Set runtime working directory
WORKDIR /app

# Expose port 8080 to document the port on which Cart service listens
EXPOSE 8080

# Multi-stage copy: Only copy built files and node_modules from the builder stage
# This keeps the final image clean, minimal, and fast to download/deploy
COPY --from=builder /app /app

# Configure application environment variables
# [NOTE / POTENTIAL ISSUE]: Cart typically requires REDIS_HOST & CATALOGUE_HOST instead of MONGO
ENV MONGO="true" \
    MONGO_URL="mongodb://mongodb:27017/catalogue"

# Security Best Practice:
# 1. Create a system group (-S) and system user (-S) named 'roboshop'
# 2. Add 'roboshop' user to 'roboshop' group (-G roboshop)
RUN addgroup -S roboshop && adduser -S roboshop -G roboshop

# Grant ownership of /app directory and its contents to the non-root user
# [NOTE / SYNTAX ISSUE]: Flag should be '-R' (capital R with hyphen) instead of 'R'
RUN chown R roboshop:roboshop /app

# Switch from root to non-root system user 'roboshop' to prevent container breakout/privilege escalation
USER roboshop

# Default argument passed to the ENTRYPOINT
CMD ["server.js"]

# Main binary executable run by the container
ENTRYPOINT [ "node" ]

# Note: ENTRYPOINT ("node") + CMD ("server.js") evaluates to: "node server.js"
# Having ENTRYPOINT allows overriding the script via CLI arguments if needed.