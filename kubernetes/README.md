# Config Minikube
minikube config set driver hyperv  --hyperv-external-adapter
minikube config set cpus 16
minikube config set memory 16384
minikube config set disk-size 150000
minikube start --hyperv-external-adapter "External Virtual Switch"

minikube addons enable metallb
minikube addons configure metallb


microk8s enable dns
microk8s enable metallb
microk8s enable hostpath-storage

Redis
Grafana Agent Flow
Vault
Istio

# Install Redis Ot Container

helm repo add ot-helm https://ot-container-kit.github.io/helm-charts/

helm repo update

helm install redis-operator ot-helm/redis-operator --namespace operators --create-namespace

helm install redis-cluster ot-helm/redis-cluster --namespace srv-redis -f redis/values.yaml --create-namespace

# Install Mongo Operator

helm repo add ot-helm https://ot-container-kit.github.io/helm-charts/

helm repo update

helm upgrade mongodb-operator ot-helm/mongodb-operator --install --namespace operators --install

helm install mongodb-cluster --namespace srv-mongodb ot-helm/mongodb-cluster --create-namespace --set storage.storageClass=microk8s-hostpath


# Install Vault Hashicorp

- https://developer.hashicorp.com/vault/tutorials/kubernetes/kubernetes-raft-deployment-guide

helm repo add hashicorp https://helm.releases.hashicorp.com

helm repo update

helm install vault hashicorp/vault -n vault-operator --create-namespace --values vault/values.yaml

kubectl label namespace vault-operator istio-injection=enabled


kubectl apply -f vault/network.yaml

# Install Istio
> https://www.solo.io/blog/3-most-common-ways-install-istio

helm repo add istio https://istio-release.storage.googleapis.com/charts

helm repo update

helm install istio-base istio/base -n istio-system --set defaultRevision=default --create-namespace

helm install istiod istio/istiod -n istio-system --wait

kubectl label namespace default istio-injection=enabled

### Deploy default Ingress Istio
kubectl create namespace istio-ingress
kubectl label namespace istio-ingress istio-injection=enabled
helm install istio-ingress istio/gateway -n istio-ingress --wait


### Deploy new Ingress Istio Cluster
kubectl create namespace cluster-ingress
kubectl label namespace cluster-ingress istio-injection=enabled
helm install cluster-ingress istio/gateway -n cluster-ingress -f istio/istio-ingress.yaml --wait

# Install Prometheus

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace

kubectl apply -f prometheus/network.yaml

# Install Loki
- https://medium.com/@davis.angwenyi/how-to-install-grafana-loki-using-helm-3006e3d3581a

helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm install loki-server grafana/loki --values loki/values.yaml -n monitoring --create-namespace

kubectl apply -f loki/network.yaml

# Install Grafana

helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana-server grafana/grafana --namespace monitoring --create-namespace


kubectl apply -f grafana/network.yaml

#- kubectl get secret --namespace monitoring grafana-server -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
kubectl get secret --namespace monitoring grafana-server -o jsonpath="{.data.admin-password}" | %{ [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }

# Install Tempo

helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm install tempo-server grafana/tempo-distributed -n monitoring -f tempo/values.yaml --create-namespace



# Install Otel

helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm repo update

helm install otel-collector open-telemetry/opentelemetry-collector -f otel/value.yaml --set image.repository="otel/opentelemetry-collector-k8s" -n otel --create-namespace 



# Install Grafana Agent Flow

https://grafana.com/docs/agent/latest/flow/get-started/install/kubernetes/

helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

kubectl create namespace grafana-agent

kubectl create configmap --namespace grafana-agent grafana-agent-flow "--from-file=config.river=./config.river"

helm install --namespace grafana-agent grafana-agent-flow grafana/grafana-agent -f grafana-agent/values.yaml

helm upgrade --namespace grafana-agent grafana-agent-flow grafana/grafana-agent -f grafana-agent/values.yaml