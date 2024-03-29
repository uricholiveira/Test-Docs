# Deploy

<!--Writerside adds this topic when you create a new documentation project.
You can use it as a sandbox to play with Writerside features, and remove it from the TOC when you don't need it anymore.
If you want to re-add it for your experiments, click + to create a new topic, choose Topic from Template, and select the 
"Starter" template.-->

## Instalando o Docker
````Shell
#!/bin/bash

sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo usermod -aG docker ${USER}
newgrp docker
````

## Logando no Registry da Lyncas
````Shell
#!/bin/bash

echo "glpat-en-fF8MdH_XxWjzJhAL2" | docker login dockerregistry.lyncas.net --username urich.o --password-stdin
````


## SSL
Caso não tenham SSL em mãos, é necessário gerar através do certbot.

Os passos são:
- Instanciar um NGINX na porta 80 e 443.
````Shell
docker run -p 80:80 -p 443:443 nginx:latest
````
- Instanciar o Certbot do **Docker Compose** abaixo
- Aguardar a geração do SSL dos domínios/subdomínios
- Parar a instância do NGINX, e do Certbot
- Iniciar o Docker Compose.


## Docker Compose
````yaml
version: '3.8'

services:
  db:
    image: postgres:16
    container_name: postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: mysecretpassword
      POSTGRES_DB: postgres
    ports:
      - "5432:5432"
    networks:
      - local

  minio:
    image: docker.io/bitnami/minio:latest
    container_name: minio
    ports:
      - '9000:9000'
      - '9001:9001'
    volumes:
      - 'minio_data:/data'
    environment:
      - MINIO_ROOT_USER=admin
      - MINIO_ROOT_PASSWORD=Capslock123@
      - MINIO_DEFAULT_BUCKETS=bucket
    networks:
      - local

  rabbitmq:
    image: 'rabbitmq:3-management'
    container_name: rabbitmq
    environment:
      - RABBITMQ_DEFAULT_USER=guest
      - RABBITMQ_DEFAULT_PASS=guest
    ports:
      - '5672:5672'
      - '15672:15672'
    networks:
      - local
  
  certbot:
    image: certbot/certbot:latest
    container_name: certbot
    volumes:
      - ./letsencrypt:/etc/letsencrypt
      - ./data:/var/www/html
    command:
      - certonly
      - --webroot
      - -w
      - /var/www/html
      - -d
      - bunker.develop.bravante.com.br
      - -d
      - bunker-auth.develop.bravante.com.br
      - -d
      - bunker-api.develop.bravante.com.br
      - -d
      - bunker-storage.develop.bravante.com.br
      - --rsa-key-size
      - "4096"
      - --agree-tos
      - --force-renewal
      - --email
      - urich.o@lyncas.net

  nginx:
    image: nginx:latest
    container_name: nginx
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./letsencrypt:/etc/letsencrypt
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - frontend
      - auth
      - backend
    networks:
      - local

  frontend:
    image: dockerregistry.lyncas.net/lyncas/projetos/fabrica-de-software/bravante/frontend:develop
    container_name: frontend
    ports:
      - "3000:3000"
    networks:
      - local

  auth:
    image: dockerregistry.lyncas.net/lyncas/projetos/fabrica-de-software/bravante/backend.auth:develop
    #    image: local/backend.mesa-operacao:develop
    container_name: auth
    ports:
      - "3001:8080"
      - "3003:3003"
    depends_on:
      - db
    environment:
      ASPNETCORE_ENVIRONMENT: "Development"
      LOGGING__LOGLEVEL__DEFAULT: "Information"
      LOGGING__LOGLEVEL__MICROSOFT_ASPNETCORE: "Warning"
      CONNECTIONSTRINGS__DATABASE: "host=db;port=5432;database=postgres;username=postgres;password=mysecretpassword;Include Error Detail=true"
      JWTOPTIONS__ISSUER: "https://bunker-api.develop.bravante.com.br"
      JWTOPTIONS__AUDIENCE: "Audience"
      JWTOPTIONS__SECURITYKEY: "x4UXPefTBvX2sEwK1xPWxkUSFqP4nFYd0G2rIYJ1hxjofOzC+FGMvv7OmaGANQKXUJSBP+Yk/0/fbGpV6tyTSA=="
      JWTOPTIONS__ACCESSTOKENEXPIRATION: "360"
      JWTOPTIONS__REFRESHTOKENEXPIRATION: "420"
      CorsPolicy__AllowOrigins__0: "*"
      CorsPolicy__AllowMethods__0: "GET"
      CorsPolicy__AllowMethods__1: "POST"
      CorsPolicy__AllowMethods__2: "PATCH"
      CorsPolicy__AllowMethods__3: "PUT"
      CorsPolicy__AllowMethods__4: "DELETE"
      CorsPolicy__AllowMethods__5: "OPTIONS"
      CorsPolicy__AllowHeaders__0: "Authorization"
      CorsPolicy__AllowHeaders__1: "Content-Type"
      Minio__Endpoint: "minio:9000"
      Minio__AccessKey: "yyu5bHcdmoBhAnkP2dU7"
      Minio__SecretKey: "snHeDOUJ5rPCg7BX2ZhbNFeBPnbn7iJ4pJ03qwOv"
      RabbitMQ__Host: "rabbitmq"
      RabbitMQ__Port: 5672
      RabbitMQ__Username: "guest"
      RabbitMQ__Password: "guest"
    networks:
      - local
  
  backend:
    image: dockerregistry.lyncas.net/lyncas/projetos/fabrica-de-software/bravante/backend.mesa-operacao:develop
    #    image: local/backend.mesa-operacao:develop
    container_name: backend
    ports:
      - "3002:8080"
    depends_on:
      - db
    environment:
      ASPNETCORE_ENVIRONMENT: "Development"
      LOGGING__LOGLEVEL__DEFAULT: "Information"
      LOGGING__LOGLEVEL__MICROSOFT_ASPNETCORE: "Warning"
      CONNECTIONSTRINGS__DATABASE: "host=db;port=5432;database=postgres;username=postgres;password=mysecretpassword;Include Error Detail=true"
      JWTOPTIONS__ISSUER: "https://bunker-api.develop.bravante.com.br"
      JWTOPTIONS__AUDIENCE: "Audience"
      JWTOPTIONS__SECURITYKEY: "x4UXPefTBvX2sEwK1xPWxkUSFqP4nFYd0G2rIYJ1hxjofOzC+FGMvv7OmaGANQKXUJSBP+Yk/0/fbGpV6tyTSA=="
      JWTOPTIONS__ACCESSTOKENEXPIRATION: "360"
      JWTOPTIONS__REFRESHTOKENEXPIRATION: "420"
      CorsPolicy__AllowOrigins__0: "*"
      CorsPolicy__AllowMethods__0: "GET"
      CorsPolicy__AllowMethods__1: "POST"
      CorsPolicy__AllowMethods__2: "PATCH"
      CorsPolicy__AllowMethods__3: "PUT"
      CorsPolicy__AllowMethods__4: "DELETE"
      CorsPolicy__AllowMethods__5: "OPTIONS"
      CorsPolicy__AllowHeaders__0: "Authorization"
      CorsPolicy__AllowHeaders__1: "Content-Type"
      Minio__Endpoint: "minio:9000"
      Minio__AccessKey: "yyu5bHcdmoBhAnkP2dU7"
      Minio__SecretKey: "snHeDOUJ5rPCg7BX2ZhbNFeBPnbn7iJ4pJ03qwOv"
      RabbitMQ__Host: "rabbitmq"
      RabbitMQ__Port: 5672
      RabbitMQ__Username: "guest"
      RabbitMQ__Password: "guest"
    networks:
      - local
      
volumes:
  minio_data:
    driver: local

networks:
  local:
    driver: bridge
````

## Configuração do NGINX

````Text
events {}
http {
server {
    listen 80;
    server_name bunker.develop.bravante.com.br;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        proxy_pass http://frontend:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

   listen 443 ssl; # managed by Certbot
   	ssl_certificate /etc/letsencrypt/live/bunker.develop.bravante.com.br/fullchain.pem; # managed by Certbot
   	ssl_certificate_key /etc/letsencrypt/live/bunker.develop.bravante.com.br/privkey.pem; # managed by Certbot
}

server {
    listen 80;
    server_name bunker-auth.develop.bravante.com.br;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        proxy_pass http://auth:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /ws {
        proxy_pass http://auth:3003;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

   listen 443 ssl; # managed by Certbot
   	ssl_certificate /etc/letsencrypt/live/bunker.develop.bravante.com.br/fullchain.pem; # managed by Certbot
   	ssl_certificate_key /etc/letsencrypt/live/bunker.develop.bravante.com.br/privkey.pem; # managed by Certbot
}

server {
    listen 80;
    server_name bunker-api.develop.bravante.com.br;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        proxy_pass http://backend:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

   listen 443 ssl; # managed by Certbot
   	ssl_certificate /etc/letsencrypt/live/bunker.develop.bravante.com.br/fullchain.pem; # managed by Certbot
   	ssl_certificate_key /etc/letsencrypt/live/bunker.develop.bravante.com.br/privkey.pem; # managed by Certbot
}

server {
    listen 80;
    server_name bunker-storage.develop.bravante.com.br;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        proxy_pass http://minio:9001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

   listen 443 ssl; # managed by Certbot
   	ssl_certificate /etc/letsencrypt/live/bunker.develop.bravante.com.br/fullchain.pem; # managed by Certbot
   	ssl_certificate_key /etc/letsencrypt/live/bunker.develop.bravante.com.br/privkey.pem; # managed by Certbot
}
}
````