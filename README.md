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
