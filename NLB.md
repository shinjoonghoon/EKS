
```
echo $vpcid
aws ec2 describe-subnets --filters "Name=vpc-id, Values=$vpcid"   --query "Subnets[*].{id:SubnetId,az:AvailabilityZone,subnet:Tags[?Key=='Name']|[0].Value}" --output text

```
```
cat >ngx-nlb.yaml <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: ngx1
  name: ngx1
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      run: ngx1
  template:
    metadata:
      labels:
        run: ngx1
    spec:
      containers:
      - image: $account.dkr.ecr.ap-northeast-2.amazonaws.com/nginx:latest
        imagePullPolicy: Always
        name: ngx1
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: external
    service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: instance
    service.beta.kubernetes.io/aws-load-balancer-scheme: internal
    service.beta.kubernetes.io/aws-load-balancer-subnets: subnet-0aa02b8424ab77883, subnet-05412c45573ba1763
  labels:
    run: ngx1
  name: ngx1
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  type: LoadBalancer
  selector:
    run: ngx1
EOF

```
```
kubectl create -f ngx-nlb.yaml
```
```
watch kubectl get svc ngx1
```
