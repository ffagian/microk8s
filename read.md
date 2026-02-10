# 🚀 Gerenciamento de Cluster via GitOps (Argo CD)

Este repositório utiliza o conceito de **GitOps** para gerenciar a infraestrutura do cluster Kubernetes (MicroK8s). A peça central dessa automação é o **Argo CD**.

## 🛠️ Resumo da Instalação (Bootstrap)

Para configurar o cluster do zero, utilizamos os seguintes comandos:

1. **`kubectl create namespace argocd`**
   * **O que faz:** Cria o isolamento lógico para as ferramentas de gerenciamento.
   * **Por que:** Manter o Argo CD fora do namespace `default` é uma prática recomendada de segurança e organização.

2. **`kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml --server-side --force-conflicts`**
   * **O que faz:** Instala os componentes base (o "motor") do Argo CD.
   * **Parâmetros:**
     * `--server-side`: Resolve o erro de `Too long` nas CRDs (anotações muito grandes).
     * `--force-conflicts`: Garante que o novo gerenciador de recursos assuma o controle sobre instalações antigas.

3. **`kubectl apply -f apps/bootstrap/application.yaml -n argocd`**
   * **O que faz:** Cria a aplicação "Mestra" (Root App).
   * **Controle:** Este arquivo faz o "App-of-Apps", dizendo ao Argo CD para olhar o repositório Git e gerenciar a si mesmo e a todos os outros componentes (MetalLB, Nginx, etc.) automaticamente.

---

## 🏗️ Arquitetura do Argo CD (Componentes)

Quando instalamos o Argo CD, ele sobe vários serviços essenciais. Aqui está a função de cada um:

| Serviço | Função |
| :--- | :--- |
| **`argocd-server`** | API e Interface Web. É a porta de entrada para o usuário (CLI/UI). |
| **`argocd-application-controller`** | O **Cérebro**. Compara o estado desejado (Git) com o real (Cluster) e corrige divergências. |
| **`argocd-repo-server`** | O **Tradutor**. Clona o Git e converte YAML/Helm/Kustomize em manifestos Kubernetes. |
| **`argocd-redis`** | O **Cache**. Melhora a performance evitando clones repetitivos do repositório Git. |
| **`argocd-dex-server`** | O **Identificador**. Gerencia a autenticação e integração com SSO (GitHub, Google, etc). |
| **`argocd-notifications-controller`** | O **Mensageiro**. Envia alertas sobre o status das sincronizações. |
| **`argocd-applicationset-controller`** | O **Automação**. Permite gerar múltiplas aplicações dinamicamente. |

---

## 🔄 Fluxo de Trabalho (Workflow GitOps)

1. **Alteração:** Você altera um arquivo no Git (ex: aumenta o número de réplicas).
2. **Commit/Push:** Você envia a alteração para o GitHub.
3. **Detecção:** O `argocd-application-controller` percebe que o cluster está "OutOfSync".
4. **Sincronização:** O Argo CD aplica a mudança automaticamente para atingir o estado "Synced".

---

## ⚠️ Troubleshooting (Resolução de Problemas)

### Erro: `metadata.annotations: Too long`
Ocorre porque o histórico de anotações do `kubectl apply` excede o limite.
* **Solução:** Usar a flag `--server-side`.

### Erro: `StatefulSet argocd-application-controller invalid`
Geralmente causado por conflitos de `value` e `valueFrom` em variáveis de ambiente após múltiplas instalações.
* **Solução:** 1. `kubectl delete statefulset argocd-application-controller -n argocd`
  2. Reaplicar o manifesto oficial.

---