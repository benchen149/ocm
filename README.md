#### Open Cluster Management (OCM)
```
wget -qO- https://github.com/open-cluster-management-io/clusteradm/releases/latest/download/clusteradm_linux_amd64.tar.gz | sudo tar -xvz -C /usr/local/bin/
sudo chmod +x /usr/local/bin/clusteradm

```

# Open Cluster Management (OCM) dashboard
```
helm --kube-context kind-hub install ocm-dashboard ./charts/ocm-dashboard \
  --namespace ocm-dashboard \
  --create-namespace

helm --kube-context kind-hub install ocm-dashboard ocm-dashboard \
  --namespace ocm-dashboard \
  --create-namespace \
  --set image.tag=latest \
  --set dashboard.env.DASHBOARD_BYPASS_AUTH=true \
  --set dashboard.env.DASHBOARD_DEBUG=true

helm --kube-context kind-hub upgrade ocm-dashboard charts/ocm-dashboard   --namespace ocm-dashboard

kubectl port-forward -n ocm-dashboard service/ocm-dashboard 3000:80

http://localhost:3000/login


export beartoken="eyJhbG..."

curl -L -X GET http://localhost:3000/api/clusters   -H "Authorization: Bearer $beartoken"   -H "Content-Type: application/json"

clusteradm get token --context  kind-hub | grep clusteradm
export joincmd=$(clusteradm get token --context  kind-hub | grep clusteradm)
$(echo ${joincmd} --force-internal-endpoint-lookup --wait --context kind-cluster2 | sed "s/<cluster_name>/kind-cluster2/g")
clusteradm accept --context kind-hub --clusters kind-cluster2 --wait

```

####
```
kubectl config use-context kind-hub
```

#### references
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