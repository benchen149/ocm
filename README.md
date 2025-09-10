#### Prerequisites
```
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.26.0/kind-linux-amd64
wget "https://github.com/istio/istio/releases/download/1.24.0/istio-1.24.0-linux-amd64.tar.gz" -O - | tar -xz
wget -qO- https://github.com/open-cluster-management-io/clusteradm/releases/latest/download/clusteradm_linux_amd64.tar.gz | sudo tar -xvz -C /usr/local/bin/
sudo chmod +x /usr/local/bin/clusteradm

./scripts/local-up.sh
./scripts/get-token.sh
kubectl config use-context kind-hub
kubectl apply -f metallb/
istioctl install -y -f cluster1-1.24.0.yaml
kubectl -n istio-system patch svc istio-ingressgateway   -p '{"spec": {"type": "LoadBalancer"}}'

curl -v http://ocm-dashboard.app.dev.com/login
```

#### Open Cluster Management (OCM)
```
kubectl get managedclusters
kubectl describe managedcluster <cluster-name>

clusteradm get clusters
kubectl get pods -n open-cluster-management-agent
kubectl get pods -n open-cluster-management

clusteradm get token --context  kind-hub | grep clusteradm
export joincmd=$(clusteradm get token --context  kind-hub | grep clusteradm)
$(echo ${joincmd} --force-internal-endpoint-lookup --wait --context kind-cluster2 | sed "s/<cluster_name>/kind-cluster2/g")
clusteradm accept --context kind-hub --clusters kind-cluster2 --wait

```

#### OCM Dashboard
```
helm --kube-context kind-hub install ocm-dashboard ocm-dashboard \
  --namespace ocm-dashboard \
  --create-namespace \
  --set image.tag=latest \
  --set dashboard.env.DASHBOARD_BYPASS_AUTH=true \
  --set dashboard.env.DASHBOARD_DEBUG=true

helm --kube-context kind-hub install ocm-dashboard ./charts/ocm-dashboard \
  --namespace ocm-dashboard \
  --create-namespace

helm --kube-context kind-hub upgrade ocm-dashboard charts/ocm-dashboard  --namespace ocm-dashboard

kubectl port-forward -n ocm-dashboard service/ocm-dashboard 3000:80

http://localhost:3000/login


export beartoken="eyJhbG..."

curl -L -X GET http://localhost:3000/api/clusters   -H "Authorization: Bearer $beartoken"   -H "Content-Type: application/json"

```
⚙️ Upgrade Node.js to v18 / v20
```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.nvm/nvm.sh
nvm install --lts
nvm use --lts
node -v
npm -v

make dev
npm install -g serve && serve dist

make clean
```

🛠️ Useful Commands
```
kind get clusters
kind delete cluster --name c1
docker network inspect -f '{{range.IPAM.Config}}{{.Gateway}}{{end}}' kind
```

#### OIDC Test
```
git clone https://github.com/xuezhaojun/dashboard.git
cd dashboard
git checkout devin/1756708705-oidc-login-support

make docker-build-local

npm install -g pnpm (optional)
pnpm install (optional)

kind load docker-image dashboard-ui:latest   --name hub
kind load docker-image dashboard-api:latest  --name hub

helm repo add dex https://charts.dexidp.io
helm repo update
kubectl create namespace dex

helm install dex dex/dex \
  --namespace dex \
  --values dex-values.yaml

helm upgrade --install dex dex/dex \
  --namespace dex \
  --values dex-values.yaml

```
#### OCM Metrics
```
CPU usage of OCM components
rate(container_cpu_usage_seconds_total{namespace=~"open-cluster-management.*"}[3m])

Memory usage of OCM components
container_memory_working_set_bytes{namespace=~"open-cluster-management.*"}

API server request rate for OCM resources
rate(apiserver_request_total{
  resource=~"managedclusters|managedclusteraddons|managedclustersetbindings|managedclustersets|addonplacementscores|placementdecisions|placements|manifestworks|manifestworkreplicasets"
}[1m])

Enter the OCM cluster-manager pod and Test the metrics endpoint
kubectl -n open-cluster-management exec -it cluster-manager-86b4db7564-dc97w -- sh
curl localhost:8443/metrics

Expose metrics service
kubectl -n open-cluster-management apply -f service-cluster-manager-metrics.yml

Grant metrics read access
kubectl apply -f rbac-ocm-metrics-reader.yml

Install addons/prometheus/grafana
kubectl apply -f istio-system apply -f samples/addons/prometheus.yaml
kubectl apply -f istio-system apply -f samples/addons/grafana.yaml
cm-prometheus.yml (scrape_configs : ocm )

Access prometheus/grafana UI
kubectl -n istio-system port-forward svc/prometheus 9090:9090
kubectl -n istio-system port-forward svc/grafana 3000 (admin/admin)

Go to Grafana → Dashboards → Import, and upload (ocm-metrics/ocm-dashboard.json)

```


#### References
```
Open Cluster Management (OCM)
https://github.com/open-cluster-management-io/ocm.git

setup-dev-environment
https://github.com/open-cluster-management-io/ocm/tree/main/solutions/setup-dev-environment

Open Cluster Management (OCM) dashboard
https://github.com/xuezhaojun/dashboard

deepwiki-dashboard
https://deepwiki.com/xuezhaojun/dashboard

quao.io/image , open-cluster-management / dashboard-api
https://quay.io/repository/open-cluster-management/dashboard-api?tab=tags

Monitoring OCM using Prometheus-Operator
https://open-cluster-management.io/docs/getting-started/administration/monitoring/
```

