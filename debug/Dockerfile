# Base Operating System Image (Enterprise Linux compatible)
FROM almalinux:9

# Install essential network troubleshooting and debugging utilities:
# - curl / wget  : HTTP/REST API endpoint testing and payload downloading
# - vim          : Inspecting and editing configuration files inside the container
# - net-tools    : Classic networking commands (netstat, ifconfig, arp, route)
# - iputils      : Network reachability tests (ping, tracepath, arping)
# - telnet       : TCP port connectivity and handshake testing
RUN dnf install -y curl wget vim net-tools iputils telnet

# Keep the debug container running in the background so you can attach via 'kubectl exec' / 'docker exec'
CMD ["sleep", "20000"]