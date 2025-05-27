# Dockerfiles

## Dockerfile api_pagamentos (PHP)
```
FROM php:8.2-cli
```
Fazemos um pull da imagem do PHP e a usamos como base para nosso container da API PHP. 

```
WORKDIR /var/www/html
```
Definimos o path dentro do container, onde nosso index.php sera copiado.

```
COPY . .
```
Copiamos tudo que está no diretório para dentro do WORKDIR do container

```
EXPOSE 3003
```
Definimos que nossa aplicação terá a porta 3003 exposta para fora do container.

```
CMD ["php", "-S", "0.0.0.0:3003", "index.php"]
```
Comando para executar a API PHP ao rodar o docker run.

## Dockerfile api_pedidos (Python)
```
FROM python:3.11-slim
```
Fazemos um pull da imagem do python, estamos utilizando a versão slim que é mais leve em comparação com a tradicional

```
WORKDIR /app
```
Definimos o path onde será copiado nosso código para dentro do container

```
COPY requirements.txt .
```
Copiamos primeiro o requirements.txt, para instalar as libs do python

```
RUN pip install --no-cache-dir -r requirements.txt
```
Instalamos as libs especificadas no requirements.txt com o pip.

```
COPY . .
```
Copiamos o restante do código para dentro do container.

```
EXPOSE 3002
```
Definimos que a porta 3002 será exposta para fora do container.

```
CMD ["python", "app.py"]
```
Comando para executar a API Python ao rodar o docker run


## Dockerfile api_produtos (Node.js)
```
FROM node:18-slim
```
Fazemos um pull da imagem do Node.js, versão slim que é mais leve.

```
WORKDIR /app
```
Definimos o path que nosso código será copiado dentro do container.

```
COPY package*.json ./
```
Copiamos o package.json e package-lock.json para instalar as dependencias.

```
RUN npm install
```
Instalamos as dependencias com o NPM.

```
COPY . .
```
Copiamos o restante do código para dentro do container.

```
EXPOSE 3001
```
Definimos que a porta 3001 será exposta pra fora do container.

```
CMD ["node", "index.js"]
```
Comando para iniciar a API Node.js ao rodar o docker run.

# Docker compose
## products
```
products:
build:
    context: ./api_produtos
container_name: products
ports:
    - "3001:3001"
networks:
    - ecommerce-net
```
Serviço da API Node.js, definimos o path do Dockerfile no context, expomos a porta 3001 para fora da rede interna no ports e associamos a network ecommerce-net para se comunicar com os demais containeres.

## orders
```
orders:
build:
    context: ./api_pedidos
container_name: orders
ports:
    - "3002:3002"
depends_on:
    - redis
    - db
    - products
networks:
    - ecommerce-net
```
Serviço da API Python, definimos o path do Dockerfile no context, expomos a porta 3002 para fora da rede interna no ports, definimos que esse serviço deve ser iniciado apenas após os serviços redis, db e products e conectamos esse serviço a rede ecommerce-net.

## payments
```
payments:
build:
    context: ./api_pagamentos
container_name: payments
ports:
    - "3003:3003"
depends_on:
    - orders
networks:
    - ecommerce-net
```
Serviço da API PHP, definimos o path do Dockerfile no context, expomos a porta 3003 para fora da rede interna, definimos que o serviço deve ser iniciado após o serviço orders no depends_on, conectamos na rede ecommerce-net.

## db
```
db:
image: mysql:8.0
container_name: mysql
environment:
    MYSQL_ROOT_PASSWORD: example
    MYSQL_DATABASE: ecommerce
ports:
    - "3306"
networks:
    - ecommerce-net
```
Serviço do banco de dados MySQL, usamos a imagem mysql:8.0, definimos o nome e senha do banco no environment, definimos a port 3306 internamente, conectamos a rede ecommerce-net.

## redis
```
redis:
image: redis:alpine
container_name: redis
ports:
    - "6379"
networks:
    - ecommerce-net
```
Serviço do redis, usamos a imagem redis:alpine, definimos a port 6379 internamente e conectamos a rede ecommerce-net.

## network
```
networks:
  ecommerce-net:
    driver: bridge
```
Network do docker-compose, no modo bridge ela se conecta a rede do computador host.
