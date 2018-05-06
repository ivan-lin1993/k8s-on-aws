# k8s-on-aws

## Tool Request

- kubectl _(K8S CLI)_
- kops _(K8S用於AWS or GCP Operations的工具)_
- AWS CLI
- configures the AWS CLI
- creates an SSH key 
- create a s3 bucket for kops state


### Script

kubectl install
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

kops install 
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

Create AWS開機器的key
`
ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa
`

設定環境變數 \
  1. 可用的AZ (以 Oregon為例) \
  2. kops state的bucket
```
export AWS_AVAILABILITY_ZONES=us-west-2a,us-west-2b,us-west-2c
export KOPS_STATE_STORE=s3://bucket-blablabla    #bucket名稱自取
```

啟動Cluster
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
- kops validate cluster

查看創建進度
```
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
kubectl 是k8s的CLI, 掌管整個K8S的config, 可以更改config來達成不同需求的部屬
### cluster-info and version
```
kubectl cluster-info
kubectl version
```
