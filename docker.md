```
sudo yum search docker
```
```
sudo yum install docker.x86_64 -y
```

```
sudo service docker status
```

```
sudo service docker start
```

```
sudo usermod -aG docker $USER
```

```
newgrp docker
```

```
docker pull busybox
docker pull nginx

```

```
docker images
```

>EKSClusterCreator: add action: ecr:*

* assume-role: EKSClusterCreator

```
aws ecr create-repository --region ap-northeast-2 --repository-name busybox
aws ecr create-repository --region ap-northeast-2 --repository-name nginx

```

```
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin $account.dkr.ecr.ap-northeast-2.amazonaws.com

```

```
docker tag busybox:latest $account.dkr.ecr.ap-northeast-2.amazonaws.com/busybox:latest
docker tag nginx:latest $account.dkr.ecr.ap-northeast-2.amazonaws.com/nginx:latest

```
```
docker push $account.dkr.ecr.ap-northeast-2.amazonaws.com/busybox:latest
docker push $account.dkr.ecr.ap-northeast-2.amazonaws.com/nginx:latest

```

---

# docker image for aws load balancer controller
```
docker pull quay.io/jetstack/cert-manager-cainjector:v1.5.4
docker pull quay.io/jetstack/cert-manager-controller:v1.5.4
docker pull quay.io/jetstack/cert-manager-webhook:v1.5.4
docker pull public.ecr.aws/eks/aws-load-balancer-controller:v2.4.7

```

```
docker images
```

>assume-role: EKSClusterCreator
```
aws sts get-caller-identity
```

```
aws ecr create-repository --region ap-northeast-2 --repository-name quay.io/jetstack/cert-manager-cainjector
aws ecr create-repository --region ap-northeast-2 --repository-name quay.io/jetstack/cert-manager-controller
aws ecr create-repository --region ap-northeast-2 --repository-name quay.io/jetstack/cert-manager-webhook
aws ecr create-repository --region ap-northeast-2 --repository-name eks/aws-load-balancer-controller

```

```
aws ecr get-login-password --region ap-northeast-2 | docker login --username AWS --password-stdin $account.dkr.ecr.ap-northeast-2.amazonaws.com

```

```
docker tag quay.io/jetstack/cert-manager-cainjector:v1.5.4 $account.dkr.ecr.ap-northeast-2.amazonaws.com/quay.io/jetstack/cert-manager-cainjector:v1.5.4
docker tag quay.io/jetstack/cert-manager-controller:v1.5.4 $account.dkr.ecr.ap-northeast-2.amazonaws.com/quay.io/jetstack/cert-manager-controller:v1.5.4
docker tag quay.io/jetstack/cert-manager-webhook:v1.5.4 $account.dkr.ecr.ap-northeast-2.amazonaws.com/quay.io/jetstack/cert-manager-webhook:v1.5.4
docker tag public.ecr.aws/eks/aws-load-balancer-controller:v2.4.7 $account.dkr.ecr.ap-northeast-2.amazonaws.com/eks/aws-load-balancer-controller:v2.4.7

```
```
docker push $account.dkr.ecr.ap-northeast-2.amazonaws.com/quay.io/jetstack/cert-manager-cainjector:v1.5.4
docker push $account.dkr.ecr.ap-northeast-2.amazonaws.com/quay.io/jetstack/cert-manager-controller:v1.5.4
docker push $account.dkr.ecr.ap-northeast-2.amazonaws.com/quay.io/jetstack/cert-manager-webhook:v1.5.4
docker push $account.dkr.ecr.ap-northeast-2.amazonaws.com/eks/aws-load-balancer-controller:v2.4.7

```

