```
unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN
```

```
cat << EOF > bb8.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: bb8
  name: bb8
spec:
  replicas: 1
  selector:
    matchLabels:
      run: bb8
  template:
    metadata:
      labels:
        run: bb8
    spec:
      containers:
      - args:
        - sleep
        - "50000"
        image: $account.dkr.ecr.ap-northeast-2.amazonaws.com/busybox:latest
        name: bb8
EOF
```

```
kubectl config current-context
```

```
kubectl create -f bb8.yaml
```
```
cat << EOF > ngx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: ngx
  name: ngx
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      run: ngx
  template:
    metadata:
      labels:
        run: ngx
    spec:
      containers:
      - image: $account.dkr.ecr.ap-northeast-2.amazonaws.com/nginx:latest
        imagePullPolicy: Always
        name: ngx
---
apiVersion: v1
kind: Service
metadata:
  labels:
    run: ngx
  name: ngx
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    run: ngx

EOF

```
```
kubectl create -f ngx.yaml
```

```
kubectl get all -A -o wide
```

```
kubectl exec -it bb8-7f64d856f8-2755b -- sh
```

```
wget -q -S -O - ngx
```
```
wget -q -S -O - ngx.default
```
```
wget -q -S -O - ngx.default.svc
```
```
wget -q -S -O - ngx.default.svc.cluster.local
```
