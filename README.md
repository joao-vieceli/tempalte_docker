# Projeto

## Estrutura do Projeto / Publicação

### Pastas:
- nginx
- postgres
- application

---

## Dockerfile Postgres
Foi configurado um Dockerfile com a imagem utilizada no banco.

```dockerfile
FROM postgres:16.4
```

---

## Dockerfile Nginx

```dockerfile
FROM nginx:alpine

COPY ./nginx.conf /etc/nginx/conf.d/default.conf
```

### nginx.conf
```nginx
server {
    listen 80;

    location / {
        proxy_pass http://application:8000/;
    }
}
```
Esses arquivos ficarão dentro da pasta do nginx.

---

## Application

Para montar a aplicação, você precisa executar os seguintes comandos.
> **OBS:** É necessário ter Laravel instalado. Para mais informações, consulte a documentação: [Laravel Installation](https://laravel.com/docs/12.x/installation)

### Arquivos de configuração:

#### afterbuild-dev.sh
```bash
#!/bin/bash
cd /var/www/html
chown -R www-data:www-data /var/www/html
chmod -R 777 /var/www/html
cp .env.example .env
npm install
composer install
php artisan key:generate
php artisan migrate:fresh
php artisan db:seed --class=TarefaSeeder
npm run dev & php artisan serve --host=0.0.0.0
```

#### Dockerfile.dev
```dockerfile
FROM php:8.2-apache

WORKDIR /var/www/html

COPY . .

RUN apt-get update && apt-get install -y gpg
RUN curl -fsSL https://packages.sury.org/php/README.txt | bash -x

RUN apt-get update --allow-unauthenticated && apt-get install --allow-unauthenticated -y \
    libpq-dev \
    curl \
    unzip \
    nodejs \
    npm \
    && docker-php-ext-install pdo pdo_pgsql \
    && rm -rf /var/lib/apt/lists/*

RUN curl -sS https://getcomposer.org/installer -o composer-setup.php \
    && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
    && rm composer-setup.php

CMD ["/var/www/html/afterbuild-dev.sh"]
```

#### .env.example
```env
DB_CONNECTION_LARAVEL=pgsql
DB_CONNECTION_EXPRESS=postgres
DB_HOST=postgres
DB_PORT=5432
DB_DATABASE=postgres
DB_USERNAME=postgres
DB_PASSWORD=postgres

TRUST_KEY=a2FrYXVfYm9tYmFkYW8=

URL_DEV=http://localhost:80
```

---

## Docker Compose

### docker-compose-dev.yml

```yaml
services:
  postgres:
    container_name: postgres
    restart: always
    build:
      context: ./postgres
      dockerfile: Dockerfile
    environment:
      POSTGRES_DB: ${DB_DATABASE}
      POSTGRES_USER: ${DB_USERNAME}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    ports:
      - "5000:5432"
    expose:
      - 5432
    volumes:
      - ./postgres/data:/var/lib/postgresql/data
      - ./postgres/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - task2

  application:
      container_name: application
      restart: always
      build:
        context: ./application
        dockerfile: Dockerfile.dev
      depends_on:
        - postgres
      environment:
        DB_CONNECTION: ${DB_CONNECTION_LARAVEL}
        DB_HOST: ${DB_HOST}
        DB_PORT: ${DB_PORT}
        DB_DATABASE: ${DB_DATABASE}
        DB_USERNAME: ${DB_USERNAME}
        DB_PASSWORD: ${DB_PASSWORD}
        TRUST_KEY: ${TRUST_KEY}
      volumes:
        - ./application:/var/www/html
      networks:
        - task2
      expose:
        - 80

  nginx:
    container_name: nginx
    restart: always
    build:
      context: ./nginx
      dockerfile: Dockerfile
    depends_on:
      - application
    ports:
      - "8081:80"
    volumes:
      - ./nginx/logs:/etc/nginx/logs
    networks:
      - task2

networks:
  task2:
    driver: bridge
```

---

## Servidor

### Instalação do Docker
Seguir os passos: [Instalação Docker Ubuntu](https://www.hostinger.com.br/tutoriais/instalar-docker-ubuntu)

### VM
- **IP:** 177.44.248.66
- **Link da Aplicação:** [Acessar](http://177.44.248.66:8081/)

---

## Aplicação

### Modelagem
Criação manual da tabela `tarefas`:
```sql
CREATE TABLE tarefas (
    id SERIAL PRIMARY KEY,
    descricao VARCHAR(255) NOT NULL,
    data_criacao TIMESTAMP NOT NULL,
    data_prevista TIMESTAMP,
    data_encerramento TIMESTAMP,
    situacao VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
---

## Link do Repositório
[GitHub - task_2](https://github.com/joao-vieceli/task_2)
