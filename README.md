Here’s the README file formatted for GitHub:

---

# Plausible Analytics Setup

This guide provides step-by-step instructions to set up Plausible Analytics with Nginx and SSL using Certbot.

## Prerequisites

Ensure that you have the following installed on your server:

- **Nginx**
- **Certbot**
- **Docker**

## Step 1: Configure Environment Variables

First, create a `.env` file named `plausible-conf.env`:

```plaintext
BASE_URL=replace-me
SECRET_KEY_BASE=replace-me
TOTP_VAULT_KEY=replace-me
```

### Generate the Secret Key Base and TOTP Vault Key

You can generate these keys using OpenSSL:

```bash
$ openssl rand -base64 48
GLVzDZW04FzuS1gMcmBRVhwgd4Gu9YmSl/k/TqfTUXti7FLBd7aflXeQDdwCj6Cz

$ openssl rand -base64 32
dsxvbn3jxDd16az2QpsX5B8O+llxjQ2SJE2i5Bzx38I=
```

Replace the placeholders in `plausible-conf.env` with the generated values. Also, set the `BASE_URL` to the domain where your instance will be accessible.

## Step 2: Nginx Configuration

1. **Install Nginx** (if not already installed):

    ```bash
    sudo apt update
    sudo apt install nginx
    ```

2. **Create an Nginx site configuration file** at `/etc/nginx/sites-available/example.com.conf`:

    ```bash
    sudo nano /etc/nginx/sites-available/example.com.conf
    ```

3. **Add the following configuration** to the file:

    ```nginx
    upstream demo {
        server 127.0.0.1:8080;
    }

    server {
        server_name example.com;

        listen 80;
        rewrite ^(.*)$ https://example.com$1 permanent;
        access_log off;
        error_log off;
    }

    server {
        listen 443 ssl;
        server_name example.com;
        gzip on;

        ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
        ssl_trusted_certificate /etc/letsencrypt/live/example.com/chain.pem;

        include /etc/nginx/conf.d/ssl;
        include /etc/nginx/conf.d/headers;

        access_log off;
        error_log /var/log/nginx/docride.error.log;

        client_max_body_size 100m;

        location / {
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-Host $http_host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_intercept_errors on;
            proxy_pass http://demo;
        }

        location /.well-known {
            root /var/www/html;
        }
    }
    ```

4. **Enable the site by creating a symbolic link**:

    ```bash
    sudo ln -s /etc/nginx/sites-available/example.com.conf /etc/nginx/sites-enabled/
    ```

5. **Restart Nginx**:

    ```bash
    sudo systemctl restart nginx
    ```

## Step 3: Obtain an SSL Certificate with Certbot

To get an SSL certificate for your domain, run:

```bash
sudo certbot certonly --nginx -d example.com
```

Make sure the domain in your Nginx configuration matches the one used with Certbot.

## Step 4: Update the Environment Variables

Finally, update the `BASE_URL` in `plausible-conf.env` with your domain:

```env
BASE_URL=https://example.com
```

## Final Steps

After completing these steps:

1. Ensure your Nginx configuration is working by testing your domain.
2. Confirm your site is accessible via HTTPS.
3. docker-compose up -d 

---
  