## 🛠 Infraestrutura de Rede e Load Balancer (MetalLB)

Neste repositório, a gestão de endereços IP externos para o cluster MicroK8s é realizada pelo **MetalLB**. Isso permite que serviços do tipo `LoadBalancer` (como o Ingress Nginx) recebam um IP acessível diretamente pela rede local (Host-Only Network do VirtualBox).

### ⚠️ Troubleshooting: Erro de Certificado no Webhook (x509)

Durante a automação via **Argo CD**, foi identificado um erro persistente ao aplicar os manifestos de `IPAddressPool` e `L2Advertisement`:

> *Error from server (InternalError): failed calling webhook "ipaddresspoolvalidationwebhook.metallb.io": x509: certificate signed by unknown authority*

#### **Causa Raiz**
O MetalLB utiliza **Admission Webhooks** para validar a sintaxe dos recursos. Em ambientes MicroK8s, os certificados TLS autoassinados gerados pelo MetalLB podem não ser reconhecidos imediatamente pelo API Server do Kubernetes. Isso gera um bloqueio ("pedágio"), impedindo que o Argo CD crie os pools de IP, resultando em serviços com status `<pending>`.

#### **Solução Aplicada**
Para garantir a resiliência do deploy e evitar intervenções manuais em reinicializações do cluster, as seguintes ações foram tomadas:

1.  **Ajuste de API Version:** Migração de `metallb.universe.tf` para `metallb.io/v1beta1` (versões v0.14.3+).
2.  **Bypass de Validação:** Remoção manual da configuração de validação que apresentava erro de certificado:
    ```bash
    kubectl delete validatingwebhookconfiguration validating-webhook-configuration
    ```
3.  **Configuração de Persistência:** Os manifestos de configuração foram organizados na **Wave 2** do Argo CD, garantindo que o controlador do MetalLB já esteja rodando antes da tentativa de provisionamento dos IPs.



---

### 📍 Planejamento de IPs (Address Pools)

Os IPs abaixo foram reservados para evitar conflitos na rede local e garantir que os serviços exponham portas padronizadas:

| Serviço | Pool Name | Endereço IP | Finalidade |
| :--- | :--- | :--- | :--- |
| **Ingress Nginx** | `nginx-pool` | `192.168.56.201` | Ponto de entrada principal (HTTP/HTTPS) |
| **Istio Gateway** | `istio-pool` | `192.168.56.200` | Malha de serviços e Service Mesh |

---

### 🚀 Como Validar a Instalação

Após o sincronismo do Argo CD, verifique se o IP foi atribuído corretamente ao Ingress Controller:

```bash
# Verificar se os pools foram criados
kubectl get IPAddressPool -n metallb-system

# Verificar se o Nginx recebeu o IP externo
kubectl get svc -n ingress-nginx