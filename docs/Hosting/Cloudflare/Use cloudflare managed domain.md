

# 1

Your `docker-compose.yml` file looks almost correct, but there are a few points to address:

### Key Updates:

1. **Mount the `certs` and `auth` directories**:
   The `certs` and `auth` directories need to be mounted so that Docker can access the TLS certificates and the `.htpasswd` file. Otherwise, the Docker service won't have access to them inside the container.

2. **Use the correct path for `registry_data` volume**:
   You've already specified a bind mount for the registry data (`/data/registry/data`), so you don't need to define the `volumes` section at the bottom for `registry_data`. You can simply remove it or update the reference.

Here’s the updated version:

### Updated `docker-compose.yml`:

```yaml
version: '3.8'

services:
  registry:
    image: registry:2
    ports:
      - "5000:5000"
    volumes:
      - /data/registry/data:/var/lib/registry          # Bind mount for registry data
      - ./certs:/etc/letsencrypt/live/registry.ogear.xyz  # Mount for Let's Encrypt certs
      - ./auth:/auth                                    # Mount for authentication files
    environment:
      - REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/var/lib/registry
      - REGISTRY_HTTP_ADDR=0.0.0.0:5000
      - VIRTUAL_HOST=registry.ogear.xyz
      - REGISTRY_HTTP_TLS_CERTIFICATE=/etc/letsencrypt/live/registry.ogear.xyz/fullchain.pem
      - REGISTRY_HTTP_TLS_KEY=/etc/letsencrypt/live/registry.ogear.xyz/privkey.pem
      - REGISTRY_AUTH=htpasswd
      - REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd
      - REGISTRY_AUTH_HTPASSWD_REALM="Registry Realm"
    restart: unless-stopped
```

### Steps to Ensure Everything is Working:

1. **Create the necessary directories:**
   Ensure you have the following directories on your host machine:
   - `/data/registry/data` (for registry storage)
   - `./certs` (for the TLS certificates)
   - `./auth` (for the `.htpasswd` file)

   The `certs` directory should contain the Let's Encrypt certificates (`fullchain.pem` and `privkey.pem`), and the `auth` directory should have your `.htpasswd` file for basic authentication.

2. **Run Docker Compose:**
   Start the registry service using the following command:

   ```bash
   docker-compose up -d
   ```

3. **Verify the Setup:**

   - **Registry Access**: Check if you can access your registry at `https://registry.ogear.xyz:5000`. If using `curl`, you might need to provide credentials.
   
   ```bash
   curl -u <username>:<password> https://registry.ogear.xyz:5000/v2/
   ```

   - **SSL Verification**: Verify that SSL is working by ensuring you can connect securely via HTTPS. If using a self-signed certificate, ensure your Docker clients trust the certificate.
   
   - **Authentication**: Test that basic authentication is required when pushing/pulling images.

4. **Certificate Renewal**: 
   Don’t forget to set up a cron job for auto-renewing your Let's Encrypt certificates as mentioned earlier:

   ```bash
   0 2 * * * /usr/bin/certbot renew --quiet --deploy-hook "docker-compose restart registry"
   ```

This updated configuration should now work as expected with Let's Encrypt TLS certificates and basic authentication.

Let me know if you need further help!


# 2


If you're using Cloudflare as your domain's DNS provider, you have several options for securing your Docker registry with TLS. You can either:

  
1. **Use Cloudflare’s SSL/TLS for Your Domain** (easier, less configuration)
2. **Use Let's Encrypt SSL Certificates** (automatic certificate management)

### Option 1: Using Cloudflare's SSL/TLS (Full/Strict Mode)

Cloudflare offers free SSL certificates for your domain, and you can use their flexible or full SSL mode to secure your Docker registry. In this case, you **don't need to generate certificates** yourself; Cloudflare handles the SSL certificates for you.
 

Here's how to configure it:


1. **Ensure SSL is enabled in Cloudflare:**
- Go to your Cloudflare dashboard.
- Select your domain (`registry.ogear.xyz`).
- Go to the **SSL/TLS** tab.
- Choose either **Full** or **Full (Strict)** mode for end-to-end encryption. (Strict requires you to install a valid SSL certificate on your origin server, while Full allows for self-signed certificates.)
  

2. **Bypass Cloudflare Proxy (optional):**
- In case you don't want to proxy through Cloudflare (use Cloudflare DNS but handle your own certificates), you can set the DNS record for `registry.ogear.xyz` to **DNS only** (the gray cloud icon).

3. **Cloudflare Origin Certificate (optional):**
If you want more security on the origin server side, you can use Cloudflare's **Origin CA Certificate** instead of self-signed or Let's Encrypt certificates. This certificate is issued by Cloudflare and is trusted by their proxy but not by browsers.

  

To do this:

- In the Cloudflare dashboard, go to **SSL/TLS** > **Origin Certificates**.
- Click on **Create Certificate**.
- Download the **origin certificate** and **private key**.
- Update your Docker Compose file to point to these files for the TLS configuration.

  

```yaml

environment:

- REGISTRY_HTTP_TLS_CERTIFICATE=/certs/cloudflare-origin.crt # Cloudflare Origin Cert

- REGISTRY_HTTP_TLS_KEY=/certs/cloudflare-origin.key # Cloudflare Private Key

```

  

### Option 2: Using Let's Encrypt with Docker (via Certbot)

  

If you prefer to handle the certificate issuance yourself, you can use **Let's Encrypt** to generate free, trusted certificates. Let's Encrypt can be automated with tools like **Certbot**.
 

Here’s how to set it up:
  
1. **Install Certbot**:
  
On your Ubuntu server, install Certbot and the necessary dependencies:
 

```bash

sudo apt update

sudo apt install certbot python3-certbot-dns-cloudflare

```

  

2. **Generate a Cloudflare API Token**:

  

To automate certificate generation with Let's Encrypt, you'll need a Cloudflare API token:
  

- Log in to Cloudflare.
- Go to **My Profile** > **API Tokens** > **Create Token**.
- Select the **Zone: DNS Edit** template.
- Grant access to the specific zone (your domain) and permissions.
- Copy the API token.

  

3. **Configure Certbot for Cloudflare**:
  

- Create a configuration file for Certbot with the API token:
  
```bash

sudo mkdir -p /etc/letsencrypt

sudo nano /etc/letsencrypt/cloudflare.ini

```

  

Add the following content to the `cloudflare.ini` file:
  
```ini

dns_cloudflare_api_token = <YOUR_CLOUDFLARE_API_TOKEN>

```

  

Make sure to protect the file with the appropriate permissions:
  
```bash

sudo chmod 600 /etc/letsencrypt/cloudflare.ini

```

  

4. **Obtain the Let's Encrypt Certificate**:


Use Certbot to issue a certificate for your domain:

```bash

sudo certbot certonly \

--dns-cloudflare \

--dns-cloudflare-credentials /etc/letsencrypt/cloudflare.ini \

-d registry.ogear.xyz

```

  

This will create the certificate and private key under `/etc/letsencrypt/live/registry.ogear.xyz/`.

  

5. **Update Docker Compose to Use Let's Encrypt Certificate**:

Once the certificate is generated, update your `docker-compose.yml` file to point to the Let's Encrypt certificate and private key:

  

```yaml

environment:

- REGISTRY_HTTP_TLS_CERTIFICATE=/etc/letsencrypt/live/registry.ogear.xyz/fullchain.pem

- REGISTRY_HTTP_TLS_KEY=/etc/letsencrypt/live/registry.ogear.xyz/privkey.pem

```

  

6. **Set Up Auto-Renewal**:
Let's Encrypt certificates are valid for 90 days. You can set up automatic renewal by adding a cron job:

  

```bash

sudo crontab -e

```

  

Add the following line to renew the certificates automatically every day:

```bash

0 2 * * * /usr/bin/certbot renew --quiet --deploy-hook "docker-compose restart registry"

```

  

This will attempt to renew the certificate daily and restart the Docker registry service upon successful renewal.

  

### Summary

  

- **Cloudflare's SSL (Flexible or Full Mode)**: You can use Cloudflare's SSL without generating your own certificates. In this case, you can skip certificate management on your side.

- **Cloudflare Origin Certificate**: You can use Cloudflare’s Origin CA certificates for added security on the origin server while using Cloudflare’s proxy.

  

- **Let's Encrypt**: If you prefer using Let's Encrypt, Certbot can help you automatically obtain and renew certificates using Cloudflare’s DNS API for validation.

  

Let me know which method you prefer, and I can guide you further!