# Installing Rancher on Amazon EKS

O Rancher é uma uma plataforma de código fonte aberto que permite o gerenciamento de uma infraestrutura de conteineres (seja usando Docker ou Kubernetes) com o objetivo de efetuar o deploy de aplicações em ambientes de rede local ou em serviços de nuvem (Azure, Digital Ocean, AWS, etc).

## Objetivo
Instalar e configurar cluster rancher no eks aws

## Premissas
Você deve usar credenciais de usuário do IAM para a criação do cluster EKS, não credenciais raiz. Se você criar seu cluster do Amazon EKS usando credenciais raiz, não poderá se autenticar no cluster.
E um cluster eks configurado

## Instalação do Ingress
Para expor o endpoint do rancher fora do cluster, precisamos configurar o controlador de entrada nginx com o serviço de balanceador de carga na frente do nginx vamos instalar usando o helm.
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --set controller.service.type=LoadBalancer --version 3.12.0 --create-namespace
kubectl get svc -n ingress-nginx
```
## Instalação do Rancher vamos utilizar o helm conforme abaixo 
```
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
kubectl create namespace cattle-system
kubectl apply --validate=false -f https://github.com/jetstack/cert-manager/releases/download/v1.0.4/cert-manager.crds.yaml
kubectl create namespace cert-manager
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v1.0.4
kubectl get pods -n cert-manager
```
Esta opção usa um gerenciador de certificados para solicitar e renovar automaticamente os certificados Let’s Encrypt. Este é um serviço gratuito que fornece um certificado válido, pois Let’s Encrypt é uma CA confiável.
```
helm install rancher rancher-latest/rancher --namespace cattle-system --set hostname=seu-dominio --set ingress.tls.source=letsEncrypt --set letsEncrypt.email=email@exemple.com
kubectl -n cattle-system get deploy rancher
```

# Import rancher

Vamos realizar o import para que o rancher obtenha as informações do cluster EKS

- Clique em **Global Apps**
<img src="https://i.imgur.com/Jm2xATH.png"
     alt="Markdown Monster icon"
     style="float: left; margin-right: 10px;" />

- Clique em **Amazon EKS**
<img src="https://i.imgur.com/6cv0f8J.png"
     alt="Markdown Monster icon"
     style="float: left; margin-right: 10px;" />

- Insira as informações necessárias
<img src="https://i.imgur.com/ASN6ett.png"
     alt="Markdown Monster icon"
     style="float: left; margin-right: 10px;" />

- Depois de concluindo a ativação temos as informações do cluster em dashboard
<img src="https://i.imgur.com/hLksD82.png"
     alt="Markdown Monster icon"
     style="float: left; margin-right: 10px;" />

## Configurando Prometheus e Grafana

Uma das vantagens do rancher é poder instalar diversos serviços no próprio painel em apps usando helm e depois de instalado é criado um painel no próprio rancher aonde é possível realizar o monitoramento, e também acessar as urls do grafana e prometheus

- Painel monitor Rancher
<img src="https://i.imgur.com/s4mB65W.png"
     alt="Markdown Monster icon"
     style="float: left; margin-right: 10px;" />

- Painel rancher metricas
<img src="https://i.imgur.com/gaxuCHn.png"
     alt="Markdown Monster icon"
     style="float: left; margin-right: 10px;" />

- Painel grafana 
<img src="https://i.imgur.com/9RljEVH.png"
     alt="Markdown Monster icon"
     style="float: left; margin-right: 10px;" />

- Painel Prometheus
<img src="https://i.imgur.com/3oVYwOJ.png"
     alt="Markdown Monster icon"
     style="float: left; margin-right: 10px;" />
