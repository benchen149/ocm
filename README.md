#### Pretask
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

#### OCM dashboard
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

#### others
```
kind get clusters
kind delete cluster --name c1
docker network inspect -f '{{range.IPAM.Config}}{{.Gateway}}{{end}}' kind
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

```