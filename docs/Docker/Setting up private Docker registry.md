## Docker Compose file for Docker Registry with Persistence and Domain

This docker-compose.yml file defines a service for running a Docker registry with persistence volume and a custom domain name.

**Please note:** You'll need to configure your DNS server to point `registry.ogear.xyz` to the IP address of your server.

YAML

```
version: '3.8'

services:
  registry:
    image: registry:2
    ports:
      - "5000:5000"
    volumes:
      - /data/registry/data:/var/lib/registry
      - /etc/letsencrypt/live/registry.ogear.xyz:/certs
      - /home/kj/.auth:/auth
    environment:
      - REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/var/lib/registry
      - REGISTRY_HTTP_ADDR=0.0.0.0:5000  # Allow access from any interface
      - VIRTUAL_HOST=registry.ogear.xyz  # Define your domain name
      - REGISTRY_HTTP_TLS_CERTIFICATE=/etc/letsencrypt/live/registry.ogear.xyz/fullchain.pem
      - REGISTRY_HTTP_TLS_KEY=/etc/letsencrypt/live/registry.ogear.xyz/privkey.pem
      - REGISTRY_AUTH=htpasswd
      - REGISTRY_AUTH_HTPASSWD_PATH=/home/kj/.auth/htpasswd
      - REGISTRY_AUTH_HTPASSWD_REALM="Registry Realm"
    restart: unless-stopped

volumes:
  registry_data: {}
```

**Explanation:**

- **version:** Specifies the Docker Compose file format version.
- **services:** Defines a service named `registry`.
- **image:** Specifies the Docker registry image to use (registry:2).
- **ports:** Maps the container port 5000 to the host port 5000. This allows access to the registry from your network.
- **volumes:**
    - Mounts a named volume named `registry_data` to the `/var/lib/registry` directory within the container. This directory stores the registry data persistently.
- **environment:** Sets environment variables for the registry:
    - `REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY`: Defines the directory where the registry will store data (matches volume mount).
    - `REGISTRY_HTTP_ADDR`: Allows access from any interface within the container.
    - `VIRTUAL_HOST`: Sets the domain name for the registry (registry.ogear.xyz).
- **restart:** Specifies that the registry container should be restarted automatically if it stops.
- **volumes:** Defines an unnamed volume named `registry_data`. This volume will persist the registry data even if the container is recreated.

**How to use this file:**

1. Save the file as `docker-compose.yml` on your server.
2. Create the directory for the volume data if it doesn't exist: `sudo mkdir -p /path/to/registry_data` (replace with your desired location).
3. Run `docker-compose up -d` to build the image and start the registry container.

**Additional Notes:**

- You can modify the port mapping, environment variables, and volume configuration to suit your needs.
- Consider using a reverse proxy like Nginx with SSL/TLS encryption for production environments.
- Refer to the official Docker Registry documentation for more advanced configuration options: [https://docs.docker.com/](https://docs.docker.com/)