# ☸️ Kubernetes Capture The Flag

> **Desafio final — Kubernetes: Fundamentos e Prática**

Um deploy realizado durante uma manutenção deixou uma aplicação Kubernetes em um estado inconsistente.

Sua missão é investigar o cluster, corrigir os problemas encontrados e capturar as **8 flags** espalhadas pelo ambiente.

Você terá **60 minutos**.

---

## 🎯 Objetivo

Use os conhecimentos trabalhados durante o curso para investigar e recuperar a aplicação.

Durante o desafio você passará por conceitos como:

`Pods` · `Deployments` · `ReplicaSets` · `ConfigMaps` · `Secrets` · `Services` · `Probes` · `PVC/PV` · `Resources`

### Informações do ambiente

| | |
|---|---|
| ⏱️ Tempo | **60 minutos** |
| 🚩 Flags | **8** |
| 🏆 Pontuação máxima | **100 pontos** |
| 📦 Namespace | `ctf` |

---

## 🚀 Preparação

Execute o comando fornecido pelo instrutor para preparar o ambiente.

Quando o processo terminar, comece conhecendo o cluster:

```bash
kubectl -n ctf get all
```

A partir daqui, a investigação é sua. 🔎

---

## 📜 Regras

- Não delete o namespace `ctf`.
- Não recrie todo o ambiente do zero.
- Corrija os recursos Kubernetes existentes.
- Você pode utilizar qualquer comando `kubectl`.
- As flags **não precisam ser encontradas em ordem**.
- Você pode consultar documentação e o material utilizado durante o curso.
- As dicas abaixo podem ser utilizadas quando necessário.
- Quando acreditar que concluiu todo o desafio, execute:

```bash
ctf-check
```

---

# 🚩 FLAG 1 — Configuração

**5 pontos**

Uma configuração **não sensível** da aplicação contém a primeira flag.

Encontre-a.

<details>
<summary>💡 Dica 1</summary>

Qual recurso Kubernetes utilizamos para separar **configurações não sensíveis** da imagem da aplicação?

</details>

<details>
<summary>💡 Dica 2</summary>

Liste os recursos desse tipo existentes no namespace `ctf`.

</details>

<details>
<summary>💡 Dica 3</summary>

O `kubectl` consegue mostrar tanto uma descrição do recurso quanto sua representação em YAML.

</details>

**Conceitos:** `ConfigMap` · `kubectl get` · `kubectl describe` · YAML

---

# 🚩 FLAG 2 — Informação Sensível

**5 pontos**

Existe uma informação sensível armazenada no ambiente que contém a segunda flag.

> **Lembre-se:** codificação não é criptografia.

<details>
<summary>💡 Dica 1</summary>

Qual recurso Kubernetes utilizamos para armazenar informações sensíveis?

</details>

<details>
<summary>💡 Dica 2</summary>

Liste os recursos desse tipo existentes no namespace `ctf`.

</details>

<details>
<summary>💡 Dica 3</summary>

Observe o campo `data`.

O conteúdo pode não estar diretamente legível.

Lembre-se do que vimos sobre **Base64**.

</details>

**Conceitos:** `Secret` · Base64 · `kubectl get` · YAML

---

# 🚩 FLAG 3 — Estado Desejado

**10 pontos**

A aplicação deveria possuir:

```text
2 réplicas
```

Porém, o estado atual do cluster não atende a esse requisito.

Corrija a configuração.

Depois de resolver o problema, procure pela variável:

```text
FLAG_3
```

dentro de um dos containers da aplicação.

<details>
<summary>💡 Dica 1</summary>

Qual recurso Kubernetes define quantas réplicas de uma aplicação devem existir?

</details>

<details>
<summary>💡 Dica 2</summary>

Observe a relação:

```text
Deployment
    │
    ▼
ReplicaSet
    │
    ▼
   Pods
```

Compare o **estado desejado** com o **estado atual**.

</details>

<details>
<summary>💡 Dica 3</summary>

Depois de corrigir as réplicas, lembre-se de que `kubectl exec` permite executar comandos dentro de um container.

Uma variável de ambiente pode ser visualizada de dentro dele.

</details>

**Conceitos:** `Deployment` · `ReplicaSet` · replicas · Desired State · `kubectl exec`

---

# 🚩 FLAG 4 — Running ≠ Ready

**15 pontos**

Os containers estão executando, mas a aplicação **não está pronta para receber tráfego**.

Seu objetivo é fazer os dois Pods da aplicação chegarem ao estado:

```text
READY   STATUS
1/1     Running
1/1     Running
```

Quando os dois Pods estiverem `Ready`, você conquistou a quarta flag:

```text
FLAG{04_RUNNING_IS_NOT_READY}
```

<details>
<summary>💡 Dica 1</summary>

Um container estar `Running` significa necessariamente que a aplicação está pronta para receber requisições?

</details>

<details>
<summary>💡 Dica 2</summary>

Investigue os detalhes de um dos Pods.

Os **Events** normalmente fornecem pistas importantes sobre problemas de saúde da aplicação.

</details>

<details>
<summary>💡 Dica 3</summary>

Durante o curso vimos diferentes tipos de Probes.

Qual delas responde à pergunta:

> **"Esta aplicação está pronta para receber tráfego?"**

</details>

**Conceitos:** `readinessProbe` · Pod Conditions · Events · `kubectl describe`

---

# 🚩 FLAG 5 — Onde estão meus Pods?

**15 pontos**

Agora os Pods podem estar saudáveis, mas existe outro problema.

O Service:

```text
web
```

não consegue encontrar a aplicação.

Faça o Service possuir **endpoints válidos**.

Quando a comunicação estiver funcionando, a flag estará disponível em:

```text
http://web/flag5.txt
```

> ⚠️ Esse endereço deve ser acessado **de dentro do cluster**.

<details>
<summary>💡 Dica 1</summary>

Como um `Service` determina quais Pods devem receber suas requisições?

</details>

<details>
<summary>💡 Dica 2</summary>

Compare:

```text
Labels dos Pods
       ↕
Selector do Service
```

Eles precisam ser compatíveis.

</details>

<details>
<summary>💡 Dica 3</summary>

Verifique se o Service possui endpoints.

Para testar DNS e HTTP dentro do cluster, você pode criar temporariamente um Pod cliente.

Durante o curso utilizamos imagens pequenas como `busybox` para esse tipo de teste.

</details>

**Conceitos:** `Service` · `ClusterIP` · Labels · Selectors · Endpoints · DNS interno

---

# 🚩 FLAG 6 — Os Pods são descartáveis, os dados não

**15 pontos**

Existe uma informação persistente armazenada no cluster.

Sua aplicação precisa utilizar o armazenamento existente e disponibilizá-lo no diretório:

```text
/dados
```

Quando conseguir, procure pelo arquivo:

```text
/dados/flag6.txt
```

<details>
<summary>💡 Dica 1</summary>

Procure pelos recursos de armazenamento existentes no namespace `ctf`.

</details>

<details>
<summary>💡 Dica 2</summary>

Um `PersistentVolumeClaim` existir não significa que o container automaticamente consegue acessar seus arquivos.

O Deployment precisa declarar que deseja utilizar esse armazenamento.

</details>

<details>
<summary>💡 Dica 3</summary>

Revise a relação:

```text
Container
    │
    ▼
volumeMount
    │
    ▼
 volume
    │
    ▼
   PVC
    │
    ▼
    PV
```

O PVC existente precisa ser utilizado pelo Pod e montado em `/dados`.

</details>

**Conceitos:** `PVC` · `PV` · `StorageClass` · Volumes · `volumeMounts`

---

# 🚩 FLAG 7 — Recursos

**15 pontos**

A aplicação não atende ao padrão de recursos definido pela equipe de arquitetura.

O container deve possuir:

| Recurso | Request | Limit |
|---|---:|---:|
| CPU | `50m` | `200m` |
| Memória | `32Mi` | `128Mi` |

Corrija o Deployment.

Depois procure pela variável:

```text
FLAG_7
```

dentro de um dos containers da aplicação.

<details>
<summary>💡 Dica 1</summary>

`requests` e `limits` possuem funções diferentes.

Um deles representa o recurso considerado necessário para executar o workload, enquanto o outro estabelece um limite de utilização.

</details>

<details>
<summary>💡 Dica 2</summary>

Observe a configuração atual do container no Deployment.

Parte da configuração de recursos já pode estar correta.

</details>

<details>
<summary>💡 Dica 3</summary>

Depois de modificar o **Pod Template** de um Deployment, observe o que acontece com:

```text
Deployment
    │
    ▼
ReplicaSet
    │
    ▼
   Pods
```

Depois do rollout, entre em um dos novos containers e procure `FLAG_7`.

</details>

**Conceitos:** `requests` · `limits` · CPU · Memória · Rollout

---

# 🏁 FLAG 8 — Recuperação Completa

**20 pontos**

A última flag representa a recuperação completa do ambiente.

Quando acreditar que corrigiu **todos os problemas**, execute:

```bash
ctf-check
```

O validador informará quantos requisitos foram atendidos.

Exemplo:

```text
Requisitos atendidos: 7/10
```

O checker **não informará quais requisitos ainda estão incorretos**.

Quando conseguir:

```text
Requisitos atendidos: 10/10
```

a última flag será liberada. 🏁

<details>
<summary>💡 Dica 1</summary>

Não pense somente nos Pods.

Revise toda a aplicação e os recursos que fazem parte dela.

</details>

<details>
<summary>💡 Dica 2</summary>

Confira os conceitos trabalhados nas flags anteriores:

- Deployment
- Réplicas
- Pods
- Probes
- Service
- Endpoints
- PVC
- Volumes
- Requests
- Limits

</details>

<details>
<summary>💡 Dica 3</summary>

O estado final esperado pode ser representado assim:

```text
                ConfigMap
                    │
                    │
 Secret ─────► Deployment
                    │
               ReplicaSet
                    │
             ┌──────┴──────┐
             ▼             ▼
           Pod 1         Pod 2
           Ready         Ready
             │             │
             └──────┬──────┘
                    │
                 Service
                    │
                Endpoints


               Deployment
                    │
                    ▼
               volumeMount
                    │
                    ▼
                   PVC
                    │
                    ▼
                    PV
```

Se alguma parte dessa arquitetura ainda não estiver funcionando corretamente, continue investigando.

</details>

**Conceitos:** integração · troubleshooting · estado desejado · validação

---

# 🏆 Pontuação

| Flag | Desafio | Pontos |
|---|---|---:|
| 🚩 FLAG 1 | ConfigMap | **5** |
| 🚩 FLAG 2 | Secret | **5** |
| 🚩 FLAG 3 | Deployment / ReplicaSet | **10** |
| 🚩 FLAG 4 | Readiness Probe | **15** |
| 🚩 FLAG 5 | Service / Selector | **15** |
| 🚩 FLAG 6 | PVC / PV | **15** |
| 🚩 FLAG 7 | Resources | **15** |
| 🏁 FLAG 8 | Recuperação completa | **20** |
| | **TOTAL** | **100** |

---

## 🧰 Comandos que podem ajudar

Você não precisa utilizar todos eles.

```bash
kubectl get
kubectl describe
kubectl logs
kubectl exec
kubectl edit
kubectl apply
kubectl scale
kubectl rollout
```

Não esqueça que vários recursos podem ser visualizados em YAML:

```bash
kubectl get <recurso> <nome> -o yaml
```

E que você está trabalhando no namespace:

```text
ctf
```

---

## 🏁 Terminou?

Execute:

```bash
ctf-check
```

Se o ambiente estiver **10/10**, você concluiu o Kubernetes Capture The Flag.

---

<div align="center">

### ☸️ Boa sorte!

**Observe → Investigue → Entenda → Corrija → Valide**

</div>

---

# 📚 Referência rápida de `kubectl`

Travou ou esqueceu a sintaxe de algum comando?

Abaixo estão alguns dos comandos utilizados durante o curso. Esta seção funciona como uma **referência rápida** durante o desafio.

> 💡 Os comandos abaixo não estão necessariamente na ordem em que serão utilizados e nem todos serão necessários.

---

<details>
<summary>🔎 Visualizar recursos</summary>

### Listar recursos

```bash
kubectl get pods
kubectl get deployments
kubectl get replicasets
kubectl get services
kubectl get configmaps
kubectl get secrets
kubectl get pvc
kubectl get pv
```

Utilizando o namespace do desafio:

```bash
kubectl -n ctf get pods
```

Listar vários recursos:

```bash
kubectl -n ctf get pods,deploy,rs,svc
```

Visão geral:

```bash
kubectl -n ctf get all
```

Mais informações:

```bash
kubectl -n ctf get pods -o wide
```

Mostrar labels:

```bash
kubectl -n ctf get pods --show-labels
```

Filtrar por label:

```bash
kubectl -n ctf get pods -l app=web
```

---

### Visualizar YAML de um recurso

```bash
kubectl -n ctf get pod <nome> -o yaml
```

Exemplos:

```bash
kubectl -n ctf get deployment web -o yaml

kubectl -n ctf get service web -o yaml
```

</details>

---

<details>
<summary>🩺 Investigar problemas</summary>

### Describe

Mostra configuração, estado e eventos relacionados ao recurso.

```bash
kubectl -n ctf describe pod <nome>
```

Também pode ser utilizado com outros recursos:

```bash
kubectl -n ctf describe deployment web

kubectl -n ctf describe service web

kubectl -n ctf describe pvc app-data
```

---

### Logs

```bash
kubectl -n ctf logs <pod>
```

Para acompanhar continuamente:

```bash
kubectl -n ctf logs -f <pod>
```

Quando houver mais de um container:

```bash
kubectl -n ctf logs <pod> -c <container>
```

---

### Observar mudanças

```bash
kubectl -n ctf get pods -w
```

Pressione `CTRL+C` para sair.

</details>

---

<details>
<summary>💻 Executar comandos dentro de containers</summary>

### Executar um comando

```bash
kubectl -n ctf exec <pod> -- <comando>
```

Exemplo:

```bash
kubectl -n ctf exec <pod> -- env
```

---

### Abrir um shell

```bash
kubectl -n ctf exec -it <pod> -- sh
```

Depois você poderá executar comandos diretamente dentro do container:

```bash
env

ls

cat /caminho/arquivo
```

Para sair:

```bash
exit
```

---

### Executar utilizando um Deployment

Também é possível utilizar o nome do Deployment:

```bash
kubectl -n ctf exec deploy/web -- env
```

</details>

---

<details>
<summary>⚙️ Deployment e ReplicaSet</summary>

### Visualizar Deployment

```bash
kubectl -n ctf get deployment
```

```bash
kubectl -n ctf describe deployment web
```

---

### Alterar quantidade de réplicas

```bash
kubectl -n ctf scale deployment web --replicas=<quantidade>
```

---

### Acompanhar rollout

```bash
kubectl -n ctf rollout status deployment/web
```

Histórico:

```bash
kubectl -n ctf rollout history deployment/web
```

Desfazer última alteração:

```bash
kubectl -n ctf rollout undo deployment/web
```

---

### Reiniciar os Pods de um Deployment

```bash
kubectl -n ctf rollout restart deployment/web
```

</details>

---

<details>
<summary>✏️ Editar recursos</summary>

### Editar diretamente no cluster

```bash
kubectl -n ctf edit deployment web
```

Outros exemplos:

```bash
kubectl -n ctf edit service web

kubectl -n ctf edit configmap <nome>
```

Ao salvar o arquivo, o Kubernetes recebe a nova configuração.

---

### Aplicar um arquivo YAML

```bash
kubectl apply -f arquivo.yaml
```

---

### Verificar alterações no Deployment

```bash
kubectl -n ctf rollout status deployment/web
```

E acompanhe os Pods:

```bash
kubectl -n ctf get pods -w
```

</details>

---

<details>
<summary>🌐 Services, Labels e Endpoints</summary>

### Listar Services

```bash
kubectl -n ctf get services
```

Forma abreviada:

```bash
kubectl -n ctf get svc
```

---

### Visualizar labels dos Pods

```bash
kubectl -n ctf get pods --show-labels
```

---

### Visualizar configuração do Service

```bash
kubectl -n ctf get service web -o yaml
```

Ou:

```bash
kubectl -n ctf describe service web
```

---

### Verificar Endpoints

```bash
kubectl -n ctf get endpoints
```

Ou para um Service específico:

```bash
kubectl -n ctf get endpoints web
```

Também é possível visualizar EndpointSlices:

```bash
kubectl -n ctf get endpointslices
```

---

### Testar um Service de dentro do cluster

Você pode criar temporariamente um Pod cliente:

```bash
kubectl -n ctf run cliente \
  --rm -it \
  --restart=Never \
  --image=busybox:1.37 \
  -- sh
```

Dentro dele, por exemplo:

```bash
wget -qO- http://<service>
```

Para sair:

```bash
exit
```

O Pod temporário será removido automaticamente.

</details>

---

<details>
<summary>🗂️ ConfigMaps e Secrets</summary>

### ConfigMaps

Listar:

```bash
kubectl -n ctf get configmaps
```

Visualizar:

```bash
kubectl -n ctf describe configmap <nome>
```

Ou:

```bash
kubectl -n ctf get configmap <nome> -o yaml
```

---

### Secrets

Listar:

```bash
kubectl -n ctf get secrets
```

Visualizar:

```bash
kubectl -n ctf get secret <nome> -o yaml
```

> ⚠️ Lembre-se: valores presentes em `data` normalmente estão codificados em Base64.

No Linux, um valor Base64 pode ser decodificado com:

```bash
echo '<valor>' | base64 -d
```

Também é possível extrair um campo utilizando JSONPath:

```bash
kubectl -n ctf get secret <nome> \
  -o jsonpath='{.data.<chave>}'
```

</details>

---

<details>
<summary>💾 PVC, PV e Storage</summary>

### PersistentVolumeClaims

```bash
kubectl -n ctf get pvc
```

Detalhes:

```bash
kubectl -n ctf describe pvc <nome>
```

---

### PersistentVolumes

```bash
kubectl get pv
```

> 💡 PV é um recurso de cluster e não pertence a um namespace.

---

### StorageClasses

```bash
kubectl get storageclass
```

Forma abreviada:

```bash
kubectl get sc
```

---

### Visualizar configuração de volumes de um Pod

```bash
kubectl -n ctf get pod <nome> -o yaml
```

Ou:

```bash
kubectl -n ctf describe pod <nome>
```

Procure informações relacionadas a:

```text
Volumes
Mounts
PersistentVolumeClaim
```

</details>

---

<details>
<summary>❤️ Probes e estado dos Pods</summary>

### Estado dos Pods

```bash
kubectl -n ctf get pods
```

Observe principalmente:

```text
READY
STATUS
RESTARTS
```

---

### Investigar uma Probe

```bash
kubectl -n ctf describe pod <nome>
```

Procure:

```text
Readiness
Liveness
Events
```

---

### Ver configuração declarada no Deployment

```bash
kubectl -n ctf get deployment web -o yaml
```

Procure pelos campos:

```yaml
readinessProbe:

livenessProbe:
```

</details>

---

<details>
<summary>📊 CPU e Memória</summary>

### Visualizar resources do Deployment

```bash
kubectl -n ctf describe deployment web
```

Ou:

```bash
kubectl -n ctf get deployment web -o yaml
```

Procure:

```yaml
resources:
  requests:
  limits:
```

---

### CPU

```text
1000m = 1 CPU
500m  = 0.5 CPU
100m  = 0.1 CPU
50m   = 0.05 CPU
```

---

### Memória

Valores comuns:

```text
32Mi
64Mi
128Mi
256Mi
512Mi
1Gi
```

</details>

---

<details>
<summary>🧰 Comandos rápidos que podem salvar seu CTF</summary>

Ver tudo:

```bash
kubectl -n ctf get all
```

Pods + labels:

```bash
kubectl -n ctf get pods --show-labels
```

Investigar Pod:

```bash
kubectl -n ctf describe pod <nome>
```

Ver Deployment:

```bash
kubectl -n ctf get deploy web -o yaml
```

Editar Deployment:

```bash
kubectl -n ctf edit deploy web
```

Editar Service:

```bash
kubectl -n ctf edit svc web
```

Endpoints:

```bash
kubectl -n ctf get endpoints web
```

Storage:

```bash
kubectl -n ctf get pvc
kubectl get pv
```

Entrar no container:

```bash
kubectl -n ctf exec -it <pod> -- sh
```

Acompanhar Pods:

```bash
kubectl -n ctf get pods -w
```

Validar o desafio:

```bash
ctf-check
```

</details>

---

## 🧠 Não sabe qual comando usar?

Pense primeiro **no recurso que você está investigando**:

```text
O Pod não está saudável?
        │
        └── describe / logs

O Deployment está incorreto?
        │
        └── get / describe / edit

O Service não encontra os Pods?
        │
        └── labels / selector / endpoints

Precisa verificar configuração?
        │
        └── ConfigMap / Secret

Problema com dados?
        │
        └── PVC / PV / volumes

Precisa olhar dentro da aplicação?
        │
        └── exec

Alterou o Deployment?
        │
        └── rollout / get pods -w
```

> A ferramenta mais importante do desafio não é decorar comandos: é saber **o que investigar em seguida**.

---

<div align="center">

### ☸️ Observe → Investigue → Entenda → Corrija → Valide

</div>
