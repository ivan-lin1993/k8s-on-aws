# Kubernetes Cluster Monitoring

The Dashboard UI is not deployed by default. To deploy it, run the following command:
```
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

Dashboard url: \
https://\<Your ELB Domain\>/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/

進來會卡關~ 需要token
```
$ kubectl -n kube-system get secret

NAME                                             TYPE                                  DATA      AGE
attachdetach-controller-token-wbf2f              kubernetes.io/service-account-token   3         3h
aws-cloud-provider-token-lbwf5                   kubernetes.io/service-account-token   3         3h
certificate-controller-token-ff72s               kubernetes.io/service-account-token   3         3h
clusterrole-aggregation-controller-token-pnhdl   kubernetes.io/service-account-token   3         3h
cronjob-controller-token-2762g                   kubernetes.io/service-account-token   3         3h
daemon-set-controller-token-czgsb                kubernetes.io/service-account-token   3         3h
default-token-7f4vz                              kubernetes.io/service-account-token   3         3h
deployment-controller-token-9b6x7                kubernetes.io/service-account-token   3         3h
disruption-controller-token-xms8w                kubernetes.io/service-account-token   3         3h
dns-controller-token-j7sxm                       kubernetes.io/service-account-token   3         3h
endpoint-controller-token-nt5n2                  kubernetes.io/service-account-token   3         3h
generic-garbage-collector-token-4zj9t            kubernetes.io/service-account-token   3         3h
horizontal-pod-autoscaler-token-t2pgt            kubernetes.io/service-account-token   3         3h
job-controller-token-7t8zv                       kubernetes.io/service-account-token   3         3h
kube-dns-autoscaler-token-2qjnz                  kubernetes.io/service-account-token   3         3h
kube-dns-token-ndvd8                             kubernetes.io/service-account-token   3         3h
kube-proxy-token-qccd5                           kubernetes.io/service-account-token   3         3h
kubernetes-dashboard-certs                       Opaque                                0         5m
kubernetes-dashboard-key-holder                  Opaque                                2         5m
kubernetes-dashboard-token-vwc5s                 kubernetes.io/service-account-token   3         5m
namespace-controller-token-nn6fp                 kubernetes.io/service-account-token   3         3h
node-controller-token-89l5x                      kubernetes.io/service-account-token   3         3h
persistent-volume-binder-token-tdcqm             kubernetes.io/service-account-token   3         3h
pod-garbage-collector-token-bqz7w                kubernetes.io/service-account-token   3         3h
pv-protection-controller-token-rpl6p             kubernetes.io/service-account-token   3         3h
pvc-protection-controller-token-9vtwg            kubernetes.io/service-account-token   3         3h
replicaset-controller-token-xk97r                kubernetes.io/service-account-token   3         3h
replication-controller-token-pc4ls               kubernetes.io/service-account-token   3         3h
resourcequota-controller-token-qv45d             kubernetes.io/service-account-token   3         3h
route-controller-token-sb4pv                     kubernetes.io/service-account-token   3         3h
service-account-controller-token-84ckm           kubernetes.io/service-account-token   3         3h
service-controller-token-lljnr                   kubernetes.io/service-account-token   3         3h
statefulset-controller-token-58txf               kubernetes.io/service-account-token   3         3h
ttl-controller-token-689z6                       kubernetes.io/service-account-token   3         3h

```
#### Get Token
```
kubectl -n kube-system describe secret namespace-controller-token-nn6fp
```
### use token login