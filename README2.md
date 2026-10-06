# 🚨 Kubernetes CTF 2 — Restore Production

> **Incidente crítico:** uma aplicação em produção apresenta múltiplas falhas após uma mudança no ambiente Kubernetes.

Seu objetivo é investigar o cluster, identificar os problemas e **restaurar completamente a aplicação**.

Este desafio foi desenvolvido para colocar em prática troubleshooting de Kubernetes utilizando situações semelhantes às encontradas em ambientes reais.

---

## 🎯 Objetivo

Você recebeu um ambiente Kubernetes com diversos problemas.

A aplicação principal é:

```text
production-api
```

O ambiente está no namespace:

```text
ctf2
```

Existem **8 flags** disponíveis.

As flags serão liberadas conforme os problemas forem corrigidos.

---

## ⏱️ Regras

- Tempo sugerido: **60 minutos**
- Total de flags: **8**
- Pontuação máxima: **100 pontos**
- Utilize apenas os recursos disponíveis no cluster
- Você pode consultar a documentação e os comandos apresentados durante o treinamento
- Diferentes soluções podem ser válidas
- O objetivo principal é **investigar antes de alterar**

> 💡 Não saia editando todos os recursos imediatamente.
>
> Observe o ambiente, formule uma hipótese, investigue e só então faça a correção.

---

# 🚀 Iniciando o desafio

Execute no terminal do Killercoda:

```bash
curl -fsSL https://raw.githubusercontent.com/lucaspaulukx/ctf-k8s-script/main/setup2.sh | bash
```

Informe os dados solicitados:

```text
Primeiro nome:
E-mail:
Matrícula:
```

Utilize os mesmos dados que serão informados no formulário de entrega.

Após a preparação do ambiente:

```bash
kubectl -n ctf2 get all
```

A partir daqui, o incidente começou. ⏱️

---

# 🔍 Validação

Durante o desafio você pode verificar seu progresso executando:

```bash
ctf-check
```

O validador verifica o estado atual do ambiente.

Exemplo:

```text
========================================================
     KUBERNETES CTF 2 - RESTORE PRODUCTION
========================================================

FLAG 1: ❌
FLAG 2: ❌
FLAG 3: ❌
FLAG 4: ❌
FLAG 5: ❌
FLAG 6: ❌
FLAG 7: ❌

--------------------------------------------------------
Progresso: 0/7
--------------------------------------------------------

A produção ainda possui problemas.
```

Quando uma condição for corrigida, a respectiva flag será liberada.

---

# 🚨 INCIDENTE #1 — Um componente não consegue iniciar

**Pontuação: 10 pontos**

A equipe de operações informou que um dos componentes do ambiente está apresentando reinicializações constantes.

Sua primeira missão é identificar:

- qual workload apresenta o problema;
- por que o container está encerrando;
- qual configuração está impedindo sua inicialização.

Restaure o componente e faça com que ele permaneça saudável.

Depois valide:

```bash
ctf-check
```

<details>
<summary>💡 Dica 1</summary>

Comece observando os Pods:

```bash
kubectl -n ctf2 get pods
```

Preste atenção nas colunas:

```text
READY
STATUS
RESTARTS
```

</details>

<details>
<summary>💡 Dica 2</summary>

Se um container estiver reiniciando, seus logs podem explicar por quê.

```bash
kubectl -n ctf2 logs <pod>
```

Você também pode consultar os logs do Deployment:

```bash
kubectl -n ctf2 logs deployment/diagnostic-worker
```

</details>

<details>
<summary>💡 Dica 3</summary>

Investigue como o container recebe seus arquivos de configuração.

Alguns comandos úteis:

```bash
kubectl -n ctf2 get deployment diagnostic-worker -o yaml
kubectl -n ctf2 get configmap diagnostic-config -o yaml
```

Compare o **arquivo disponível** com o **arquivo esperado pela aplicação**.

</details>

---

# 🚨 INCIDENTE #2 — Aplicação iniciou no modo incorreto

**Pontuação: 10 pontos**

A aplicação principal está utilizando uma configuração que não corresponde ao ambiente de produção.

A equipe confirmou que o ambiente deveria estar operando em:

```text
production
```

Descubra de onde a aplicação recebe essa configuração e corrija o valor.

Depois:

```bash
ctf-check
```

<details>
<summary>💡 Dica 1</summary>

Configurações de aplicações Kubernetes frequentemente são armazenadas fora da imagem do container.

Liste os recursos de configuração:

```bash
kubectl -n ctf2 get configmap
```

</details>

<details>
<summary>💡 Dica 2</summary>

Investigue:

```bash
kubectl -n ctf2 get configmap production-config -o yaml
```

</details>

<details>
<summary>💡 Dica 3</summary>

O valor esperado é:

```text
APP_MODE=production
```

Você pode editar o recurso diretamente:

```bash
kubectl -n ctf2 edit configmap production-config
```

Lembre-se de que alterar um ConfigMap utilizado como variável de ambiente **não altera automaticamente o ambiente de containers que já estão executando**.

</details>

---

# 🚨 INCIDENTE #3 — Capacidade abaixo do esperado

**Pontuação: 10 pontos**

O time responsável pela aplicação informou que o ambiente de produção deveria possuir redundância mínima.

Atualmente a quantidade de instâncias da API não atende ao requisito operacional.

A aplicação deve possuir:

```text
2 réplicas
```

Faça a correção e valide o ambiente.

<details>
<summary>💡 Dica 1</summary>

Observe:

```bash
kubectl -n ctf2 get deployment
```

</details>

<details>
<summary>💡 Dica 2</summary>

Veja o estado desejado do Deployment:

```bash
kubectl -n ctf2 get deployment production-api -o yaml
```

Procure por:

```yaml
spec:
  replicas:
```

</details>

<details>
<summary>💡 Dica 3</summary>

Uma maneira de alterar a quantidade de réplicas é:

```bash
kubectl -n ctf2 scale deployment production-api --replicas=2
```

Depois acompanhe:

```bash
kubectl -n ctf2 get pods -w
```

Use `Ctrl+C` para sair do modo de acompanhamento.

</details>

---

# 🚨 INCIDENTE #4 — API não responde pela rede interna

**Pontuação: 15 pontos**

Os Pods da aplicação parecem existir, porém outros workloads do cluster não conseguem acessar a API utilizando o Service:

```text
production-api
```

Investigue o caminho de rede entre:

```text
Cliente
   ↓
Service
   ↓
Pod
   ↓
Container
```

Restaure a comunicação.

<details>
<summary>💡 Dica 1</summary>

Comece pelo Service:

```bash
kubectl -n ctf2 get service production-api
```

Depois:

```bash
kubectl -n ctf2 describe service production-api
```

</details>

<details>
<summary>💡 Dica 2</summary>

Veja quais portas estão configuradas:

```bash
kubectl -n ctf2 get service production-api -o yaml
```

Também investigue qual porta o container expõe:

```bash
kubectl -n ctf2 get deployment production-api -o yaml
```

</details>

<details>
<summary>💡 Dica 3</summary>

O Service recebe tráfego na porta:

```text
80
```

A aplicação dentro do container está ouvindo na porta:

```text
8080
```

Revise a relação entre:

```text
port
targetPort
containerPort
```

</details>

---

# 🚨 INCIDENTE #5 — Kubernetes reinicia containers saudáveis

**Pontuação: 15 pontos**

A equipe percebeu que containers da aplicação estão sendo reiniciados mesmo quando a API aparentemente consegue responder requisições.

Investigue os eventos dos Pods e determine por que o Kubernetes considera o container não saudável.

Corrija a verificação responsável pelas reinicializações.

<details>
<summary>💡 Dica 1</summary>

Observe:

```bash
kubectl -n ctf2 get pods
```

Depois escolha um Pod da aplicação:

```bash
kubectl -n ctf2 describe pod <pod>
```

Preste atenção em:

```text
Events
Restart Count
Liveness
Readiness
```

</details>

<details>
<summary>💡 Dica 2</summary>

Investigue as probes:

```bash
kubectl -n ctf2 get deployment production-api -o yaml
```

Compare:

```text
readinessProbe
livenessProbe
```

Elas possuem objetivos diferentes.

</details>

<details>
<summary>💡 Dica 3</summary>

A aplicação disponibiliza um endpoint de saúde em:

```text
/health
```

na porta:

```text
8080
```

</details>

---

# 🚨 INCIDENTE #6 — Dados persistentes desapareceram da aplicação

**Pontuação: 15 pontos**

A equipe garante que existe armazenamento persistente contendo dados da aplicação.

O PVC existe e os dados foram gravados anteriormente, porém a aplicação não consegue encontrá-los no caminho esperado:

```text
/data
```

Investigue o armazenamento e restaure o acesso aos dados.

<details>
<summary>💡 Dica 1</summary>

Verifique:

```bash
kubectl -n ctf2 get pvc
```

E:

```bash
kubectl -n ctf2 get pv
```

Observe o estado do PVC.

</details>

<details>
<summary>💡 Dica 2</summary>

Investigue como o Deployment utiliza o volume:

```bash
kubectl -n ctf2 get deployment production-api -o yaml
```

Procure por:

```text
volumes
volumeMounts
mountPath
persistentVolumeClaim
```

</details>

<details>
<summary>💡 Dica 3</summary>

O PVC:

```text
production-data
```

deve estar disponível dentro do container em:

```text
/data
```

Depois da correção, você pode verificar:

```bash
POD=$(kubectl -n ctf2 get pod -l app=production-api -o jsonpath='{.items[0].metadata.name}')

kubectl -n ctf2 exec "$POD" -- ls -la /data
```

</details>

---

# 🚨 INCIDENTE #7 — Recursos fora do padrão de produção

**Pontuação: 10 pontos**

A aplicação foi implantada sem seguir o padrão de recursos definido pela plataforma.

O padrão obrigatório para o container principal é:

```text
CPU Request:     50m
Memory Request:  32Mi

CPU Limit:       200m
Memory Limit:    128Mi
```

Corrija o Deployment.

<details>
<summary>💡 Dica 1</summary>

Investigue:

```bash
kubectl -n ctf2 get deployment production-api -o yaml
```

Procure por:

```yaml
resources:
```

</details>

<details>
<summary>💡 Dica 2</summary>

Um container pode possuir:

```yaml
resources:
  requests:
  limits:
```

`requests` e `limits` possuem funções diferentes.

</details>

<details>
<summary>💡 Dica 3</summary>

O resultado esperado é:

```yaml
resources:
  requests:
    cpu: "50m"
    memory: "32Mi"
  limits:
    cpu: "200m"
    memory: "128Mi"
```

</details>

---

# 🏁 INCIDENTE FINAL — Restore Production

**Pontuação: 15 pontos**

Você acredita que todos os problemas foram corrigidos.

Agora execute:

```bash
ctf-check
```

Para concluir o incidente, o ambiente deve atender a todos os requisitos e a aplicação principal deve possuir:

```text
2/2 Ready
```

Quando todo o ambiente estiver saudável, o sistema liberará a última flag:

```text
FLAG{XXXXXXXX_08_PRODUCTION_RESTORED}
```

Parabéns. Produção restaurada. ☸️

---

# 🧮 Pontuação

| Desafio | Pontos |
|---|---:|
| FLAG 1 — Troubleshooting / CrashLoopBackOff | 10 |
| FLAG 2 — Configuração | 10 |
| FLAG 3 — Deployment / Réplicas | 10 |
| FLAG 4 — Service / Networking | 15 |
| FLAG 5 — Probes | 15 |
| FLAG 6 — Storage | 15 |
| FLAG 7 — Resources | 10 |
| FLAG 8 — Restore Production | 15 |
| **TOTAL** | **100** |

---

# 🧭 Estratégia de Troubleshooting

Quando encontrar um problema, tente seguir esta sequência:

```text
1. Observar
      ↓
2. Identificar o recurso
      ↓
3. Ver estado e eventos
      ↓
4. Consultar logs
      ↓
5. Inspecionar configuração
      ↓
6. Formular uma hipótese
      ↓
7. Corrigir
      ↓
8. Validar
```

Evite alterar recursos sem entender primeiro o sintoma.

---

# 📚 Referência rápida de kubectl

<details>
<summary>🔎 Visualizar recursos</summary>

Listar recursos:

```bash
kubectl -n ctf2 get all
```

Pods:

```bash
kubectl -n ctf2 get pods
```

Mais informações:

```bash
kubectl -n ctf2 get pods -o wide
```

Deployments:

```bash
kubectl -n ctf2 get deployments
```

Services:

```bash
kubectl -n ctf2 get services
```

ConfigMaps:

```bash
kubectl -n ctf2 get configmaps
```

PVCs:

```bash
kubectl -n ctf2 get pvc
```

Ver YAML:

```bash
kubectl -n ctf2 get <recurso> <nome> -o yaml
```

</details>

<details>
<summary>🩺 Investigar problemas</summary>

Detalhes de um Pod:

```bash
kubectl -n ctf2 describe pod <pod>
```

Detalhes de um Deployment:

```bash
kubectl -n ctf2 describe deployment <deployment>
```

Logs:

```bash
kubectl -n ctf2 logs <pod>
```

Logs através do Deployment:

```bash
kubectl -n ctf2 logs deployment/<deployment>
```

Logs do container anterior:

```bash
kubectl -n ctf2 logs <pod> --previous
```

Eventos:

```bash
kubectl -n ctf2 get events --sort-by=.metadata.creationTimestamp
```

</details>

<details>
<summary>💻 Executar comandos dentro de containers</summary>

Abrir shell:

```bash
kubectl -n ctf2 exec -it <pod> -- sh
```

Executar um comando:

```bash
kubectl -n ctf2 exec <pod> -- <comando>
```

Exemplo:

```bash
kubectl -n ctf2 exec <pod> -- ls -la /data
```

</details>

<details>
<summary>📝 Editar recursos</summary>

Deployment:

```bash
kubectl -n ctf2 edit deployment production-api
```

Service:

```bash
kubectl -n ctf2 edit service production-api
```

ConfigMap:

```bash
kubectl -n ctf2 edit configmap production-config
```

Após uma alteração no Deployment:

```bash
kubectl -n ctf2 rollout status deployment production-api
```

</details>

<details>
<summary>📦 Deployment e réplicas</summary>

Visualizar:

```bash
kubectl -n ctf2 get deployment
```

Alterar réplicas:

```bash
kubectl -n ctf2 scale deployment <nome> --replicas=<quantidade>
```

Acompanhar rollout:

```bash
kubectl -n ctf2 rollout status deployment <nome>
```

Histórico:

```bash
kubectl -n ctf2 rollout history deployment <nome>
```

Reiniciar rollout:

```bash
kubectl -n ctf2 rollout restart deployment <nome>
```

</details>

<details>
<summary>🌐 Services, Labels e Endpoints</summary>

Services:

```bash
kubectl -n ctf2 get svc
```

Detalhes:

```bash
kubectl -n ctf2 describe svc <service>
```

Endpoints:

```bash
kubectl -n ctf2 get endpoints
```

Labels dos Pods:

```bash
kubectl -n ctf2 get pods --show-labels
```

Selecionar Pods por label:

```bash
kubectl -n ctf2 get pods -l app=production-api
```

</details>

<details>
<summary>💾 Storage</summary>

PVC:

```bash
kubectl -n ctf2 get pvc
```

PV:

```bash
kubectl get pv
```

Detalhes:

```bash
kubectl -n ctf2 describe pvc production-data
```

Procure no Deployment por:

```text
volumes
volumeMounts
persistentVolumeClaim
mountPath
```

</details>

<details>
<summary>❤️ Probes</summary>

Ver configuração:

```bash
kubectl -n ctf2 get deployment production-api -o yaml
```

Ver eventos:

```bash
kubectl -n ctf2 describe pod <pod>
```

Lembre-se:

```text
Readiness Probe
      ↓
O Pod pode receber tráfego?

Liveness Probe
      ↓
O container continua saudável?
Se falhar repetidamente, o kubelet pode reiniciá-lo.
```

</details>

<details>
<summary>⚙️ CPU e Memória</summary>

Ver resources:

```bash
kubectl -n ctf2 get deployment production-api \
  -o jsonpath='{.spec.template.spec.containers[0].resources}'; echo
```

Referência:

```text
1000m CPU = 1 CPU
 200m CPU = 0,2 CPU
  50m CPU = 0,05 CPU
```

Estrutura:

```yaml
resources:
  requests:
    cpu: "50m"
    memory: "32Mi"
  limits:
    cpu: "200m"
    memory: "128Mi"
```

</details>

---

# 🆘 Comandos rápidos

Quando estiver perdido, comece por:

```bash
kubectl -n ctf2 get all
```

Depois:

```bash
kubectl -n ctf2 get pods
kubectl -n ctf2 describe pod <pod>
kubectl -n ctf2 logs <pod>
```

Para networking:

```bash
kubectl -n ctf2 get svc
kubectl -n ctf2 get endpoints
```

Para configuração:

```bash
kubectl -n ctf2 get configmap
```

Para armazenamento:

```bash
kubectl -n ctf2 get pvc
kubectl get pv
```

Para validar seu progresso:

```bash
ctf-check
```

---

# ☸️ Boa sorte!

O objetivo deste desafio não é apenas encontrar flags.

O objetivo é desenvolver um processo de troubleshooting:

> **Observe → investigue → formule uma hipótese → corrija → valide.**

Produção está fora. Agora é com você. 🚨
