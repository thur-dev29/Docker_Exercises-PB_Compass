# Nível Fácil

## 1 - Rodando um container básico
```bash
docker pull nginx

docker run -d -p 8080:80 --name Nginx-A nginx

docker cp /mnt/c/sites/tailwind/. Nginx-A:/usr/share/nginx/html
```
http://localhost:8080/

## 2 - Criando e rodando um container interativo
```bash
docker run -dti --name Ubuntu-A ubuntu

docker exec -ti Ubuntu-A bash

apt update && apt install nano

nano logs.sh
```

Dentro de **logs.sh**

```bash
#!/bin/bash

echo "=== Logs do Kernel ==="
dmesg | tail -50
```
```bash
chmod +x logs.sh

./logs.sh
```

## 3 - Listando e removendo containers

Listar containers em execução => 
```bash
docker ps
```
Listar todos os containers (incluindo os parados) => 
```bash
docker ps -a
```
Parar um container => 
```bash
docker stop <container_id>
```
Remover um container => 
```bash
docker rm <container_id>
```

## 4 - Criando um Dockerfile para uma aplicação simples em Python
```bash
mkdir api-flask && cd api-flask

sudo nano app.py
```
Dentro de **app.py**
```bash
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "API Flask funcionando corretamente!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```
```bash
echo "Flask==2.2.5" > requirements.txt

sudo nano Dockerfile
```
Dentro de **Dockerfile**
```bash
FROM python:3.9

WORKDIR /app

COPY . .

RUN pip install --no-cache-dir -r requirements.txt

EXPOSE 5000

CMD ["python", "app.py"]
```
```bash
docker build -t api .

docker run -p 5000:5000 api
```
http://localhost:5000/

# Nível Médio

## 5 - Criando e utilizando volumes para persistência de dados

```bash
sudo apt update && sudo apt install -y php-cli php-mbstring php-xml php-bcmath php-curl unzip curl git composer php-mysql

curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

docker volume create mysql_data

docker run --name mysql-breeze -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=laravel -e MYSQL_USER=laravel -e MYSQL_PASSWORD=secret -p 3306:3306 -v mysql_data:/var/lib/mysql -d mysql:8

composer create-project laravel/laravel laravel-breeze && cd laravel-breeze

sudo nano .env

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=secret

composer require laravel/breeze --dev
php artisan breeze:install
php artisan migrate

npm install
npm run dev

php artisan serve
http://127.0.0.1:8000

docker rm -f mysql-breeze

docker run --name mysql-breeze -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=laravel -e MYSQL_USER=laravel -e MYSQL_PASSWORD=secret -p 3306:3306 -v mysql_data:/var/lib/mysql -d mysql:8

docker exec -it mysql-breeze mysql -u laravel -p
USE laravel;
SELECT * FROM users;
```

## 6 Criando e rodando um container multi-stage
```bash
wget https://go.dev/dl/go1.24.1.linux-amd64.tar.gz
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.24.1.linux-amd64.tar.gz
sudo nano ~/.bashrc

export PATH=$PATH:/usr/local/go/bin
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin

source ~/.bashrc

mkdir go-fiber-app && cd go-fiber-app

go mod init example.com/go-fiber-app

go get github.com/gofiber/fiber/v2

sudo nano main.go

package main

import (
        "github.com/gofiber/fiber/v2"
)

func main() {
        app := fiber.New()

        app.Get("/", func(c *fiber.Ctx) error {
                return c.SendString("A aplicação está funcionando corretamente!")
        })

        app.Listen(":3000")
}

sudo nano Dockerfile

FROM golang:1.24 AS builder

WORKDIR /app

ENV CGO_ENABLED=0 GOOS=linux GOARCH=amd64

COPY go.mod go.sum ./
RUN go mod download

COPY . .

RUN go build -o main .

FROM alpine:latest

WORKDIR /root/

COPY --from=builder /app/main .
RUN chmod +x main

EXPOSE 3000

CMD ["./main"]

docker build -t go-fiber-app .

docker run -d -p 3000:3000 --name meu-fiber go-fiber-app

curl http://localhost:3000
```

## 7 - Construindo uma rede Docker para comunicação entre containers

### 1. Criar os arquivos do projeto
```sh
mkdir meu-projeto && cd meu-projeto
touch Dockerfile docker-compose.yml index.js package.json
```

### 2. Criar o `Dockerfile`
```dockerfile
FROM node:18
WORKDIR /app
COPY package.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

### 3. Criar o `docker-compose.yml`
```yaml
version: '3.8'

services:
  app:
    build: .
    container_name: meu_node
    networks:
      - minha_rede
    depends_on:
      mongo:
        condition: service_healthy
    ports:
      - "3000:3000"
    volumes:
      - .:/app

  mongo:
    image: mongo:6
    container_name: meu_mongo
    networks:
      - minha_rede
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
    healthcheck:
      test: echo 'db.runCommand("ping").ok' | mongosh --quiet localhost:27017
      interval: 10s
      timeout: 5s
      retries: 5

networks:
  minha_rede:

volumes:
  mongo_data:
```

### 4. Criar o `package.json`
```json
{
  "name": "meu-projeto",
  "version": "1.0.0",
  "main": "index.js",
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.6.0"
  },
  "scripts": {
    "start": "node index.js"
  }
}
```

### 5. Criar o `index.js`
```js
const express = require('express');
const mongoose = require('mongoose');

const app = express();
const PORT = 3000;
const MONGO_URL = 'mongodb://mongo:27017/meubanco';

app.use(express.json());

mongoose.connect(MONGO_URL, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log('🔥 Conectado ao MongoDB'))
  .catch(err => console.error('Erro ao conectar ao Mongo:', err));

app.listen(PORT, () => {
    console.log(`🚀 Servidor rodando em http://localhost:${PORT}`);
});
```

### 6. Instalar Dependências
```sh
npm install
```

Se estiver usando Docker, instale as dependências antes de construir o container:
```sh
docker run --rm -v $(pwd):/app -w /app node:18 npm install
```

### 7. Subir os Containers
```sh
docker-compose up -d --build
```

Se houver erro de módulos ausentes:
```sh
docker-compose down -v
npm install
docker-compose up -d --build
```

### 8. Verificar se os containers estão rodando
```sh
docker ps
```

### 9. Testar a Conexão com o MongoDB
```sh
docker logs meu_node
```

## 8 - Criando um compose file para rodar uma aplicação com banco de dados

### Configuração

1. Clone este repositório e entre no diretório do projeto:
   ```sh
   git clone <url-do-repositorio>
   cd <nome-do-diretorio>
   ```

2. Construa a imagem do Django:
   ```sh
   docker-compose build
   ```

3. Suba os containers:
   ```sh
   docker-compose up -d
   ```

4. Verifique se os containers estão rodando:
   ```sh
   docker ps
   ```

### Configuração do Django

1. Acesse o container do Django:
   ```sh
   docker exec -it django_container bash
   ```

2. Aplique as migrações do banco de dados:
   ```sh
   python manage.py migrate
   ```

3. Crie um superusuário:
   ```sh
   python manage.py createsuperuser
   ```

4. Saia do container:
   ```sh
   exit
   ```

### Testando a Aplicação

1. Acesse a aplicação no navegador:
   ```
   http://localhost:8000
   ```

2. Acesse o painel administrativo:
   ```
   http://localhost:8000/admin
   ```

3. Teste a API via terminal:
   ```sh
   curl -i http://localhost:8000
   ```

### Parar e Remover Containers

Para parar e remover os containers, use:
```sh
   docker-compose down -v
```

## 9 - Criando uma imagem personalizada com um servidor web e arquivos estáticos
### 1. Criar a estrutura do projeto
```bash
mkdir -p ~/meu-site-nginx
cd ~/meu-site-nginx
```

### 2. Copiar os arquivos do site para a pasta do projeto
Baixar e extrair os arquivos do site na pasta `~/meu-site-nginx`.

### 3. Criar o arquivo Dockerfile
```bash
nano Dockerfile
```
**Conteúdo:**
```dockerfile
# Etapa 1: Construir o projeto
FROM node:18 AS build

WORKDIR /app

# Copiar os arquivos do projeto
COPY . .

# Instalar dependências e construir o projeto
RUN npm install && npm run build

# Etapa 2: Criar a imagem final com Nginx
FROM nginx:latest

# Copiar os arquivos compilados para a pasta padrão do Nginx
COPY --from=build /app/build /usr/share/nginx/html

# Expor a porta 80
EXPOSE 80
```

### 4. Construir a imagem Docker
```bash
docker build -t meu-site-nginx .
```

### 5. Remover container antigo (se necessário)
```bash
docker stop site-container
docker rm site-container
```

### 6. Rodar o novo container
```bash
docker run -d -p 8080:80 --name site-container meu-site-nginx
```

### 7. Acessar o site no navegador
```
http://localhost:8080
```
