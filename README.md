# GKE Kube State Metrics

Create namesapce and grant permissions to the `default` user on the `monitoring` namespace.

```shell
kubectl apply -f rbac-setup.yaml --as=admin --as-group=system:masters
```

the requires permissions are `list,get,watch` on:

```console
$cat perms.txt

*v1.PersistentVolume
*v1.PersistentVolumeClaim
*v1.Service
*v1.Pod
*v1.Node
*v1.ReplicationController
*v1.Endpoints
*v1beta1.Deployment
*v1beta1.CronJob
*v1.Job
*v1beta1.ReplicaSet
*v2beta1.HorizontalPodAutoscaler
*v1beta1.DaemonSet
*v1beta1.StatefulSet
*v1.LimitRange
*v1.ResourceQuota
```

## Tiller Method

```shell
# Bootstrap
kubectl create ns monitoring

# Install
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get | bash

# Allow tiller to deploy
kubectl create serviceaccount tiller --namespace=monitoring
kubectl create rolebinding tiller --clusterrole=admin --serviceaccount=monitoring:tiller --namespace=monitoring

# If Tiller is not installed:
 helm init  --service-account tiller --tiller-namespace monitoring --debug
 ```


```shell
helm install stable/kube-state-metrics \
    --namespace monitoring \
    --name kube-state-metrics \
    --tiller-namespace monitoring

ENDPOINT=http://kube-state-metrics.monitoring.svc.cluster.local:8080

helm install stable/prometheus-to-sd \
    --namespace monitoring \
    --set metricsSources.kube-state-metrics=$ENDPOINT \
    --name prometheus-to-sd \
    --tiller-namespace monitoring
```

## Deployment

```shell
kubectl apply -f prom-to-sd-kube-state-metrics
```