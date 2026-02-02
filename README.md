# Websolutions POC

Este projeto é uma prova de conceito (POC) para orquestração de containers com Kubernetes,
utilizando os servidores Nginx e Apache, além de MariaDB como banco de dados.

## Estrutura do projeto
```bash
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
```
## Tecnologias utilizadas

- Docker
- Kubernetes (Minikube)
- Nginx
- Apache HTTP Server
- MariaDB
- Git e GitHub

## Fluxo de execução



1. Iniciar o Cluster
```bash
   minikube start --driver=docker --kubernetes-version=v1.30.0 \
    --extra-config=apiserver.service-node-port-range=1-65535
```
2. Build das imagens personalizadas:
```bash
   docker build -t nginx-poc:1.0 -f docker/nginx/Dockerfile docker/nginx
   docker build -t apache-poc:1.0 -f docker/apache/Dockerfile docker/apache
```
3. Carregar as imagens no Minikube:
```bash
   minikube image load nginx-poc:latest
   minikube image load apache-poc:latest 
```
4. Aplicar os manifests Kubernetes:
```bash
   minikube kubectl -- apply -f k8s/
```
5. Verificar os pods:
```bash
   minikube kubectl -- get pods -n websolutions
```
6. Verificar o IP do Minikube:
```bash
   minikube ip
```
7. Acessar os serviços:
```bash
   Nginx: http://<IP_MINIKUBE>:8080
   Apache: http://<IP_MINIKUBE>:8081
```
## Acessando o MariaDB

Para testar a conexão com o banco de dados dentro do cluster:
   minikube kubectl -- run -it --rm mariadb-client --image=mariadb:11.4 -n websolutions -- \
     mysql -h mariadb-svc -uwebuser -pwebpass webdb

### ⚡ Observações sobre desligar a máquina

- Ao desligar o computador, o **Minikube** é parado e os pods deixam de rodar.
- As **imagens Docker** e os **manifests** permanecem salvos localmente.
- Os dados do **MariaDB** são mantidos graças ao **PVC (PersistentVolumeClaim)**.
- Ao ligar novamente, basta executar:

```bash
sudo systemctl restart docker.socket
sudo systemctl restart docker

minikube start --driver=docker --kubernetes-version=v1.30.0 \
  --extra-config=apiserver.service-node-port-range=1-65535
  
minikube kubectl -- apply -f k8s/namespace.yaml
minikube kubectl -- get pods -n websolutions
```
## Troubleshooting
```bash
- ErrImagePull → usar minikube image load para carregar imagens locais.
- ContainerCreating → verificar PVC ou ConfigMap.
- CrashLoopBackOff → checar probes e variáveis de ambiente.
```
##  Compatibilidade

| Ferramenta   | Versão testada | Observações |
|--------------|----------------|-------------|
| **Docker**   | 26.x (ex: 26.1.3) | Compatível com Minikube v1.37.0. Versões 28/29 causam erro de comunicação. |
| **Minikube** | v1.37.0         | Funciona bem com Docker 25.x e 26.x. Para Docker 29.x é necessário Minikube >= v1.38. |
| **Kubernetes** | v1.30.0       | Estável e suportado. Versão v1.34.0 apresentou falhas de inicialização. |
| **Sistema**  | Debian 13 (Trixie) | Repositório oficial do Docker só fornece 28/29. Usar repositório Bookworm para instalar 25/26. |

###  Notas importantes
- Se usar **Docker 29.x**, atualize o Minikube para v1.38 ou superior.  
- Se permanecer no **Minikube v1.37.0**, mantenha Docker na versão 25.x ou 26.x.  
- Sempre especifique a versão do Kubernetes ao iniciar o cluster:
```bash
  minikube start --driver=docker --kubernetes-version=v1.30.0
```

==============================================================================================================================
==============================================================================================================================

## Entendendo o Fluxo da Aplicação

Para compreender como tudo funciona, vamos seguir o caminho desde a criação da imagem até o acesso pelo navegador:

1. **Dockerfile**
   - É o roteiro que ensina o Docker a montar uma imagem.
   - Nele definimos:
     - Qual imagem base usar (ex.: `httpd:latest` ou `nginx:latest`).
     - Quais arquivos copiar (HTML personalizado).
     - Qual porta o container deve expor.
   - Resultado: uma **imagem Docker** pronta para rodar.

2. **Imagem Docker**
   - É como uma "receita congelada": contém o servidor web + seus arquivos.
   - Exemplo: `apache-poc:1.0` e `nginx-poc:1.0`.
   - A imagem é estática, não roda sozinha. Precisa ser instanciada.

3. **Deployment (Kubernetes)**
   - Define como os pods devem ser criados e gerenciados.
   - Especifica:
     - Qual imagem usar.
     - Quantas réplicas (pods) rodar.
     - Labels para identificar os pods.
   - Garante que os pods estejam sempre rodando e escaláveis.

4. **Pod**
   - É a instância em execução da imagem dentro do cluster.
   - Cada pod roda um container baseado na imagem (Apache ou Nginx).
   - Se um pod cair, o Deployment recria automaticamente.

5. **Service**
   - Cria um endereço fixo para acessar os pods.
   - Usa **NodePort** para expor para fora do cluster.
   - Exemplo:
   ```bash
     - Apache → `http://<minikube-ip>:8081`
     - Nginx → `http://<minikube-ip>:8080`
   ```
6. **Usuário no navegador**
   - Digita a URL.
   - O tráfego passa pelo Service → chega ao Pod → o Pod serve o HTML.
   - Resultado: o usuário vê a página personalizada.

---

### Fluxo resumido
```bash
----------------------------------------------------------------------------------------------------------------------------
FLUXO RESUMIDO:

[ Dockerfile ] → [ Imagem Docker ] → [ Deployment ] → [ Pod ] → [ Service ] → [ Usuário ]
----------------------------------------------------------------------------------------------------------------------------
``

---

### Por que separar Apache e Nginx em Deployments e Services diferentes?

- **Modularidade**: cada aplicação tem sua própria configuração.
- **Escalabilidade independente**: é possível aumentar réplicas de um sem afetar o outro.
- **Manutenção facilitada**: se o Apache precisar de atualização, não interfere no Nginx.
- **Boas práticas do Kubernetes**: cada serviço deve ter seu próprio Deployment e Service.



==============================================================================================================================
==============================================================================================================================


## Entendendo o Cluster Kubernetes

Um cluster Kubernetes é o conjunto de máquinas que trabalham juntas para rodar e orquestrar containers. 
Ele garante que aplicações como Apache, Nginx e MariaDB rodem de forma organizada, escalável e acessível.

O cluster é formado por duas partes principais:

1. PLANO DE CONTROLE (Control Plane)
   - É o "cérebro" do cluster.
   - Responsável por decidir o que deve rodar, onde e como.
   - Componentes principais:
     - API Server → ponto de entrada para comandos (kubectl).
     - Scheduler → decide em qual nó cada pod vai rodar.
     - Controller Manager → garante que o estado desejado seja mantido.
     - etcd → banco de dados interno que guarda todo o estado do cluster.

2. NÓS (Nodes)
   - São as "máquinas operárias" que realmente rodam os containers.
   - Cada nó possui:
     - Kubelet → agente que conversa com o Control Plane.
     - Container Runtime → ex.: Docker ou containerd, que roda os containers.
     - Kube-proxy → cuida da rede e do roteamento de tráfego para os pods.

----------------------------------------------------
COMO FUNCIONA NO PROJETO WEBSOLUTIONS

- Usamos o Minikube, que cria um cluster local na máquina.
- Aplicamos os YAMLs → o Control Plane interpreta e entende o que precisa ser criado.
- O Scheduler decide em qual nó os pods vão rodar (no Minikube, há apenas um nó).
- O Kubelet cria os pods usando suas imagens Docker (apache-poc:1.0 e nginx-poc:1.0).
- O Service expõe esses pods para fora do cluster, permitindo acesso via navegador.

----------------------------------------------------
FLUXO RESUMIDO
```bash
[ Usuário aplica YAML ]
        ↓
[ Control Plane interpreta ]
        ↓
[ Scheduler decide onde rodar ]
        ↓
[ Node cria Pods com containers ]
        ↓
[ Service expõe Pods para acesso externo ]
        ↓
[ Usuário acessa via navegador ]
```
----------------------------------------------------
POR QUE PRECISAMOS DO CLUSTER?

- Escalabilidade → aumentar réplicas de Apache ou Nginx facilmente.
- Resiliência → se um pod cair, o cluster recria automaticamente.
- Isolamento → cada aplicação roda em seu próprio pod, sem conflito.
- Orquestração → o cluster garante que tudo esteja rodando conforme o desejado.


==============================================================================================================================
==============================================================================================================================



## Cenário do Cliente

Imagine que o cliente chega até a Websolutions com a seguinte necessidade:
- Ele tem um site institucional (HTML simples) → hospedado no Nginx.
- Ele tem uma aplicação dinâmica (PHP ou outro backend) que precisa de banco → hospedada no Apache + MariaDB.

----------------------------------------------------
COMO O CLIENTE USARIA NA PRÁTICA

1. Entrega dos arquivos
   - O cliente envia seus arquivos HTML/PHP para a Websolutions.
   - Esses arquivos são colocados dentro das pastas:
     - docker/nginx/html
     - docker/apache/html

2. Empacotamento em imagem Docker
   - A Websolutions gera uma imagem personalizada (nginx-poc:1.0 ou apache-poc:1.0) com os arquivos do cliente já embutidos.
   - Isso garante que o site do cliente está pronto para rodar em qualquer ambiente.

3. Deploy no Kubernetes
   - A imagem é carregada no cluster Minikube (ou em produção, num cluster Kubernetes real).
   - O Deployment cria os pods e o Service expõe a aplicação.

4. Acesso ao site
   - O cliente recebe uma URL/IP para acessar sua aplicação:
   ```bash
     - http://192.168.49.2:8080 → site no Nginx
     - http://192.168.49.2:8081 → aplicação no Apache
   - Em produção, isso seria um domínio próprio (ex.: www.clienteA.com), apontando para o LoadBalancer do cluster.
   ```
5. Banco de dados (MariaDB)
   - Se a aplicação precisar de persistência, o cliente usa o MariaDB já configurado no cluster.
   - A Websolutions fornece credenciais seguras via Secret e configurações via ConfigMap.

----------------------------------------------------
EXPERIÊNCIA DO CLIENTE

Do ponto de vista do cliente, ele não precisa se preocupar com Docker ou Kubernetes. Ele apenas:
- Entrega os arquivos da aplicação.
- Recebe um endereço (URL/IP) para acessar.
- Se precisar, recebe credenciais para o banco.

Toda a parte de infraestrutura, escalabilidade e disponibilidade é responsabilidade da Websolutions.

