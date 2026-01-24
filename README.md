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

## Fluxo de execução

1. Build das imagens personalizadas:
   docker build -t nginx-poc:1.0 -f docker/nginx/Dockerfile docker/nginx
   docker build -t apache-poc:1.0 -f docker/apache/Dockerfile docker/apache

2. Carregar as imagens no Minikube:
   minikube image load nginx-poc:1.0
   minikube image load apache-poc:1.0

3. Aplicar os manifests Kubernetes:
   minikube kubectl -- apply -f k8s/

4. Verificar os pods:
   minikube kubectl -- get pods -n websolutions

5. Verificar o IP do Minikube:
   minikube ip

6. Acessar os serviços:
   Nginx: http://<IP_MINIKUBE>:30080
   Apache: http://<IP_MINIKUBE>:30081

## Observações sobre desligar a máquina

- Ao desligar o computador, o Minikube é parado e os pods deixam de rodar.
- As imagens Docker e os manifests permanecem salvos localmente.
- Os dados do MariaDB são mantidos graças ao PVC (PersistentVolumeClaim).
- Ao ligar novamente, basta executar:
  minikube start
  minikube kubectl -- apply -f k8s/
  minikube kubectl -- get pods -n websolutions