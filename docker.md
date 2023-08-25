```
sudo yum search docker
```
```
sudo yum install docker.x86_64 -y
```

```
docker pull busybox
docker pull nginx
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

