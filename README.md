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



  ## Acesso aos serviços a partir do Windows (host)

Como o ambiente está rodando dentro do Debian (VM no VirtualBox), o Windows não acessa diretamente o IP interno do Minikube (192.168.49.x).  
Para permitir acesso, é necessário redirecionar portas usando minikube kubectl -- port-forward.

### Exemplo: Nginx
No Debian, execute:
minikube kubectl -- port-forward svc/nginx-svc 8080:80 -n websolutions

Depois, no Windows, acesse:
http://<IP_DEBIAN>:8080

### Exemplo: Apache
No Debian, execute:
minikube kubectl -- port-forward svc/apache-svc 8081:80 -n websolutions

Depois, no Windows, acesse:
http://<IP_DEBIAN>:8081

### Como descobrir o IP do Debian
No Debian, rode:
ip addr show

Procure o IP da interface enp0s3 (exemplo: 172.29.230.11).  
Esse é o IP que o Windows deve usar para acessar os serviços redirecionados.

Como liberar para o Windows

Você tem três opções:
1. Usar minikube service
Esse comando abre o serviço e mostra a URL:
minikube service nginx-svc -n websolutions --url
minikube service apache-svc -n websolutions --url


Ele retorna algo como:
http://192.168.49.2:30080
http://192.168.49.2:30081


Esses endereços funcionam dentro do Debian.
Se a rede do VirtualBox estiver em Bridge, o Windows também consegue acessar.

2. Usar minikube tunnel
Esse comando cria um túnel e expõe os serviços como se fossem LoadBalancer:
minikube tunnel


Depois rode:
minikube service list


→ Vai mostrar os IPs e portas que ficam acessíveis.
Esses IPs podem ser usados no Windows.