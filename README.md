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
