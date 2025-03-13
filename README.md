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

Dentro do **logs.sh**

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
