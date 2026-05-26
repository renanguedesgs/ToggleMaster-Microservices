# Roteiro de Gravacao - GitOps + ArgoCD (Fase 3)

> Leia este roteiro antes de gravar. Cada passo indica **onde executar**, **quando executar** e uma **explicacao curta** para voce falar no video.

---

## PARTE 1 — Conectar ao cluster EKS

**Onde:** Terminal (qualquer terminal — bash, zsh ou CMD)
**Quando:** Logo no inicio do video, antes de qualquer outro comando

```bash
aws eks update-kubeconfig --region us-east-1 --name fiap-tc-f2-eks
```

**O que falar:** "Aqui estou configurando o kubectl para apontar para o nosso cluster EKS na AWS. Esse comando baixa as credenciais do cluster e salva no arquivo de configuracao local."

```bash
kubectl get nodes
```

**O que falar:** "Verifico que os nodes do cluster estao prontos. Se aparecer STATUS Ready, estamos conectados corretamente."

---

## PARTE 2 — Instalar o ArgoCD no cluster

**Onde:** Mesmo terminal da Parte 1
**Quando:** Logo apos confirmar que os nodes estao Ready

```bash
kubectl create namespace argocd
```

**O que falar:** "Crio o namespace dedicado para o ArgoCD. Ele vai rodar isolado dos nossos servicos de aplicacao."

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

**O que falar:** "Instalo o ArgoCD aplicando o manifesto oficial diretamente do repositorio deles. Esse comando sobe todos os componentes: servidor, controller, dex e redis internos."

```bash
kubectl get pods -n argocd -w
```

**O que falar:** "Aqui monitoro os pods do ArgoCD subindo em tempo real. Aguardo todos ficarem com STATUS Running antes de continuar. Isso leva em torno de 2 a 3 minutos."

> Aguarde todos os pods aparecerem como Running, depois pressione Ctrl+C para sair do watch.

---

## PARTE 3 — Acessar o painel web do ArgoCD

**Onde:** Terminal 1 (deixar aberto e nao fechar)
**Quando:** Assim que todos os pods do ArgoCD estiverem Running

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

**O que falar:** "Faco um port-forward para acessar o painel do ArgoCD localmente na porta 8080. Esse terminal precisa ficar aberto enquanto uso o painel."

> Abra um segundo terminal para continuar com os proximos comandos. Nao feche o Terminal 1.

---

## PARTE 4 — Pegar a senha do admin

**Onde:** Terminal 2 (segundo terminal que voce acabou de abrir)
**Quando:** Logo apos o port-forward estar rodando no Terminal 1

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}"
```

**O que falar:** "Busco a senha inicial do usuario admin. Ela fica armazenada como um Secret do Kubernetes em formato Base64."

Copie o valor retornado e rode o comando abaixo para decodificar:

```bash
# Linux ou macOS
echo "VALOR_COPIADO" | base64 -d
```

**O que falar:** "Decodifico o valor de Base64 para texto legivel. O resultado e a senha que vou usar para logar no painel."

> Acesse no navegador: **http://localhost:8080**
> Login: `admin`
> Senha: resultado do comando acima

**O que falar ao abrir o painel:** "Aqui esta o painel do ArgoCD. Por enquanto nao temos nenhuma aplicacao cadastrada. O proximo passo e conectar o ArgoCD ao nosso repositorio do GitHub."

---

## PARTE 5 — Conectar o ArgoCD ao repositorio (aplicar o Application)

**Onde:** Terminal 2
**Quando:** Apos estar logado no painel do ArgoCD

```bash
kubectl apply -f argocd/application.yaml
```

**O que falar:** "Aplico o manifesto Application que ja esta no nosso repositorio. Ele diz para o ArgoCD: monitore o repositorio do GitHub, branch main, pasta k8s/common, e mantenha tudo sincronizado no namespace fiap-tc-f2 do cluster."

> Volte para o navegador em http://localhost:8080

**O que falar:** "Veja que o cartao togglemaster apareceu no painel. O ArgoCD ja esta lendo o repositorio e exibindo a arvore de recursos — Deployments, Services, HPA e Ingress. O status Synced significa que o cluster esta igual ao que esta no Git."

---

## PARTE 6 — Verificar os recursos no cluster

**Onde:** Terminal 2
**Quando:** Apos o status aparecer Synced no painel

```bash
kubectl get pods -n fiap-tc-f2
```

**O que falar:** "Confirmo que todos os pods dos nossos microsservicos estao rodando no namespace correto."

```bash
kubectl get services -n fiap-tc-f2
```

**O que falar:** "Vejo os Services criados para cada microsservico."

```bash
kubectl get ingress -n fiap-tc-f2
```

**O que falar:** "Aqui esta o Ingress, que e o ponto de entrada unico para todas as rotas da aplicacao."

```bash
kubectl get hpa -n fiap-tc-f2
```

**O que falar:** "E aqui o HPA — Horizontal Pod Autoscaler — configurado para escalar os pods automaticamente conforme a carga de CPU."

---

## PARTE 7 — Demonstrar o GitOps na pratica (parte principal para a banca)

> Esta e a parte mais importante do video. Mostra a mudanca automatica sem nenhum kubectl apply manual.

**Onde:** Editor de texto (VS Code, Bloco de Notas, qualquer editor)
**Quando:** Com o painel do ArgoCD visivel no navegador em paralelo

### 7.1 — Abrir e editar o arquivo de deployment

Abra o arquivo: `k8s/auth-service/auth-service-deployment.yml`

Localize a linha que começa com `spec:` e adicione `replicas: 3` logo abaixo:

```yaml
# Como estava
spec:
  selector:

# Como deve ficar
spec:
  replicas: 3
  selector:
```

Salve o arquivo.

**O que falar:** "Aqui estou editando o manifesto do auth-service para aumentar o numero de replicas de 1 para 3. Estou so editando o arquivo no repositorio — nao vou rodar nenhum kubectl apply. O GitOps vai cuidar disso."

---

### 7.2 — Commitar e fazer push

**Onde:** Terminal 2
**Quando:** Imediatamente apos salvar o arquivo

```bash
git add k8s/auth-service/auth-service-deployment.yml
```

**O que falar:** "Adiciono o arquivo modificado para o stage do git."

```bash
git commit -m "scale auth-service para 3 replicas"
```

**O que falar:** "Crio o commit com a descricao da mudanca."

```bash
git push origin main
```

**O que falar:** "Envio a mudanca para a branch main do GitHub. A partir daqui o ArgoCD vai detectar que o repositorio mudou."

---

### 7.3 — Observar a sincronizacao no painel do ArgoCD

**Onde:** Navegador em http://localhost:8080
**Quando:** Imediatamente apos o git push

**O que falar:** "Veja que em alguns instantes o status do cartao togglemaster muda de Synced para OutOfSync — isso significa que o ArgoCD detectou que o cluster esta diferente do que esta no Git."

**O que falar em seguida:** "O ArgoCD entao aplica a mudanca automaticamente, sem eu precisar rodar nenhum comando. O status volta para Synced e se clicarmos no cartao podemos ver a arvore de recursos com os 3 pods do auth-service subindo em tempo real."

---

### 7.4 — Confirmar os pods pelo terminal

**Onde:** Terminal 2
**Quando:** Enquanto mostra o painel sincronizando

```bash
kubectl get pods -n fiap-tc-f2 -w
```

**O que falar:** "Aqui no terminal confirmo em tempo real os pods do auth-service. Veja que o ArgoCD criou os 3 pods automaticamente so com o push no Git — esse e o GitOps funcionando."

> Pressione Ctrl+C para sair do watch apos mostrar os 3 pods Running.

---

## RESUMO DO FLUXO (fale isso ao final do video)

```
1. Desenvolvedor edita o deployment.yaml
2. git push para a branch main
3. ArgoCD detecta a mudanca (OutOfSync)
4. ArgoCD aplica automaticamente no cluster EKS
5. Status volta para Synced
6. Pods novos sobem sem nenhum kubectl apply manual
```

---

## Checklist antes de gravar

- [ ] Cluster EKS esta rodando (`kubectl get nodes` retorna Ready)
- [ ] ArgoCD esta instalado e todos os pods em Running
- [ ] Port-forward esta ativo no Terminal 1 (`http://localhost:8080` abre o painel)
- [ ] Logado no painel como admin
- [ ] `kubectl apply -f argocd/application.yaml` ja foi executado
- [ ] Cartao togglemaster aparece com status Synced no painel
- [ ] Abrir o painel e o terminal lado a lado na tela antes de comecar a Parte 7
