# Base Operating System Image (Enterprise Linux compatible)
FROM almalinux:9

# Install essential network troubleshooting and debugging utilities:
# - net-tools    : Classic networking commands (netstat, ifconfig, arp, route)
# - iputils      : Network reachability tests (ping, tracepath, arping)
# - telnet       : TCP port connectivity and handshake testing
# - bind-utils   : DNS lookup utilities (dig, nslookup, host)
RUN dnf install net-tools iputils telnet bind-utils -y

# Keep the debug container running in the background so you can attach via 'kubectl exec' / 'docker exec'
CMD ["sleep", "2000"]