# ==============================================================================
# LEGACY / UNOPTIMIZED BUILD (FOR COMPARISON & REFERENCE)
# ==============================================================================
# Issues with unoptimized approach:
# 1. Used heavy Debian base image python:3.9 (~1GB+).
# 2. Runs as default root user (security vulnerability).
# 3. Compiler build tools and pip wheels remained in the production layer.
#
# FROM python:3.9 
# EXPOSE 8080
# WORKDIR /app
# COPY payment.ini .
# COPY *.py .
# COPY requirements.txt .
# RUN pip3 install -r requirements.txt
# CMD ["uwsgi", "--ini", "payment.ini"]

# ==============================================================================
# STAGE 1: BUILD STAGE (Compiling C Extensions & Python Wheels for uWSGI)
# ==============================================================================
# - Uses lightweight Alpine Linux base.
# - Installs temporary native C build tools (gcc, make, headers) needed to compile uWSGI.
# - Installs Python packages into an isolated target folder (/install) using --prefix.
FROM python:3.9.25-alpine3.22 AS builder

# Set working directory inside build container
WORKDIR /app

# Install native compilation dependencies required to build uWSGI C-extensions:
# - build-base    : Meta-package containing gcc, g++, make, libc-dev
# - linux-headers : Kernel headers needed for POSIX socket interfaces
# - pcre-dev      : Perl-compatible regular expression development headers for uWSGI routing
RUN apk add --no-cache build-base linux-headers pcre-dev

# Copy dependency manifest first to leverage Docker layer caching
COPY requirements.txt .

# Install Python dependencies into isolated '/install' prefix directory:
# This ensures binaries and site-packages can be cleanly copied to the runtime stage
RUN pip3 install --prefix=/install -r requirements.txt


# ==============================================================================
# STAGE 2: PRODUCTION / RUNTIME STAGE (Minimal & Secure Image)
# ==============================================================================
# - Clean Alpine container with NO compilers, headers, or build tools.
FROM python:3.9.25-alpine3.22

# Expose port 8080 to document the port on which Payment service listens
EXPOSE 8080

# Set runtime working directory
WORKDIR /app

# Install only runtime C-libraries (without dev headers/compilers):
# - pcre : Shared library needed by uWSGI at runtime for regex processing
RUN apk add --no-cache pcre

# Security Best Practice:
# 1. Create a system group (-S) and system user (-S) named 'roboshop'
# 2. Add 'roboshop' user to 'roboshop' group (-G roboshop)
RUN addgroup -S roboshop && adduser -S roboshop -G roboshop

# Grant ownership of /app directory to non-root user
RUN chown -R roboshop:roboshop /app

# Multi-stage copy:
# Copy pre-compiled Python binaries and site-packages from builder stage into system path
COPY --from=builder /install /usr/local

# Copy application source code and uWSGI configuration with roboshop file ownership
COPY --chown=roboshop:roboshop payment.ini *.py requirements.txt .

# Switch from root to non-root user 'roboshop' to prevent container breakout
USER roboshop

# Start Payment microservice using uWSGI application server
CMD ["uwsgi", "--ini", "payment.ini"]