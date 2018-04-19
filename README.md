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






