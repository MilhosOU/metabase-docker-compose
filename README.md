# metabase-docker-container

Easily deploy Metabase using Docker container with this simple setup. It targets port 80, allowing you to access the Metabase instance directly through your IP address in a browser.


<!-- vim-markdown-toc GFM -->

* [Prerequisites](#prerequisites)
* [Set Up](#set-up)
* [(OPTIONAL) Create HTTPS Certificate](#optional-create-https-certificate)

<!-- vim-markdown-toc -->

## Prerequisites
- Docker and Docker Compose installed on your system.

## Set Up

1. Create a directory called Metabase

```shell
mkdir metabase
cd metabase
```

2. Create a ´docker-compose.yml´ file with the following content:

```yaml
version: '3'
services:
  metabase:
    image: metabase/metabase
    ports:
      - 80:3000
    environment:
      MB_DB_TYPE: postgres
      MB_DB_DBNAME: metabase
      MB_DB_PORT: 5432
      MB_DB_USER: metabase
      MB_DB_PASS: metabase
      MB_DB_HOST: postgres
  postgres:
    image: postgres:latest
    environment:
      POSTGRES_USER: metabase
      POSTGRES_DB: metabase
      POSTGRES_PASSWORD: metabase
    volumes:
      - ./pg:/var/lib/postgresql/data
```

3. Pull the required Docker image

```shell
docker-compose pull
```

4. Start the Metabase and PostgreSQL containers:

```shell
docker-compose up
```

## (OPTIONAL) Create HTTPS Certificate

<details>

> **Warning**
> In order to make it working, you need to update your `docker-container.yml`:
> Change from `80:3000` to `3000:3000`

1. Stop the Docker Compose services:
```
docker-compose down
```

2. Update the package list and install Certbot:
```
sudo apt-get update
sudo apt install -y certbot python3-certbot-apache
```

3. Generate SSL certificates using Certbot:
```
sudo certbot certonly --standalone -d example.com --preferred-challenges http --agree-tos -m your@email.com --keep-until-expiring
```

4. Copy the generated certificates to your project directory:
```
sudo cp -r /etc/letsencrypt/live/example.com ./certs
sudo chown -R $USER:$USER ./certificates __> MAYBE NOT USEFUL
```

5. Install Nginx:
```
sudo apt update
sudo apt install nginx
```

6. Create a new Nginx configuration file:
```
sudo touch /etc/nginx/sites-available/example.com
```

7. Add the following configuration to the file, replacing `example.com` with your domain and `YOUR_APP_IP:3000` with the IP and port of your application:
```
server {
    listen 80;
    server_name example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
       proxy_pass http://YOUR_APP_IP:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```


8. Create a symbolic link to the `sites-enabled` directory:
```
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
```

9. Test the Nginx configuration:
```
sudo nginx -t
```

10. Restart Nginx:

```
sudo systemctl restart nginx
```

11. Start the Docker Compose services:
```
docker-compose up
```

</details>
