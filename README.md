![Arch_Diagram](https://github.com/rachakondadharmendra/Ops-Knowledge-Base/blob/main/Arch-Daigrams/apisix-istio-arch.gif)



# Instance Creation with Configuration

- **OS:** Ubuntu 22.04
- **Storage:** 25GB
- **Open Ports:** 80, 8080, 22 (default for SSH)
- **IAM Instance Profile:** Admin_access_role (To access AWS resources securely)

## User Data

```bash
#!/bin/bash

sudo apt-get update -y && sudo apt-get dist-upgrade -y && sudo apt install unzip -y

# Optional
curl -q "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip -q awscliv2.zip
sudo ./aws/install
/usr/local/bin/aws --version

sudo apt install docker.io -y docker-compose -y
sudo usermod -aG docker ubuntu
newgrp docker
```
Sure, here's the content formatted in a proper Markdown (.md) file:

```markdown
# Instance Creation with Configuration

- **OS:** Ubuntu 22.04
- **Storage:** 25GB
- **Open Ports:** 80, 8080, 22 (default for SSH)
- **IAM Instance Profile:** Admin_access_role (To access AWS resources securely)

## User Data

```bash
#!/bin/bash

sudo apt-get update -y && sudo apt-get dist-upgrade -y && sudo apt install unzip -y

# Optional
curl -q "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip -q awscliv2.zip
sudo ./aws/install
/usr/local/bin/aws --version

sudo apt install docker.io -y docker-compose -y
sudo usermod -aG docker ubuntu
newgrp docker
```

## Kubectl Installation

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```

## Creation of Minikube

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
sudo dpkg -i minikube_latest_amd64.deb
minikube start
kubectl get po -A
```

## Helm Installation

```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

# Installation of Istio

```bash
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
helm install my-istio-base-release -n istio-system --create-namespace istio/base --set global.istioNamespace=istio-system
helm install my-istiod-release -n istio-system --create-namespace istio/istiod --set telemetry.enabled=true --set global.istioNamespace=istio-system

curl -sL https://istio.io/downloadIstioctl | sh -
export PATH=$HOME/.istioctl/bin:$PATH 
istioctl install --set profile=default

# Doubtful Installation
helm install gateway -n istio-ingress --create-namespace istio/gateway

# Store Default Values in a Folder
helm search repo istio/base && helm search repo istio/istiod
helm show values istio/base --version 1.22.0 >> helmdefaults/istio-base-default.yaml
helm show values istio/istiod --version 1.22.0 >> helmdefaults/istio-base-default.yaml

# Command to Check Installation
helm status my-istio-base-release -n istio-system
helm get all my-istio-base-release -n istio-system

# Configure Minikube for Istio
minikube config set driver kvm2
minikube start --memory=16384 --cpus=4 --kubernetes-version=v1.26.1
```

# APISIX Installation

```bash
# Source: https://apisix.apache.org/docs/ingress-controller/next/tutorials/configure-ingress-with-gateway-api/

helm repo add apisix https://charts.apiseven.com
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
kubectl create ns ingress-apisix
kubectl label ns ingress-apisix istio-injection=disabled
helm install apisix apisix/apisix --namespace ingress-apisix \
--set service.type=NodePort \
--set ingress-controller.enabled=true \
--set ingress-controller.config.apisix.serviceNamespace=ingress-apisix \
--set ingress-controller.config.kubernetes.enableGatewayAPI=true \
--set gateway.tls.enabled=true

kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v0.5.0/standard-install.yaml

export NODE_PORT=$(kubectl get --namespace ingress-apisix -o jsonpath="{.spec.ports[0].nodePort}" services apisix-gateway)
export NODE_IP=$(kubectl get nodes --namespace ingress-apisix -o jsonpath="{.items[0].status.addresses[0].address}")
echo http://$NODE_IP:$NODE_PORT

minikube tunnel 
kubectl describe pod <podname>

kubectl get svc -n demo
minikube ssh
curl http<>
curl http://192.168.49.2:32663/svr1 -H 'Host: apisix.rachakonda.info'
curl --header "Host: apisix.rachakonda.info" http://10.108.88.249:80/svr1

kubectl get svc -n demo
kubectl exec -it client -n demo -- sh

while true; do curl http://srv1.demo.svc.cluster.local:80/svr1 && echo "" && sleep 1; done
while true; do curl http://srv2.demo.svc.cluster.local:80/svr2 && echo "" && sleep 1; done

https://apisix.apache.org/docs/ingress-controller/tutorials/mtls/

kubectl -n apisix exec -it apisix-75fc49f866-8smdv -- curl "http://127.0.0.1:5001/" -H "Host: mtls.srv.local"

minikube service apisix-gateway --url -n ingress-apisix

kubectl port-forward svc/apisix-gateway -n ingress-apisix 8080:80
```
