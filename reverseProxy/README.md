# Nginx Reverse Proxy with Automatic SSL

This setup provides a reverse proxy using Nginx with automatic SSL certificate management via Certbot/Let's Encrypt.

## Directory Structure

```
reverseProxy/
├── docker-compose.yml
├── nginx/
│   └── conf.d/
│       └── default.conf
└── certbot/
    ├── conf/       # Let's Encrypt configurations and certificates (auto-managed)
    └── www/        # Webroot for domain validation
```

## Features

- Nginx reverse proxy
- Automatic SSL certificate management with Let's Encrypt
- Automatic certificate renewal
- HTTP to HTTPS redirection
- Modern SSL configuration

## Prerequisites

1. Ensure your domain(s) are pointing to your server's IP address
2. Make sure ports 80 and 443 are open and accessible

## Initial Setup

The setup requires minimal manual configuration. The certbot container will automatically:
- Create necessary certificate directories
- Handle certificate renewals
- Manage renewal configurations

You only need to create the base directories:
```bash
mkdir -p nginx/conf.d certbot/conf certbot/www
```

## Usage

Optional step: Allow unprivileged user to listen to 80/443 

```bash
sudo vim /etc/sysctl.conf
# add
# net.ipv4.ip_unprivileged_port_start = 80
# net.ipv4.ip_unprivileged_port_start = 443
sudo sysctl -p
```

1. Start the services:
   ```bash
   # use sudo if there is no privilege
   docker compose up -d
   ```

2. To add a new domain:

   a. Create a new configuration file in `nginx/conf.d/` (e.g., `example.com.conf`)
   b. Use this basic configuration for initial certificate obtainment:
   ```nginx
   server {
       listen 80;
       listen [::]:80;
       server_name your-domain.com www.your-domain.com;
   
       location /.well-known/acme-challenge/ {
           root /var/www/certbot;
       }
   }
   ```

3. Reload nginx to apply the configuration:
   ```bash
   sudo docker compose exec nginx nginx -s reload
   ```

4. Obtain certificates for your domain:
   ```bash
   sudo docker compose run --rm certbot certonly \
     --webroot \
     --webroot-path /var/www/certbot \
     --email your-email@example.com \
     --agree-tos \
     --no-eff-email \
     --force-renewal \
     --domain example.com 
   ```

   Or you can login container to run:
   ```bash
   certbot certonly --webroot --webroot-path /var/www/certbot --email dev@lizhao.net --agree-tos --no-eff-email -d n8n.lizhao.net
   ```

5. After obtaining certificates, update your domain's nginx configuration with the full HTTPS setup:
   ```nginx
   server {
       listen 80;
       listen [::]:80;
       server_name example.com www.example.com;
   
       location /.well-known/acme-challenge/ {
           root /var/www/certbot;
       }
   
       location / {
           return 301 https://$host$request_uri;
       }
   }
   
   server {
       listen 443 ssl;
       listen [::]:443 ssl;
       http2 on;  # Modern way to enable HTTP/2
       server_name example.com www.example.com;
   
       ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
       ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
       
       # SSL configuration
       ssl_session_timeout 1d;
       ssl_session_cache shared:SSL:50m;
       ssl_session_tickets off;
   
       # Modern configuration
       ssl_protocols TLSv1.2 TLSv1.3;
       ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
       ssl_prefer_server_ciphers off;
   
       # HSTS
       add_header Strict-Transport-Security "max-age=63072000" always;
   
       # OCSP Stapling
       ssl_stapling on;
       ssl_stapling_verify on;
       resolver 8.8.8.8 8.8.4.4 valid=300s;
       resolver_timeout 5s;
   
       location / {
           proxy_pass http://your-backend-service:port;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

Note: The HTTP/2 configuration uses the modern `http2 on;` directive instead of the deprecated `listen ... http2` syntax.

6. Test the configuration and reload Nginx:
   ```bash
   sudo docker compose exec nginx nginx -t
   sudo docker compose exec nginx nginx -s reload
   ```

## SSL Certificate Renewal

Certificates will be automatically renewed when they are close to expiration. The certbot container handles this automatically by:

1. Checking certificate expiration every 12 hours
2. Managing renewal configurations in certbot/conf/renewal/
3. Using the webroot authenticator for validation
4. Reloading nginx after successful renewal

No manual intervention is required for renewals.

## Troubleshooting

1. If certbot fails to obtain certificates:
   - Ensure your domain is pointing to your server's IP
   - Check that port 80 is accessible
   - Verify nginx is running: `docker compose ps`
   - Check nginx logs: `docker compose logs nginx`
   - Check certbot logs: `docker compose logs certbot`

2. If nginx fails to start:
   - Check the configuration: `docker compose exec nginx nginx -t`
   - Verify the certificate paths exist
   - Check nginx error logs: `docker compose logs nginx`

3. If you see HTTP/2 related warnings:
   - Update your nginx configuration to use `http2 on;` instead of `listen ... http2`
   - This is the modern way to enable HTTP/2 in newer versions of nginx

## Security Considerations

- The included Nginx configuration uses modern SSL settings
- HSTS is enabled by default in the HTTPS configuration template
- Only TLS 1.2 and 1.3 are enabled
- HTTP traffic is automatically redirected to HTTPS

## Volumes

- `/etc/nginx/conf.d`: Nginx configuration files
- `/etc/letsencrypt`: Let's Encrypt certificates and configuration (auto-managed by certbot)
- `/var/www/certbot`: Webroot for Let's Encrypt domain validation