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

設定環境變數 
  1. 可用的AZ (以 Oregon為例)
  2. kops state的bucket
```
export AWS_AVAILABILITY_ZONES=us-west-2a,us-west-2b,us-west-2c
export KOPS_STATE_STORE=s3://bucket-blablabla    #bucket名稱自取
```

啟動Cluster
--name {之後 Route53建立的domain name， 如果有public就可以填入 也可以填local來}
```
kops create cluster \
  --name ecv.k8s.local \
  --zones $AWS_AVAILABILITY_ZONES
```

修改cluster instancegroup的參數 ig = instancegroup
```
kops edit instancegroup --name=ecv.k8s.local nodes
```

會看到如下的資訊
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
  machineType: t2.micro  # 針對機型做更改
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
(Optional)
如果要開Spot機型



