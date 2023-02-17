# Installing Rancher on Amazon EKS

O Rancher é uma uma plataforma de código fonte aberto que permite o gerenciamento de uma infraestrutura de conteineres (seja usando Docker ou Kubernetes) com o objetivo de efetuar o deploy de aplicações em ambientes de rede local ou em serviços de nuvem (Azure, Digital Ocean, AWS, etc).

## Objetivo
Instalar e configurar cluster rancher no eks aws

## Premissas
Você deve usar credenciais de usuário do IAM para a criação do cluster EKS, não credenciais raiz. Se você criar seu cluster do Amazon EKS usando credenciais raiz, não poderá se autenticar no cluster.
E um cluster eks configurado

## Testado
 - Kubernetes 1.23
 - ingress-nginx/controller:v1.9.2
 - cert manager v1.9.2

## Instalação do Ingress
Para expor o endpoint do rancher fora do cluster, precisamos configurar o controlador de entrada nginx com o serviço de balanceador de carga na frente do nginx vamos instalar usando o helm.
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --set controller.service.type=LoadBalancer --create-namespace
kubectl get svc -n ingress-nginx
```
## Instalação do Rancher vamos utilizar o helm conforme abaixo 
```
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
kubectl create namespace cattle-system
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.9.2/cert-manager.crds.yaml
kubectl create namespace cert-manager
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v1.9.2
kubectl get pods -n cert-manager
```
Esta opção usa um gerenciador de certificados para solicitar e renovar automaticamente os certificados Let’s Encrypt. Este é um serviço gratuito que fornece um certificado válido, pois Let’s Encrypt é uma CA confiável.
```
helm install rancher rancher-latest/rancher --namespace cattle-system --set hostname=rancher.example.com --set ingress.tls.source=letsEncrypt --set letsEncrypt.email=email@exemple.com
kubectl -n cattle-system get deploy rancher
```
### Usando ALB AWS e ACM
Para utilizar o ALB e ACM AWS você deve instalar o controller segue o link, é necessário um certificado válido:
https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html

Depois de instalado execute o  helm do rancher
```
helm install rancher rancher-latest/rancher --namespace cattle-system --set hostname=rancher.example.com --set 'ingress.extraAnnotations.alb\.ingress\.kubernetes\.io/scheme=internet-facing' --set 'ingress.extraAnnotations.alb\.ingress\.kubernetes\.io/success-codes=200\,404\,301\,302' --set 'ingress.extraAnnotations.alb\.ingress\.kubernetes\.io/subnets=subnet-XXX\,subnet-XXX\,subnet-XXX' --set 'ingress.extraAnnotations.alb\.ingress\.kubernetes\.io/listen-ports=[{\"HTTP\": 80}\, {\"HTTPS\": 443}]' --set 'ingress.extraAnnotations.alb\.ingress\.kubernetes\.io/certificate-arn=arn:aws:acm:eu-central-1:XXX:certificate/XXX' --set 'ingress.extraAnnotations.kubernetes\.io/ingress\.class=alb'  --set replicas=3 --set tls=external --create-namespace
``` 
Feito basta apontar seu loadbalancer ao domínio no route53

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
