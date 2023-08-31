# Create a service account
* https://kubernetes-sigs.github.io/aws-load-balancer-controller/
* https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/aws-load-balancer-controller.html
* AmazonEKSLoadBalancerControllerRole-$cluster: policy
```
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.7/docs/install/iam_policy.json
```
```
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```

* AmazonEKSLoadBalancerControllerRole-$cluster: trust relationship
```
cat >load-balancer-role-trust-policy.json <<EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::111122223333:oidc-provider/oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:aud": "sts.amazonaws.com",
                    "oidc.eks.region-code.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:sub": "system:serviceaccount:kube-system:aws-load-balancer-controller"
                }
            }
        }
    ]
}
EOF

```

```
sed -i "s/111122223333/$account/g" ./load-balancer-role-trust-policy.json 
sed -i 's/region\-code/ap\-northeast\-2/g' ./load-balancer-role-trust-policy.json
sed -i "s/EXAMPLED539D4633E53DE1B71EXAMPLE/$oidc_id/g" ./load-balancer-role-trust-policy.json
```

```
cluster=$(aws eks list-clusters --query clusters --output text)
echo $cluster
```

```
aws iam create-role \
  --role-name AmazonEKSLoadBalancerControllerRole-$cluster \
  --assume-role-policy-document file://"load-balancer-role-trust-policy.json"

```

```
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::$account:policy/AWSLoadBalancerControllerIAMPolicy \
  --role-name AmazonEKSLoadBalancerControllerRole-payments
```


```
cat >aws-load-balancer-controller-service-account.yaml <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/name: aws-load-balancer-controller
  name: aws-load-balancer-controller
  namespace: kube-system
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::111122223333:role/AmazonEKSLoadBalancerControllerRole
EOF
```

```
sed -i "s/111122223333/$account/g" ./aws-load-balancer-controller-service-account.yaml
sed -i "s/AmazonEKSLoadBalancerControllerRole/AmazonEKSLoadBalancerControllerRole\-$cluster/g" ./aws-load-balancer-controller-service-account.yaml


```

```
kubectl apply -f aws-load-balancer-controller-service-account.yaml
```

```
kubectl get sa -n kube-system aws-load-balancer-controller -o yaml
```

```
aws ecr describe-repositories --query 'repositories[].[repositoryArn]' --output text
```

# cert-manager
```
curl -Lo cert-manager.yaml https://github.com/jetstack/cert-manager/releases/download/v1.5.4/cert-manager.yaml
```
```
sed -i.bak -e "s|quay|$account.dkr.ecr.ap-northeast-2.amazonaws.com/quay|" ./cert-manager.yaml
```
```
kubectl apply \
    --validate=false \
    -f ./cert-manager.yaml
```

# aws-load-balancer-controller
```
curl -Lo v2_4_7_full.yaml https://github.com/kubernetes-sigs/aws-load-balancer-controller/releases/download/v2.4.7/v2_4_7_full.yaml

```

```
echo $cluster
echo $account
```
```
sed -i.bak -e '561,569d' ./v2_4_7_full.yaml
sed -i.bak -e "s|your-cluster-name|$cluster|" ./v2_4_7_full.yaml
sed -i.bak -e "s|public.ecr.aws/eks/aws-load-balancer-controller|$account.dkr.ecr.ap-northeast-2.amazonaws.com/eks/aws-load-balancer-controller|" ./v2_4_7_full.yaml
```
```
kubectl apply -f v2_4_7_full.yaml
```

# ingress
```
curl -Lo v2_4_7_ingclass.yaml https://github.com/kubernetes-sigs/aws-load-balancer-controller/releases/download/v2.4.7/v2_4_7_ingclass.yaml
```

```
kubectl apply -f v2_4_7_ingclass.yaml
````
