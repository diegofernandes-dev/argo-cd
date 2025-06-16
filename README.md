# Configurar Argo CD com K3D + Ingress NGINX



## 1. Criar cluster com Load Balancer e HTTPS mapeado

```bash
k3d cluster create dev-cluster \
  --k3s-arg "--disable=traefik@server:0" \
  --agents 1 \
  --port 80:80@loadbalancer \
  --port 443:443@loadbalancer

```
## 2. Instalar o ingress-nginx via Helm

```bash

kubectl create namespace ingress-nginx

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm install ingress-nginx ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --set controller.service.type=LoadBalancer

```

## 3. Instalar o Argo CD 

```bash
kubectl create namespace argocd

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## 4. Criar o Ingress para o ArgoCD
```bash
kubectl apply -f argocd-ingress.yaml
```


## 5. Gerar certificado com SAN
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout argocd.key \
  -out argocd.crt \
  -config openssl-san.cnf
```
Depois gerar o secret:

```bash
kubectl create secret tls argocd-tls \
  --cert=argocd.crt \
  --key=argocd.key \
  -n argocd
  ```
E force o NGINX Ingress a recarregar:
```bash:
kubectl annotate ingress argocd-ingress -n argocd kubernetes.io/ingress.class=nginx --overwrite
```

## 6. Apontar o domínio no /etc/hosts
```bash
sudo echo "127.0.0.1 argocd.localhost" | sudo tee -a /etc/hosts
```

## 7. Teste
```bash
curl -vk https://argocd.local
```