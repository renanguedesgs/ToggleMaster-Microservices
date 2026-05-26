# Roteiro de Gravacao - GitOps ArgoCD Local Fase 3

> Cada passo indica: ONDE executar, QUANDO executar e O QUE FALAR no video.
> Sem AWS. Tudo roda localmente com Docker Desktop e Kubernetes.

---

## VISAO GERAL

1. Build das 5 imagens localmente com docker build
2. ArgoCD instalado no cluster local do Docker Desktop
3. ArgoCD monitora a pasta k8s/local do GitHub na branch main
4. Voce edita um manifesto e faz git push
5. ArgoCD detecta e aplica sozinho no cluster sem kubectl apply manual
6. A banca ve tudo acontecendo no painel do ArgoCD em tempo real

---

## PRE-REQUISITOS (fazer antes de gravar, nao aparece no video)

Habilitar Kubernetes no Docker Desktop:
- Abra o Docker Desktop
- Clique em Settings (engrenagem no canto superior direito)
- Clique em Kubernetes no menu lateral
- Marque Enable Kubernetes
- Clique em Apply and Restart
- Aguarde o icone do Kubernetes ficar verde no canto inferior esquerdo

---

## PARTE 1 - Confirmar que o Kubernetes esta rodando

Onde: Terminal (Git Bash ou Prompt de Comando)
Quando: Primeira coisa do video

```bash
kubectl get nodes
```

O que falar: O Docker Desktop sobe um cluster chamado docker-desktop.
STATUS Ready significa que estamos prontos para comecar.

---

## PARTE 2 - Build das imagens dos microsservicos

Onde: Mesmo terminal
Quando: Logo apos confirmar o node Ready

```bash
cd C:\Users\Renan\source\repos\ToggleMaster-Microservices
```

O que falar: Entro na pasta raiz do projeto onde estao os Dockerfiles de cada servico.

```bash
docker build -t auth-service:local ./auth-service
```

O que falar: Buildo a imagem do auth-service com a tag local.
Os manifestos usam imagePullPolicy Never, entao o Kubernetes usa essa imagem local
sem tentar baixar do ECR da AWS.

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

O que falar: Instalo o ArgoCD com o manifesto oficial. Esse comando sobe o servidor web,
o controller que monitora o Git e todos os componentes internos dele.

```bash
kubectl get pods -n argocd -w
```

O que falar: Monitoro os pods subindo. Aguardo todos ficarem STATUS Running.
Leva cerca de 2 a 3 minutos. Quando todos estiverem Running pressione Ctrl+C.

---

## PARTE 4 - Abrir o painel do ArgoCD

Onde: Terminal 1 - DEIXAR ABERTO durante todo o video
Quando: Assim que todos os pods do ArgoCD estiverem Running

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

O que falar: Faco um port-forward para acessar o painel localmente na porta 8080.
Esse terminal precisa ficar aberto. Se fechar, o painel cai.

Abra um SEGUNDO terminal para os proximos comandos.

---

## PARTE 5 - Pegar a senha e logar

Onde: Terminal 2
Quando: Logo apos o port-forward estar ativo no Terminal 1

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}"
```

O que falar: Busco a senha do admin. Fica armazenada como Secret do Kubernetes em Base64.

Copie o valor retornado e rode no Git Bash para decodificar:

```bash
echo COLE_AQUI | base64 -d
```

O que falar: Decodifico o Base64 para obter a senha legivel.

Abra o navegador em: http://localhost:8080
Se aparecer aviso de certificado: Advanced > Proceed to localhost
Login: admin | Senha: resultado do decode acima

O que falar ao mostrar o painel: Painel do ArgoCD aberto, ainda vazio.
O proximo passo e conectar ao repositorio do GitHub.

---

## PARTE 6 - Registrar o repositorio (somente se for PRIVADO)

Se o repositorio for PUBLICO, pule para a Parte 7.

Onde: Painel do ArgoCD no navegador em http://localhost:8080
Quando: Antes de aplicar o application.yaml

Passos no painel:
- Menu lateral: Settings > Repositories > Connect Repo
- Connection Method: HTTPS
- Repository URL: https://github.com/renanguedesgs/ToggleMaster-Microservices
- Coloque seu username e Personal Access Token do GitHub como senha
- Clique Connect e aguarde aparecer Successful

O que falar: Repositorio privado, entao autentico o ArgoCD para ele ler os manifestos.

---

## PARTE 7 - Conectar o ArgoCD ao repositorio

Onde: Terminal 2
Quando: Apos estar logado e ter registrado o repo se necessario

```bash
kubectl apply -f argocd/application.yaml
```

O que falar: Aplico o manifesto Application que ja esta no repositorio.
Ele instrui o ArgoCD a monitorar a pasta k8s/local na branch main
e manter o cluster sempre sincronizado com o que esta no Git.
Essa pasta tem os manifestos adaptados para o ambiente local:
imagens buildadas aqui mesmo, PostgreSQL e Redis como pods no cluster
em vez dos servicos da AWS.

Volte para o navegador em http://localhost:8080

O que falar: O cartao togglemaster apareceu no painel.
O ArgoCD ja exibe a arvore: 5 Deployments, PostgreSQL, Redis, Services e Ingress.
Status Synced confirma que o cluster esta identico ao Git.

---

## PARTE 8 - Confirmar os pods rodando

Onde: Terminal 2
Quando: Apos o status Synced aparecer no painel

```bash
kubectl get pods -n fiap-tc-f2
```

O que falar: Todos os pods Running no namespace fiap-tc-f2.
Os 5 microsservicos mais PostgreSQL e Redis rodando localmente.

```bash
kubectl get services -n fiap-tc-f2
```

O que falar: Services de cada microsservico para comunicacao interna entre eles.

```bash
kubectl get ingress -n fiap-tc-f2
```

O que falar: Ingress com as rotas /auth /flags /targeting /evaluate /analytics.

---

## PARTE 9 - DEMONSTRACAO GITOPS (cena principal para a banca)

ANTES de comecar: posicione o painel do ArgoCD e o Terminal 2 lado a lado na tela.

### 9.1 - Editar o manifesto

Onde: VS Code ou qualquer editor de texto
Quando: Com o painel visivel em paralelo na tela

Abra o arquivo: k8s/local/auth-service.yml

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

O que falar: Estou aumentando as replicas do auth-service de 1 para 3.
Isso simula o que acontece na vida real ao escalar um servico.
Importante: estou apenas editando o arquivo no repositorio.
Nenhum kubectl apply sera executado por mim.
O ArgoCD vai detectar e aplicar automaticamente.

### 9.2 - Commitar e fazer push

Onde: Terminal 2
Quando: Imediatamente apos salvar o arquivo

```bash
git add k8s/local/auth-service.yml
```

O que falar: Adiciono o arquivo ao stage do Git.

```bash
git commit -m "scale auth-service para 3 replicas"
```

O que falar: Crio o commit descrevendo a mudanca.

```bash
git push origin main
```

O que falar: Envio para a branch main do GitHub.
A partir desse momento o ArgoCD detecta que o repositorio mudou em relacao ao cluster.

### 9.3 - Observar a sincronizacao no painel

Onde: Navegador em http://localhost:8080
Quando: Logo apos o git push, fique olhando o painel

O que falar: Em alguns instantes o status muda de Synced para OutOfSync.
O ArgoCD detectou a diferenca entre o Git e o cluster.

O que falar quando sincronizar: O ArgoCD aplica automaticamente.
Status voltou para Synced.
Clicando no cartao togglemaster vemos a arvore com os 3 pods subindo em tempo real.
Nenhum kubectl apply foi executado.

### 9.4 - Confirmar no terminal

Onde: Terminal 2
Quando: Enquanto mostra o painel sincronizando

```bash
kubectl get pods -n fiap-tc-f2 -w
```

O que falar: Confirmo em tempo real os 3 pods do auth-service rodando.
So um git push foi necessario.
O Git e a unica fonte de verdade e o cluster converge automaticamente.
Pressione Ctrl+C apos mostrar os 3 pods Running.

---

## PARTE 10 - Encerramento

O que falar:
Resumindo o que demonstramos: 5 microsservicos rodando no Kubernetes local
com PostgreSQL e Redis dentro do proprio cluster.
O ArgoCD monitora a pasta k8s/local na branch main do GitHub.
Qualquer commit aplicado nessa branch e detectado e implantado automaticamente,
sem nenhuma intervencao manual.
O fluxo e: desenvolvedor edita o manifesto, faz git push,
ArgoCD detecta a divergencia, aplica a mudanca, cluster converge para o novo estado.
Isso e GitOps.

---

## CHECKLIST - Confirme antes de apertar REC

[ ] Docker Desktop com Kubernetes verde no canto inferior esquerdo
[ ] kubectl get nodes mostra STATUS Ready
[ ] docker images grep local mostra as 5 imagens buildadas
[ ] kubectl get pods -n argocd mostra todos Running
[ ] Terminal 1 com port-forward ativo: http://localhost:8080 abre o painel
[ ] Logado como admin no painel
[ ] kubectl apply -f argocd/application.yaml ja executado
[ ] Cartao togglemaster com status Synced no painel
[ ] kubectl get pods -n fiap-tc-f2 todos Running
[ ] Painel e Terminal 2 posicionados lado a lado antes de comecar a Parte 9

---

## ARQUIVOS CRIADOS PARA O AMBIENTE LOCAL

k8s/local/kustomization.yml      - lista os recursos para o ArgoCD ler via Kustomize
k8s/local/namespace.yml          - cria o namespace fiap-tc-f2
k8s/local/postgres.yml           - PostgreSQL local (substitui o RDS da AWS)
k8s/local/redis.yml              - Redis local (substitui o ElastiCache da AWS)
k8s/local/auth-service.yml       - Secret + ConfigMap + Deployment + Service
k8s/local/flag-service.yml       - Secret + ConfigMap + Deployment + Service
k8s/local/targeting-service.yml  - Secret + ConfigMap + Deployment + Service
k8s/local/evaluation-service.yml - Secret + ConfigMap + Deployment + Service
k8s/local/analytics-service.yml  - Secret + ConfigMap + Deployment + Service
k8s/local/ingress.yml            - Ingress com rotas para os 5 microsservicos

argocd/application.yaml          - aponta para k8s/local, branch main, auto-sync ativo
