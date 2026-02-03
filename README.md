# MicroK8S

How to install ArgoCD using the helm

documentation and packages: https://artifacthub.io/packages/helm/argo/argo-cd

file: values.yaml
Containing the haa configuration argocd

Commands:

Create a argocd namespace
#kubectl create namespace argocd

How to install argocd using the kubectl
#kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-cd/refs/heads/master/manifests/ha/install.yaml -n argoc

List all the pods from argocd namespace
#kubectl get all -n argocd

If the error show:
argocd-applicationset-controller is in CrashLoopBackOff.
  Install the applicationset
  #kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj-labs/applicationset/v0.4.0/manifests/install.yaml


How to get the password
#kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d

password = RGoHz0OHeARxOgK4


Enable the ArgoCD portal access
# kubectl port-forward svc/argocd-server -n argocd --address 0.0.0.0 8080:443 &

How to access the argocd portal
#https://127.0.0.1:8080


Generate ssh-keygen
#ssh-keygen -t ed25519 -C "argocd@microk8s" -f argocd-github

Create a ssh-key in "Deploy keys" in GitHUB portal
#Faça esse passo pelo portal

Criar o Secret do repositório no Argo CD
#kubectl -n argocd create secret generic microk8s-repo \
  --from-literal=url=git@github.com:ffagian/microk8s.git \
  --from-file=sshPrivateKey=argocd-github

Adicione o label
#kubectl -n argocd label secret microk8s-repo \
  argocd.argoproj.io/secret-type=repository












