# k8s-on-aws

## Tool Request

- kubectl _(K8S CLI)_
- kops _(K8S用於AWS or GCP Operations的工具)_
- AWS CLI
- configures the AWS CLI
- creates an SSH key 
- create a s3 bucket for kops state


### Script

#### kubectl install
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

#### kops install 
```
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops
```

download awscli and configure your key ...
```
pip install awscli --upgrade --user
aws configure
```

blablabla...

#### 新增AWS開機器的key
```
ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa
```

設定環境變數 \
  1. 可用的AZ (以 Oregon為例) \
  2. kops state的bucket
```
export AWS_AVAILABILITY_ZONES=us-west-2a,us-west-2b,us-west-2c
export KOPS_STATE_STORE=s3://bucket-blablabla    #bucket名稱自取
```

## Start with kops
### Create cluster
--name (填入Route53建立的domain name， 如果有public就可以填入 也可以填local做private)\
這裡以 ecv.k8s.local 為例\
預設是1個master node 與2個worker nodes
```
kops create cluster \
  --name ecv.k8s.local \
  --zones $AWS_AVAILABILITY_ZONES
```

### Muti-Clusters Example
```
kops create cluster \
  --name ecv.k8s.local \
  --master-count 3 \
  --node-count 5 \
  --zones $AWS_AVAILABILITY_ZONES \
```

當create的時候, kops會去掃描AWS上的資源看你的環境是否可以創這些資源(iam, ec2, elb...)

有幾個指令可用, 可以看看裡面的參數
```
kops get cluster                            // list clusters
kops edit cluster ecv.k8s.local             // edit cluster
kops edit ig --name=ecv.k8s.local nodes     // edit worker node group
kops edit ig --name=ecv.k8s.local master-us-west-2a     // edit master node group
```

修改cluster instancegroup的參數 ig = instancegroup
```
kops edit ig --name=ecv.k8s.local nodes
```

會以vim開啟以下資訊
```
apiVersion: kops/v1alpha2
kind: InstanceGroup
metadata:
  creationTimestamp: 2018-04-26T16:12:54Z
  labels:
    kops.k8s.io/cluster: ecv.k8s.local
  name: nodes
spec:
  image: kope.io/k8s-1.8-debian-jessie-amd64-hvm-ebs-2018-02-08  #AWS預設之AMI
  machineType: t2.medium  # 針對機型做更改
  maxSize: 2  # cluster內含的機器數量設定
  minSize: 2
  nodeLabels:
    kops.k8s.io/instancegroup: nodes
  role: Node
  subnets:
  - us-west-2a
  - us-west-2b
  - us-west-2c
```
### (Optional)
如果要開Spot機型, 在spec底下多加一個參數 maxPrice
```
spec:
  maxPrice: "0.02"
```
以max price開t2.medium的spot instances

### 修改完後update
```
kops update cluster ecv.k8s.local --yes
```
開始實際在AWS上創建資源, 約3~5min
- 修改後所有的資訊都會update到剛剛設定的s3 bucket上

查看創建進度
```
$ kops validate cluster

INSTANCE GROUPS
NAME                    ROLE    MACHINETYPE     MIN     MAX     SUBNETS
master-us-west-2a       Master  t2.medium       1       1       us-west-2a
nodes                   Node    t2.micro        2       2       us-west-2a,us-west-2b,us-west-2c

NODE STATUS
NAME                                            ROLE    READY
ip-172-20-43-65.us-west-2.compute.internal      node    True
ip-172-20-43-67.us-west-2.compute.internal      master  True
ip-172-20-75-169.us-west-2.compute.internal     node    True
```
Note: 如果為muti-cluster, 所有的masters會座落在不同的AZ~

## Start with kubectl
kubectl (cube control) 是 K8S 的 CLI, 掌管整個 K8S 的 config, 將指令透過 API 的方式打到 master node 上, 更改 config 來達成不同需求的部屬
### cluster-info and version
```
kubectl cluster-info
kubectl version
```
### Display Nodes
```
kubectl get nodes
```
### Create Pod
```
A Pod is the smallest deployable unit that can be created, scheduled, and managed. It’s a logical collection of containers that belong to an application. Pods are created in a namespace. All containers in a pod share the namespace, volumes and networking stack. This allows containers in the pod to “find” each other and communicate using localhost.
```
在cluster中建立 pod 裡面包含了 nginx container 
```
kubectl run nginx --image=nginx
```
### List deployment 
```
$ kubectl get deployments

NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx     1         1         1            1           41s
```
### List running pods
```
$ kubectl get pods
NAME                   READY     STATUS    RESTARTS   AGE
nginx-8586cf59-gmqc9   1/1       Running   0          3m
```

### 看 pod 的詳細資訊
```
$ kubectl describe pod/nginx-8586cf59-gmqc9

Name:           nginx-8586cf59-gmqc9
Namespace:      default
Node:           ip-172-20-116-189.us-west-2.compute.internal/172.20.116.189
Start Time:     Tue, 15 May 2018 09:15:28 +0000
Labels:         pod-template-hash=41427915
                run=nginx
Annotations:    kubernetes.io/limit-ranger=LimitRanger plugin set: cpu request for container nginx
Status:         Running
IP:             100.96.2.4
Controlled By:  ReplicaSet/nginx-8586cf59
Containers:
  nginx:

blablabla...
```
裡面會有詳細的結構, 另外 namespace 的 default 值是 "default"\
可以來看看另外的保留 namespace "kube-system"

```
$ kubectl get pods --namespace kube-system

NAME                                                                 READY     STATUS    RESTARTS   AGE
dns-controller-769b5f68b6-lx2vr                                      1/1       Running   0          32m
etcd-server-events-ip-172-20-60-21.us-west-2.compute.internal        1/1       Running   0          31m
etcd-server-ip-172-20-60-21.us-west-2.compute.internal               1/1       Running   0          31m
kube-apiserver-ip-172-20-60-21.us-west-2.compute.internal            1/1       Running   0          32m
kube-controller-manager-ip-172-20-60-21.us-west-2.compute.internal   1/1       Running   0          32m
kube-dns-7785f4d7dc-4c972                                            3/3       Running   0          30m
kube-dns-7785f4d7dc-trq7w                                            3/3       Running   0          32m
kube-dns-autoscaler-787d59df8f-6qb8c                                 1/1       Running   0          32m
kube-proxy-ip-172-20-116-189.us-west-2.compute.internal              1/1       Running   0          30m
kube-proxy-ip-172-20-60-21.us-west-2.compute.internal                1/1       Running   0          31m
kube-proxy-ip-172-20-75-20.us-west-2.compute.internal                1/1       Running   0          30m
kube-scheduler-ip-172-20-60-21.us-west-2.compute.internal            1/1       Running   0          32m

```

### Log from the Pod
```
$ kubectl logs nginx-8586cf59-gmqc9 --namespace default
```
(剛開的nginx不會有任何log)

### Execute a shell on the running pod

This command will open a TTY to a shell in your pod:
```
$ kubectl exec -it nginx-8586cf59-gmqc9 /bin/bash
```

### (Optional) 試著把 Pod 刪除 
```
$ kubectl delete pods/nginx-8586cf59-gmqc9

$ kubectl get pods
NAME                   READY     STATUS              RESTARTS   AGE
nginx-8586cf59-gmqc9   0/1       Terminating         0          2m
nginx-8586cf59-j7fqc   0/1       ContainerCreating   0          4s
```
### Delete Deployment
```
$ kubectl delete deployment/nginx
```

### Create another Pod using yaml file

```
$ cat pod.yaml

apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```
執行
```
$ kubectl apply -f pod.yaml
pod "nginx-pod" created
```

### 設定 Memory 與 CPU
在 Pod 中以 _request_ ( memory/CPU 的最小值) 或 _limit_ ( 最大值 ) 來設定

<table border=1>
<tr><th>type</th><th>Field</th></tr>
<tr><td>Memory request</td><td>spec.containers[].resources.requests.memory</td></tr>
<tr><td>Memory limit</td><td>spec.containers[].resources.limits.memory</td></tr>
<tr><td>CPU request</td><td>spec.containers[].resources.requests.cpu</td></tr>
<tr><td>CPU limit</td><td>spec.containers[].resources.limits.cpu</td></tr>
</table>

CPU can be requested in cpu units. 1 cpu unit is equivalent 1 AWS vCPU. It can also be requested in fractional units, such as 0.5 or in millicpu such as 500m.


