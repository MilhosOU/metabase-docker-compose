# metabase-docker-compose

Easily deploy Metabase using Docker container with this simple setup. It targets port 80, allowing you to access the Metabase instance directly through your IP address in a browser.


<!-- vim-markdown-toc GFM -->

* [Prerequisites](#prerequisites)
* [Set Up](#set-up)
* [Update Metabase](#update-metabase)
* [(OPTIONAL) Create HTTPS Certificate](#optional-create-https-certificate)

<!-- vim-markdown-toc -->

## Prerequisites
- Docker and Docker Compose installed on your system.

## Set Up

<details>

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
    image: metabase/metabase:latest
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

</details>

## Update Metabase

<details>

From time to time, Metabase releases new versions with feature improvements, bug fixes, or security patches. It's a good practice to keep your Metabase instance up-to-date. Here are the steps to update Metabase:

1. **Stop Nginx Service (if running)**:
   
   Before making any changes, especially if you have set up HTTPS as shown above, ensure the Nginx service is stopped.
   
   ```shell
   sudo systemctl stop nginx 
   ```

2. **Stop Current Docker Services**:
   
   Ensure your Metabase and PostgreSQL services are stopped before updating.
   
   ```shell
   docker-compose down
   ```

3. **Pull the Latest Metabase Image**:
   
   This step fetches the latest version of the Metabase Docker image.
   
   ```shell
   docker pull metabase/metabase:latest
   ```

4. **Start Nginx Service**:

   If you're using Nginx for SSL termination, start it back up.

   ```shell
   sudo systemctl start nginx
   ```

5. **Restart Docker Services**:
   
   Now, with the updated Metabase image, start the services again.
   
   ```shell
   docker-compose up
   ```

</details>

## (OPTIONAL) Create HTTPS Certificate

<details>

⚠️ Warning: You need to update your docker-compose.yml. Change the port mapping from 80:3000 to 3000:3000.

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
sudo chown -R $USER:$USER ./certificates
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
