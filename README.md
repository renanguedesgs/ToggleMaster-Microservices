# 🚀 Tech Challenge Fase 2 - ToggleMaster Microservices

[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://aws.amazon.com/)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)

> ⚠️ **PROJETO DIDÁTICO** - Este projeto foi desenvolvido como parte do Tech Challenge Fase 2 da Pós-Tech FIAP em Arquitetura Cloud e DevOps.

## 📋 Sobre o Projeto

Este projeto representa a evolução do ToggleMaster, uma plataforma de Feature Flags, migrando de uma arquitetura monolítica para microsserviços distribuídos orquestrados pelo Kubernetes (AWS EKS).

### 🎓 Créditos

| Componente | Autor |
|------------|-------|
| Código dos 5 microsserviços (Go/Python) | Professor da FIAP |
| Dockerfiles (multi-stage builds) | Edson Nascimento|
| Docker Compose | Edson Nascimento |
| Terraform       | Edson Nascimento |
| Manifestos Kubernetes (deployments, services, secrets, configmaps, ingress, hpa) | Edson Nascimento |
| Infraestrutura AWS (EKS, RDS, ElastiCache, DynamoDB, SQS, ECR) | Edson Nascimento |

## 🏗️ Arquitetura

O sistema é composto por 5 microsserviços:

| Serviço | Linguagem | Banco de Dados | Porta | Descrição |
|---------|-----------|----------------|-------|-----------|
| **auth-service** | Go | PostgreSQL | 8001 | Gerencia chaves de API e autenticação |
| **flag-service** | Python | PostgreSQL | 8002 | CRUD das definições de feature flags |
| **targeting-service** | Python | PostgreSQL | 8003 | Gerencia regras de segmentação |
| **evaluation-service** | Go | Redis | 8004 | Hot path de alta performance (true/false) |
| **analytics-service** | Python | DynamoDB + SQS | 8005 | Consome eventos e salva dados de análise |

### 📊 Diagrama da Arquitetura

![Alt text](./image/aws_arquitetura_togglemaster.png)


## 🛠️ Tecnologias Utilizadas

- **Orquestração:** Kubernetes (AWS EKS)
- **Containerização:** Docker com multi-stage builds
- **Banco de Dados:** PostgreSQL (AWS RDS), Redis (AWS ElastiCache), DynamoDB
- **Mensageria:** AWS SQS
- **Load Balancer:** Nginx Ingress Controller gateway fabric
- **Escalabilidade:** Horizontal Pod Autoscaler (HPA)
- **Registry:** AWS ECR

## 📁 Estrutura do Projeto

```
├── analytics-service
│   ├── app.py
│   ├── config.env
│   ├── Dockerfile
│   ├── README.md
│   └── requirements.txt
├── auth-service
│   ├── config.env
│   ├── db
│   │   └── init.sql
│   ├── Dockerfile
│   ├── go.mod
│   ├── go.sum
│   ├── handlers.go
│   ├── key.go
│   ├── main.go
│   └── README.md
├── docker-compose.yml
├── evaluation-service
│   ├── config.env
│   ├── Dockerfile
│   ├── evaluator.go
│   ├── go.mod
│   ├── go.sum
│   ├── handlers.go
│   ├── main.go
│   ├── README.md
│   ├── scripts
│   │   └── aws-sqs.sh
│   ├── sqs.go
│   └── types.go
├── flag-service
│   ├── app.py
│   ├── config.env
│   ├── db
│   │   └── init.sql
│   ├── Dockerfile
│   ├── README.md
│   └── requirements.txt
├── iac
│   └── terraform
│       ├── main.tf
│       ├── modules
│       │   ├── cluster
│       │   │   ├── data.tf
│       │   │   ├── locals.tf
│       │   │   ├── main.tf
│       │   │   ├── output.tf
│       │   │   ├── README.md
│       │   │   └── variable.tf
│       │   ├── dynamodb
│       │   │   ├── locals.tf
│       │   │   ├── main.tf
│       │   │   ├── output.tf
│       │   │   ├── README.md
│       │   │   └── variable.tf
│       │   ├── elasticache
│       │   │   ├── locals.tf
│       │   │   ├── main.tf
│       │   │   ├── output.tf
│       │   │   └── variable.tf
│       │   ├── network
│       │   │   ├── eip.tf
│       │   │   ├── igw.tf
│       │   │   ├── natgateway.tf
│       │   │   ├── output.tf
│       │   │   ├── README.md
│       │   │   ├── rt.tf
│       │   │   ├── subnet.tf
│       │   │   ├── variable.tf
│       │   │   └── vpc.tf
│       │   ├── rds
│       │   │   ├── data.tf
│       │   │   ├── locals.tf
│       │   │   ├── main.tf
│       │   │   ├── output.tf
│       │   │   ├── README.md
│       │   │   └── variable.tf
│       │   ├── registry
│       │   │   ├── locals.tf
│       │   │   ├── main.tf
│       │   │   ├── output.tf
│       │   │   ├── README.md
│       │   │   └── variable.tf
│       │   └── sqs
│       │       ├── locals.tf
│       │       ├── main.tf
│       │       ├── output.tf
│       │       ├── README.md
│       │       └── variable.tf
│       ├── output.tf
│       ├── provider.tf
│       ├── README.md
│       ├── required.tf
│       ├── terraform.tfstate
│       ├── terraform.tfstate.1773632854.backup
│       ├── terraform.tfstate.1773804334.backup
│       ├── terraform.tfstate.1773804411.backup
│       ├── terraform.tfstate.1773804417.backup
│       ├── terraform.tfstate.backup
│       ├── terraform.tfvars
│       └── variable.tf
├── image
│   └── aws_arquitetura_togglemaster.png
├── k8s
│   ├── analytics-service
│   │   ├── analytics-service-configMap.yml
│   │   ├── analytics-service-deployment.yml
│   │   ├── analytics-service-hpa.yml
│   │   ├── analytics-service-secret.yml
│   │   └── analytics-service-service.yml
│   ├── auth-service
│   │   ├── auth-service-configMap.yml
│   │   ├── auth-service-deployment.yml
│   │   ├── auth-service-hpa.yml
│   │   ├── auth-service-secret.yml
│   │   └── auth-service-service.yml
│   ├── common
│   │   ├── gatewayclass.yml
│   │   ├── ingress.yml
│   │   ├── kustomization.yml
│   │   └── namespace.yml
│   ├── evaluation-service
│   │   ├── evaluation-service-configMap.yml
│   │   ├── evaluation-service-deployment.yml
│   │   ├── evaluation-service-hpa.yml
│   │   ├── evaluation-service-secret.yml
│   │   └── evaluation-service-service.yml
│   ├── flag-service
│   │   ├── flag-service-configMap.yml
│   │   ├── flag-service-deployment.yml
│   │   ├── flag-service-hpa.yml
│   │   ├── flag-service-secret.yml
│   │   └── flag-service-service.yml
│   └── targeting-service
│       ├── targeting-service-configMap.yml
│       ├── targeting-service-deployment.yml
│       ├── targeting-service-hpa.yml
│       ├── targeting-service-secret.yml
│       └── targeting-service-service.yml
├── README.md
└── targeting-service
    ├── app.py
    ├── config.env
    ├── db
    │   └── init.sql
    ├── Dockerfile
    ├── README.md
    ├── requirements.txt
    └── terraform.tfstate

```

---

# 🚀 Como Executar o Projeto

## Opção 1: Ambiente Local (Docker Compose)

### Pré-requisitos
- Docker e Docker Compose instalados
- Git

### Passo a Passo

**1. Clone o repositório:**
```bash
git clone https://github.com/nascied/tech-challenge-fase-2-fiap.git
cd tech-challenge-fase-2-fiap
```

**2. Suba os containers:**
```bash
docker-compose up --build
```

**3. Aguarde todos os containers subirem:**
- 5 microsserviços
- 2 PostgreSQL
- 1 Redis
- 1 DynamoDB Local
- 1 LocalStack (SQS)

**4. Teste os health checks:**
```bash
curl http://localhost:8001/health  # auth-service
curl http://localhost:8002/health  # flag-service
curl http://localhost:8003/health  # targeting-service
curl http://localhost:8004/health  # evaluation-service
curl http://localhost:8005/health  # analytics-service
```

**5. Crie uma chave de API:**
```bash
curl -X POST http://localhost:8001/admin/keys \
  -H "Content-Type: application/json" \
  -H "X-Master-Key: <sua-master-key>" \
  -d '{"name": "minha-chave"}'
```

**6. Use a chave para criar uma feature flag:**
```bash
curl -X POST http://localhost:8002/flags \
  -H "Content-Type: application/json" \
  -H "X-API-Key: <sua-api-key>" \
  -d '{"name": "enable-new-feature", "enabled": true}'
```

**7. Faça uma avaliação:**
```bash
curl "http://localhost:8004/evaluate?flag_name=enable-new-feature&user_id=user-123"
```

---

## Opção 2: Ambiente AWS (EKS)

### ⚠️ Nota Importante: AWS Academy vs Conta Pessoal

| Aspecto | AWS Academy | Conta Pessoal |
|---------|-------------|---------------|
| IAM Roles | Usar `LabRole` existente | Criar roles via eksctl |
| Credenciais | Requer `AWS_SESSION_TOKEN` | Apenas Access Key e Secret Key |
| Expiração | Tokens expiram a cada ~4 horas | Sem expiração |
| Load Balancer | Funciona com LabRole | Pode ter restrições em contas novas |
| KEDA/IRSA | Não funciona | Funciona normalmente |

### Pré-requisitos
- Conta AWS (Academy ou pessoal)
- AWS CLI configurado
- kubectl instalado
- eksctl instalado (para conta pessoal)
- Docker instalado

### Passo 1: Criar infraestrutura na aws via Terraform

### ⚠️ Os módulos terraform criam os seguintes componentes 
### Necessário configurar o seu aws-cli 

| modulo | recurso | 
|---------|-------------|
| Cluster | Criar cluster eks |
| network | Cria toda a pavimentação de redes na aws tais como vpc,subnet,internet gateway, nat gateway, route table e elastic IP|
| registry | Cria um ECR container registry |
| sqs | Cria uma instancia sqs |
| elasticache | Cria um cluster elasticcache rodando redis |
| db          | Cria 3 banco de dados RDS com engine do postgres|


```bash
git clone https://github.com/nascied/tech-challenge-fase-2-fiap.git
cd tech-challenge-fase-2-fiap
```

### navega até o diretório iac/terraform
> ⚠️ **Ajsute no EKS: necessário trocar   "principal_arn = "arn:aws:iam::226226079541:role/voclabs" com respectivo arn da role que pode ser pego com o comando aws iam list-roles --output table"** 


```bash
curl https://releases.hashicorp.com/terraform/1.14.7/terraform_1.14.7_linux_amd64.zip -o terraform_1.14.7_linux_amd64.zip
unzip terraform_1.14.7_linux_amd64.zip 
chmod a+x terraform
sudo mv -v terraform  /usr/local/bin
cd iac/terraform
terraform plan
terrform apply -auto-approve
```
![Alt text](./image/terraform_apply_finish.png)

### Passo 2: Build e Push das Imagens

```bash
# Login no ECR
aws ecr get-login-password  --region us-east-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com/fiap-tc-f2-repo

# Build e push de cada serviço
docker build -t auth-service:v1 .
docker tag auth-service:v1 <account-id>.dkr.ecr.us-east-1.amazonaws.com/fiap-tc-f2-reg:auth-service-v1
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/fiap-tc-f2-reg:auth-service-v1

docker build -t flag-service:v1
docker tag flag-service:v1 <account-id>.dkr.ecr.us-east-1.amazonaws.com/fiap-tc-f2-reg:flag-service-v1
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/fiap-tc-f2-repo/flag-service:v1

docker build -t targeting-servic:v1 .
docker tag targeting-service:v1 <account-id>.dkr.ecr.us-east-1.amazonaws.com/fiap-tc-f2-reg:targeting-service-v1
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/fiap-tc-f2-repo/targeting-service:v1

docker build -t evaluation-service:v1
docker tag evaluation-service:v1 <account-id>.dkr.ecr.us-east-1.amazonaws.com/fiap-tc-f2-reg:evaluation-service-v1 
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/fiap-tc-f2-reg: evaluation-service-v1 

docker build -t analytics-service:v1
docker tag analytics-service:v1 <account-id>.dkr.ecr.us-east-1.amazonaws.com/fiap-tc-f2-reg:analytics-service-v1
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/fiap-tc-f2-reg:analytics-service-v1
```

### Passo 3: Instalar e configurar kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo mv -v kubeconfig /usr/local/bin
```

```bash
aws eks update-kubeconfig --region us-east-1 --name fiap-tc-f2-eks
kubectl get nodes  # Verificar conexão
```

### Passo 4: Instalar Componentes do Cluster

**4.1. Nginx Ingress Controller**
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.5/deploy/static/provider/aws/deploy.yaml

- validar pods do nginx controller
kubectl get po -n nginx-gateway
```

### Passo 5 Configurar Secrets

> ⚠️ **AWS Academy:** Inclua `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` e `AWS_SESSION_TOKEN` nos secrets.

Edite o arquivo `k8s/<diretorio_do_servico>/<nomeservico-service-secrets.yml` com seus endpoints:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: auth-service-secret
  namespace: fiap-tc-f2
type: Opaque
data:
  DATABASE_URL: "postgres://postgres:<senha>@<endpoint-rds>:5432/postgres"
  MASTER_KEY: "sua-master-key-secreta"

---
apiVersion: v1
kind: Secret
metadata:
  name: flag-service-secret
  namespace: fiap-tc-f2
type: Opaque
data:
  DATABASE_URL: "postgres://postgres:<senha>@<endpoint-rds>:5432/postgres"

---
apiVersion: v1
kind: Secret
metadata:
  name: targeting-service-secret
  namespace: fiap-tc-f2
type: Opaque
data:
  DATABASE_URL: "postgres://postgres:<senha>@<endpoint-rds>:5432/postgres"

---
apiVersion: v1
kind: Secret
metadata:
  name: redis-secret
  namespace: tech-challenge
type: Opaque
stringData:
  REDIS_URL: "redis://<endpoint-elasticache>:6379"

---
apiVersion: v1
kind: Secret
metadata:
  name: aws-secret
  namespace: tech-challenge
type: Opaque
stringData:
  AWS_REGION: "us-east-1"
  AWS_SQS_URL: "https://sqs.us-east-1.amazonaws.com/<account-id>/tech-challenge-events"
  AWS_DYNAMODB_TABLE: "ToggleMasterAnalytics"
  # Apenas para AWS Academy - remova se usar conta pessoal com IAM roles
  AWS_ACCESS_KEY_ID: "<sua-access-key>"
  AWS_SECRET_ACCESS_KEY: "<sua-secret-key>"
  AWS_SESSION_TOKEN: "<seu-session-token>"
```

### Passo 8: Deploy no Kubernetes

```bash
# Aplicar na ordem correta
kubectl apply -k -f k8s/common/kustomization.yml

### Passo 9: Inicializar Tabelas nos Bancos RDS

```bash
# Criar pod temporário com psql
kubectl run psql-client --rm -it --restart=Never \
  --image=postgres:15-alpine \
  --namespace=fiap-tc-f2 \
  -- bash

# Dentro do pod, conectar em cada banco e criar as tabelas:

# Auth DB
psql "postgres://postgres:<senha>@<endpoint-auth-db>:5432/postgres"
CREATE TABLE IF NOT EXISTS api_keys (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    key_hash VARCHAR(64) NOT NULL UNIQUE,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
\q

# Flag DB
psql "postgres://postgres:<senha>@<endpoint-flag-db>:5432/postgres"
CREATE TABLE IF NOT EXISTS flags (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    is_enabled BOOLEAN NOT NULL DEFAULT false,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
\q

# Targeting DB
psql "postgres://postgres:<senha>@<endpoint-targeting-db>:5432/postgres"
CREATE TABLE IF NOT EXISTS targeting_rules (
    id SERIAL PRIMARY KEY,
    flag_name VARCHAR(100) UNIQUE NOT NULL,
    is_enabled BOOLEAN NOT NULL DEFAULT true,
    rules JSONB NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
\q

exit
```

### Passo 10: Verificar o Deploy

```bash
# Ver pods
kubectl get pods -n fiap-tc-f2

# Ver services
kubectl get svc -n fiap-tc-f2

# Ver HPAs
kubectl get hpa -n fiap-tc-f2

# Pegar URL do Load Balancer
kubectl get svc -n ingress-nginx ingress-nginx-controller
```

### Passo 11: Testar os Endpoints

```bash
# Health checks
curl http://<load-balancer-url>/auth/health
curl http://<load-balancer-url>/flags/health
curl http://<load-balancer-url>/targeting/health
curl http://<load-balancer-url>/evaluate/health
curl http://<load-balancer-url>/analytics/health

# Criar chave de API
curl -X POST http://<load-balancer-url>/auth/admin/keys \
  -H "Content-Type: application/json" \
  -H "X-Master-Key: <sua-master-key>" \
  -d '{"name": "minha-chave"}'

# Criar uma feature flag
curl -X POST http://<load-balancer-url>/flags/flags \
  -H "Content-Type: application/json" \
  -H "X-API-Key: <sua-api-key>" \
  -d '{"name": "enable-new-dashboard", "enabled": true}'

# Fazer uma avaliação
curl "http://<load-balancer-url>/evaluate/evaluate?flag_name=enable-new-dashboard&user_id=user-123"

# Verificar dados no DynamoDB
aws dynamodb scan --table-name ToggleMasterAnalytics --region us-east-1
```

---

## 📈 Testando a Escalabilidade (HPA e Keda)

**1. Instale o hey (ferramenta de load testing):**
```bash
# Ubuntu/Debian
sudo apt install hey -y

# Ou via Go
go install github.com/rakyll/hey@latest

helm repo add kedacore https://kedacore.github.io/charts
helm repo update
helm install keda kedacore/keda --namespace keda --create-namespace
```

**2. Gere carga no evaluation-service:**
```bash
hey -z 120s -c 200 "http://<load-balancer-url>/evaluate/evaluate?flag_name=enable-new-dashboard&user_id=test"
```

**3. Em outro terminal, monitore o HPA:**
```bash
kubectl get hpa -n fiap-tc-f2 -w
```

**4. Monitore os pods escalando:**
```bash
kubectl get pods -n fiap-tc-f2 -w
```

Você verá o HPA aumentar o número de réplicas quando a CPU ultrapassar 70%.

---

## 🔗 Endpoints da API

| Serviço | Rota Local | Rota AWS (via Ingress) |
|---------|------------|------------------------|
| auth-service-svc | `localhost:8001/*` | `/auth/*` |
| flag-service-svc | `localhost:8002/*` | `/flags/*` |
| targeting-service-svc | `localhost:8003/*` | `/targeting/*` |
| evaluation-service-svc | `localhost:8004/*` | `/evaluate/*` |
| analytics-service-svc | `localhost:8005/*` | `/analytics/*` |

---

## 📝 Variáveis de Ambiente

### Obrigatórias para todos os ambientes

| Variável | Serviço | Descrição |
|----------|---------|-----------|
| `DATABASE_URL` | auth, flag, targeting | Connection string PostgreSQL |
| `MASTER_KEY` | auth | Chave mestra para criar API keys |
| `AUTH_SERVICE_URL` | flag, targeting | URL interna do auth-service |
| `REDIS_URL` | evaluation | Connection string Redis |
| `AWS_SQS_URL` | evaluation, analytics | URL da fila SQS |
| `AWS_DYNAMODB_TABLE` | analytics | Nome da tabela (usar `ToggleMasterAnalytics`) |
| `AWS_REGION` | evaluation, analytics | Região AWS (ex: `us-east-1`) |

### Obrigatórias apenas para AWS Academy

| Variável | Serviço | Descrição |
|----------|---------|-----------|
| `AWS_ACCESS_KEY_ID` | evaluation, analytics | Access Key do AWS Academy |
| `AWS_SECRET_ACCESS_KEY` | evaluation, analytics | Secret Key do AWS Academy |
| `AWS_SESSION_TOKEN` | evaluation, analytics | Session Token do AWS Academy (expira a cada ~4h) |

---

## 🔧 Troubleshooting
### Erro: "Unable to locate credentials"
- **Causa:** Variáveis de ambiente AWS não estão sendo injetadas no pod
- **Solução:** Verifique se o deployment inclui as variáveis do secret:
```bash
kubectl get deployment <nome> -n fiap-tc-f2 -o yaml | grep -A 30 "env:"
kubect exec <nome pod> -- env
```

### Erro: "relation does not exist"
- **Causa:** Tabelas não foram criadas no banco RDS ou endereço das variaveis estão errados
- **Solução:** Execute os scripts SQL de inicialização (ver Passo 9) revisar as variaveis config Map



---

## 👨‍💻 Autores

**Edson Leandro da Silva Nascimento**
- Pós-Tech FIAP - Arquitetura Cloud e DevOps
- Tech Challenge Fase 2

---

## 📄 Licença

Este projeto é apenas para fins educacionais como parte do programa de pós-graduação devops arquitetura Cloud da instituição FIAP.

---

# GitOps + ArgoCD - Fase 3

## Fluxo GitOps

```
Edson (Terraform)  -->  Sandro (CI/CD)               -->  Renan (ArgoCD)
Cria EKS + ECR          Build, push da imagem              ArgoCD detecta mudanca
na AWS                  Atualiza tag no deployment.yaml     Deploy automatico no cluster
```

Esta secao descreve como o ArgoCD monitora este repositorio e aplica automaticamente qualquer alteracao nos manifestos Kubernetes no cluster EKS, sem intervencao manual.

---

## Pre-requisitos

- Cluster EKS ja provisionado (via Terraform - ver secao anterior)
- `kubectl` configurado apontando para o cluster:

```bash
aws eks update-kubeconfig --region us-east-1 --name fiap-tc-f2-eks
kubectl get nodes
```

---

## Instalacao do ArgoCD no Cluster

**1. Criar o namespace e instalar o ArgoCD:**

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

**2. Aguardar todos os pods ficarem Running (aguarde ~3 minutos):**

```bash
kubectl get pods -n argocd -w
```

---

## Acessar o Painel do ArgoCD

**1. Abrir o port-forward (deixe este terminal aberto):**

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

**2. Abrir um segundo terminal para os proximos comandos.**

**3. Acessar no navegador:** http://localhost:8080

> Se aparecer aviso de certificado, clique em Avancado > Prosseguir para localhost.

**4. Obter a senha do usuario `admin`:**

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}"
```

Copie o resultado e decodifique:

```bash
# Linux / macOS
echo "<VALOR_COPIADO>" | base64 -d

# Windows (PowerShell)
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String("VALOR_COPIADO"))
```

Use `admin` como usuario e a senha decodificada para fazer login.

---

## Conectar o ArgoCD a Este Repositorio

Aplique o manifesto `Application` do ArgoCD. Ele instrui o ArgoCD a observar a pasta `k8s/common` (Kustomize) deste repositorio e sincronizar com o namespace `fiap-tc-f2` no cluster EKS:

```bash
kubectl apply -f argocd/application.yaml
```

No painel do ArgoCD aparecera o cartao **togglemaster** com todos os recursos: Deployments, Services, HPA e Ingress.

> O arquivo `argocd/application.yaml` esta neste repositorio e aponta para:
> - **Repositorio:** https://github.com/renanguedesgs/ToggleMaster-Microservices
> - **Branch:** `main`
> - **Caminho:** `k8s/common` (Kustomize)
> - **Destino:** namespace `fiap-tc-f2` no cluster EKS

---

## Demonstrar o GitOps na Pratica (para a banca)

O objetivo e mostrar uma mudanca sendo aplicada automaticamente apos o merge de um PR, sem nenhum `kubectl apply` manual.

### Exemplo: aumentar replicas do auth-service

**1. Edite o deployment** `k8s/auth-service/auth-service-deployment.yml`

Localize e altere (ou adicione) o campo `replicas` logo abaixo de `spec:`:

```yaml
# Antes
spec:
  selector:

# Depois
spec:
  replicas: 3
  selector:
```

**2. Commit e push para a branch `main`:**

```bash
git add k8s/auth-service/auth-service-deployment.yml
git commit -m "scale auth-service para 3 replicas"
git push origin main
```

**3. Observar no painel do ArgoCD (http://localhost:8080):**

- Em ate 3 minutos o ArgoCD detecta a divergencia
- O status muda de **Synced** para **OutOfSync**
- O ArgoCD aplica a mudanca automaticamente (auto-sync ativado)
- O status volta para **Synced**
- Na arvore de recursos e possivel ver o Deployment, os 3 Pods, o Service e o HPA mudando de status em tempo real

**4. Confirmar os pods pelo terminal:**

```bash
kubectl get pods -n fiap-tc-f2 -w
```

Voce vera 3 pods do `auth-service` com status `Running`.

---

## Verificacoes Rapidas

```bash
# Status geral da aplicacao no ArgoCD
kubectl get applications -n argocd

# Todos os recursos do namespace
kubectl get all -n fiap-tc-f2

# Ver o Ingress e o endereco do Load Balancer
kubectl get ingress -n fiap-tc-f2

# Ver HPAs em tempo real
kubectl get hpa -n fiap-tc-f2 -w
```

---

## Sincronizacao Manual (se necessario)

Caso queira forcar uma sincronizacao sem esperar o intervalo automatico:

```bash
# Via painel: clique no cartao "togglemaster" > botao SYNC

# Via CLI do ArgoCD (instale com: brew install argocd ou choco install argocd)
argocd app sync togglemaster
```

---

## Troubleshooting ArgoCD

| Sintoma | Causa Provavel | Solucao |
|---------|----------------|---------|
| App fica em OutOfSync indefinidamente | Auto-sync nao ativado | Painel > App Details > Enable Auto-Sync |
| ComparisonError no painel | Repositorio privado sem credenciais | Settings > Repositories > adicionar repo |
| Pods em ImagePullBackOff | Tag de imagem incorreta ou ECR sem permissao | Verificar tag no deployment e permissoes do node group |
| ArgoCD nao detecta mudanca | Branch errada no application.yaml | Confirmar que targetRevision e main |
