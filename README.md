# Websolutions POC

Este projeto é uma prova de conceito (POC) para orquestração de containers com Kubernetes, utilizando os servidores Nginx e Apache, além de MariaDB como banco de dados.

## Estrutura do projeto

websolutions/
├── README.md
├── docker/
│   ├── nginx/
│   │   ├── Dockerfile
│   │   └── html/
│   │       └── index.html
│   ├── apache/
│   │   ├── Dockerfile
│   │   └── html/
│   │       └── index.html
├── k8s/
│   ├── apache-deploy.yaml
│   ├── apache-svc.yaml
│   ├── mariadb-config.yaml
│   ├── mariadb-deploy.yaml
│   ├── mariadb-pvc.yaml
│   ├── mariadb-secret.yaml
│   ├── mariadb-svc.yaml
│   ├── namespace.yaml
│   ├── nginx-deploy.yaml
│   └── nginx-svc.yaml

## Tecnologias utilizadas

- Docker
- Kubernetes (Minikube)
- Nginx
- Apache HTTP Server
- MariaDB
- Git e GitHub

## Como executar

1. Build das imagens:
   docker build -t nginx-poc:1.0 -f docker/nginx/Dockerfile docker/nginx
   docker build -t apache-poc:1.0 -f docker/apache/Dockerfile docker/apache

2. Carregar no Minikube:
   minikube image load nginx-poc:1.0
   minikube image load apache-poc:1.0

3. Aplicar os manifests:
   minikube kubectl -- apply -f k8s/

4. Acessar os serviços:
   Nginx: http://192.168.49.2:30080
   Apache: http://192.168.49.2:30081