# Roteiro de Gravacao - GitOps ArgoCD Local Fase 3

> Cada passo indica: ONDE executar, QUANDO executar e O QUE FALAR no video.
> Sem AWS. Tudo roda localmente com Docker Desktop e Kubernetes.

---

## RESULTADO FINAL NO PAINEL DO ARGOCD

O painel vai mostrar 5 cartoes separados, um por microsservico:

- analytics-application  ->  namespace: analytics-ns  ->  path: k8s/apps/analytics-service
- auth-application       ->  namespace: auth-ns        ->  path: k8s/apps/auth-service
- evaluation-application ->  namespace: evaluation-ns  ->  path: k8s/apps/evaluation-service
- flag-application       ->  namespace: flag-ns        ->  path: k8s/apps/flag-service
- targeting-application  ->  namespace: targeting-ns   ->  path: k8s/apps/targeting-service

Cada cartao com Labels, Status Healthy + Synced, Repository, Target Revision main e Namespace proprio.

---

## PRE-REQUISITOS (fazer ANTES de gravar, nao aparece no video)

Habilitar Kubernetes no Docker Desktop:
- Abra o Docker Desktop
- Settings > Kubernetes > Enable Kubernetes > Apply and Restart
- Aguarde o icone verde no canto inferior esquerdo

---

## PARTE 1 - Confirmar que o Kubernetes esta rodando

Onde: Terminal (Git Bash)
Quando: Primeira coisa do video

```bash
kubectl get nodes
```

O que falar: Confirmo que o cluster local esta rodando com Docker Desktop.
Node chamado docker-desktop com STATUS Ready significa que estamos prontos.

---

## PARTE 2 - Build das imagens dos microsservicos

Onde: Terminal, na pasta raiz do projeto
Quando: Logo apos confirmar o node Ready

```bash
cd C:\Users\Renan\source\repos\ToggleMaster-Microservices
```

O que falar: Entro na pasta raiz do projeto onde estao os Dockerfiles de cada servico.

```bash
docker build -t auth-service:local ./auth-service
```

O que falar: Buildo a imagem do auth-service com a tag local. Os manifestos usam imagePullPolicy Never,
entao o Kubernetes usa essa imagem diretamente sem baixar do ECR da AWS.

```bash
docker build -t flag-service:local ./flag-service
```

O que falar: Imagem do flag-service.

```bash
docker build -t targeting-service:local ./targeting-service
```

O que falar: Imagem do targeting-service.

```bash
docker build -t evaluation-service:local ./evaluation-service
```

O que falar: Imagem do evaluation-service.

```bash
docker build -t analytics-service:local ./analytics-service
```

O que falar: Ultima imagem. Tenho as 5 prontas localmente.

```bash
docker images | grep local
```

O que falar: Confirmo as 5 imagens listadas com a tag local.

---

## PARTE 3 - Instalar o ArgoCD no cluster

Onde: Mesmo terminal
Quando: Apos confirmar as 5 imagens

```bash
kubectl create namespace argocd
```

O que falar: Crio o namespace dedicado para o ArgoCD, separado dos microsservicos.

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

O que falar: Instalo o ArgoCD com o manifesto oficial. Sobe o servidor web,
o controller que monitora o Git e todos os componentes internos.

```bash
kubectl get pods -n argocd -w
```

O que falar: Monitoro os pods subindo. Aguardo todos ficarem Running. Leva 2 a 3 minutos.
Quando todos estiverem Running pressione Ctrl+C.

---

## PARTE 4 - Abrir o painel do ArgoCD

Onde: Terminal 1 - DEIXAR ABERTO durante todo o video
Quando: Todos os pods do ArgoCD em Running

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

O que falar: Port-forward para acessar o painel na porta 8080 localmente.
Esse terminal precisa ficar aberto. Se fechar, o painel cai.

Abra um SEGUNDO terminal para os proximos comandos.

---

## PARTE 5 - Pegar a senha e logar

Onde: Terminal 2
Quando: Port-forward ativo no Terminal 1

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}"
```

O que falar: Busco a senha do admin. Fica armazenada como Secret do Kubernetes em Base64.

Copie o valor e rode no Git Bash:

```bash
echo COLE_AQUI | base64 -d
```

O que falar: Decodifico para obter a senha legivel.

Abra o navegador em: http://localhost:8080
Se aparecer aviso de certificado: Advanced > Proceed to localhost
Login: admin | Senha: resultado do decode

O que falar ao mostrar o painel: Painel do ArgoCD aberto, ainda vazio.
Vou aplicar as Applications agora para ele comecar a monitorar cada microsservico individualmente.

---

## PARTE 6 - Registrar o repositorio (somente se for PRIVADO)

Se o repositorio for PUBLICO, pule para a Parte 7.

Onde: Painel do ArgoCD no navegador
Quando: Antes de aplicar os YAMLs

- Settings > Repositories > Connect Repo
- Connection Method: HTTPS
- URL: https://github.com/renanguedesgs/ToggleMaster-Microservices
- Username e Personal Access Token do GitHub
- Clique Connect

O que falar: Autenticar o ArgoCD para ele conseguir ler os manifestos do repositorio privado.

---

## PARTE 7 - Remover a Application antiga (se existir)

Onde: Terminal 2
Quando: Antes de aplicar as novas Applications

Se voce ja tinha uma Application chamada togglemaster rodando, delete ela agora:

```bash
kubectl delete application togglemaster -n argocd
```

O que falar: Removo a Application anterior que usava um unico cartao para tudo.
Agora vamos subir 5 Applications separadas, uma por microsservico,
igual ao modelo que o professor vai avaliar.

Se nao tiver nenhuma Application antiga, ignore este passo e va para a Parte 8.

---

## PARTE 8 - Subir a infraestrutura base

Onde: Terminal 2
Quando: Apos remover a Application antiga

```bash
kubectl apply -f argocd/infra-application.yaml
```

O que falar: Primeiro subo a Application de infraestrutura.
Ela cria os 5 namespaces e sobe o PostgreSQL e o Redis dentro do cluster.
Os microsservicos precisam desses recursos para funcionar.

Aguarde no painel aparecer o cartao infra-application com status Synced antes de continuar.

```bash
kubectl get pods -n auth-ns
```

O que falar: Confirmo que o PostgreSQL no namespace auth-ns esta Running.

```bash
kubectl get pods -n evaluation-ns
```

O que falar: Confirmo que o Redis no namespace evaluation-ns esta Running.

---

## PARTE 9 - Aplicar as 5 Applications dos microsservicos

Onde: Terminal 2
Quando: infra-application Synced e pods de infra Running

```bash
kubectl apply -f argocd/auth-application.yaml
kubectl apply -f argocd/flag-application.yaml
kubectl apply -f argocd/targeting-application.yaml
kubectl apply -f argocd/evaluation-application.yaml
kubectl apply -f argocd/analytics-application.yaml
```

O que falar: Aplico as 5 Applications de uma vez, uma por microsservico.
Cada uma aponta para seu proprio caminho no Git e seu proprio namespace isolado.
O ArgoCD vai sincronizar cada uma de forma independente e paralela.

Volte para o navegador em http://localhost:8080

O que falar: Agora vemos 5 cartoes no painel:
analytics-application, auth-application, evaluation-application, flag-application e targeting-application.
Cada cartao mostra o label do servico, o namespace proprio, o path no Git e o status.
Em alguns instantes todos ficam Healthy e Synced, igualzinho a referencia.

---

## PARTE 10 - Confirmar os pods de cada servico

Onde: Terminal 2
Quando: Todos os 5 cartoes Synced no painel

```bash
kubectl get pods -n auth-ns
```

O que falar: Pod do auth-service rodando no namespace auth-ns.

```bash
kubectl get pods -n flag-ns
```

O que falar: Pod do flag-service no namespace flag-ns.

```bash
kubectl get pods -n targeting-ns
```

O que falar: Pod do targeting-service no namespace targeting-ns.

```bash
kubectl get pods -n evaluation-ns
```

O que falar: Pod do evaluation-service junto com o Redis no namespace evaluation-ns.

```bash
kubectl get pods -n analytics-ns
```

O que falar: Pod do analytics-service no namespace analytics-ns.
Cada microsservico isolado no seu proprio namespace. Boa pratica de separacao no Kubernetes.

---

## PARTE 11 - DEMONSTRACAO GITOPS (cena principal para a banca)

ANTES de comecar: posicione o painel do ArgoCD e o Terminal 2 lado a lado na tela.
Clique no cartao auth-application para deixar a arvore de recursos visivel.

### 11.1 - Editar o manifesto

Onde: VS Code
Quando: Com o painel e a arvore do auth-application visiveis

Abra o arquivo: k8s/apps/auth-service/auth-service.yml

Encontre o bloco do Deployment e mude replicas de 1 para 3:

```yaml
# Como esta
spec:
  replicas: 1
  selector:

# Como deve ficar
spec:
  replicas: 3
  selector:
```

Salve com Ctrl+S.

O que falar: Vou escalar o auth-service de 1 para 3 replicas.
Estou apenas editando o arquivo no repositorio local.
Nenhum kubectl apply sera feito. O ArgoCD vai detectar e aplicar automaticamente.

### 11.2 - Commitar e fazer push

Onde: Terminal 2
Quando: Imediatamente apos salvar o arquivo

```bash
git add k8s/apps/auth-service/auth-service.yml
```

O que falar: Adiciono o arquivo ao stage do Git.

```bash
git commit -m "scale auth-service para 3 replicas"
```

O que falar: Commit descrevendo a mudanca.

```bash
git push origin main
```

O que falar: Envio para a branch main. A partir daqui o ArgoCD detecta a divergencia.

### 11.3 - Observar no painel

Onde: Navegador em http://localhost:8080
Quando: Logo apos o git push, fique olhando o cartao auth-application

O que falar: O status do cartao auth-application muda para OutOfSync.
O ArgoCD detectou que o cluster tem 1 replica mas o Git diz 3.
Agora ele aplica automaticamente. O status volta para Synced.
Na arvore vemos os 3 pods do auth-service subindo em tempo real,
junto com o Service e o HPA. Nenhum kubectl apply foi executado.

### 11.4 - Confirmar no terminal

Onde: Terminal 2
Quando: Enquanto mostra o painel sincronizando

```bash
kubectl get pods -n auth-ns -w
```

O que falar: Confirmo os 3 pods do auth-service rodando no namespace auth-ns.
So um git push foi necessario.
O Git e a unica fonte de verdade e o cluster converge automaticamente.
Pressione Ctrl+C apos mostrar os 3 pods Running.

---

## PARTE 12 - Encerramento

O que falar:
Demonstramos o GitOps com ArgoCD rodando localmente.
Temos 5 microsservicos, cada um com sua propria Application no ArgoCD,
seu proprio namespace e seus recursos completamente isolados.
Qualquer mudanca commitada na branch main e detectada e aplicada automaticamente
sem nenhuma intervencao manual.
O Git e a unica fonte de verdade do estado do cluster.

---

## CHECKLIST - Confirme antes de apertar REC

- Docker Desktop com Kubernetes verde no canto inferior esquerdo
- kubectl get nodes retorna STATUS Ready
- docker images grep local mostra as 5 imagens buildadas
- kubectl get pods -n argocd todos Running
- Terminal 1 com port-forward ativo: http://localhost:8080 abre o painel
- Logado como admin no painel
- Application antiga deletada se existia (kubectl delete application togglemaster -n argocd)
- infra-application aplicada e Synced no painel
- PostgreSQL Running em auth-ns, flag-ns e targeting-ns
- Redis Running em evaluation-ns
- 5 Applications aplicadas e todas Synced no painel
- Todos os pods Running em seus respectivos namespaces
- Cartao auth-application aberto com arvore visivel antes da Parte 11
- Painel e Terminal 2 posicionados lado a lado

---

## ESTRUTURA DOS ARQUIVOS

argocd/
  infra-application.yaml        namespaces + PostgreSQL + Redis
  auth-application.yaml         auth-service no namespace auth-ns
  flag-application.yaml         flag-service no namespace flag-ns
  targeting-application.yaml    targeting-service no namespace targeting-ns
  evaluation-application.yaml   evaluation-service no namespace evaluation-ns
  analytics-application.yaml    analytics-service no namespace analytics-ns

k8s/apps/
  infra/
    kustomization.yml
    namespace.yml                cria os 5 namespaces
    postgres.yml                 PostgreSQL em auth-ns, flag-ns e targeting-ns
    redis.yml                    Redis em evaluation-ns
  auth-service/
    kustomization.yml
    auth-service.yml             Secret + ConfigMap + Deployment + Service + HPA
  flag-service/
    kustomization.yml
    flag-service.yml
  targeting-service/
    kustomization.yml
    targeting-service.yml
  evaluation-service/
    kustomization.yml
    evaluation-service.yml
  analytics-service/
    kustomization.yml
    analytics-service.yml
