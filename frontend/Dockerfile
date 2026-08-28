# ==============================================================================
# LEGACY / UNOPTIMIZED BUILD (FOR COMPARISON & REFERENCE)
# ==============================================================================
# Issues with unoptimized approach:
# 1. Used heavy Debian base image (~140MB+).
# 2. Runs as root user (Nginx master process runs with root privileges).
# 3. Security vulnerabilities from default root container execution.
#
# FROM nginx:1.24.0
# RUN rm -f /usr/share/nginx/html/index.html
# RUN rm -f /etc/nginx/nginx.conf
# RUN rm -f /etc/nginx/conf.d/default.conf
# COPY nginx.conf /etc/nginx/nginx.conf
# COPY static /usr/share/nginx/html/

# ==============================================================================
# PRODUCTION RUNTIME: HARDENED NON-ROOT NGINX ON ALPINE LINUX
# ==============================================================================
# - Uses ultra-lightweight Alpine Linux (~20-40MB).
# - Configured specifically to run as non-root user 'nginx' (UID/GID 101).
FROM nginx:1.24.0-alpine3.17

# Remove default unconfigured Nginx boilerplate configuration files
RUN rm /etc/nginx/nginx.conf /etc/nginx/conf.d/default.conf

# Security & Permissions Configuration for Non-Root Nginx:
# Nginx needs write permissions to temporary cache folders, logs, and configuration directories
# when running as non-root 'nginx' user:
RUN mkdir -p /var/cache/nginx/client_temp && \
    mkdir -p /var/cache/nginx/proxy_temp && \
    mkdir -p /var/cache/nginx/fastcgi_temp && \
    mkdir -p /var/cache/nginx/uwsgi_temp && \
    mkdir -p /var/cache/nginx/scgi_temp && \
    chown -R nginx:nginx /var/cache/nginx && \
    chown -R nginx:nginx /etc/nginx/ && \
    chmod -R 755 /etc/nginx/ && \
    chown -R nginx:nginx /var/log/nginx

# SSL Certificate Directory Setup:
# Grants permissions for potential SSL/TLS certs mounting
RUN mkdir -p /etc/nginx/ssl/ && \
    chown -R nginx:nginx /etc/nginx/ssl/ && \
    chmod -R 755 /etc/nginx/ssl/

# Nginx Process ID (PID) File Permissions:
# By default, Nginx writes its PID file to /var/run/nginx.pid.
# When running as non-root, the file must be pre-created and owned by 'nginx' user.
RUN touch /var/run/nginx.pid && \
    chown -R nginx:nginx /var/run/nginx.pid /run/nginx.pid

# Copy custom reverse-proxy configuration and static HTML/CSS/JS frontend assets
COPY nginx.conf /etc/nginx/nginx.conf
COPY static /usr/share/nginx/html/

# Switch from root to non-root system user 'nginx' to prevent privilege escalation attacks
USER nginx

# Start Nginx web server in foreground (daemon off) so container stays alive
CMD ["nginx", "-g", "daemon off;"]