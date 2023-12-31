# Creating a managed node group
* https://docs.aws.amazon.com/eks/latest/userguide/create-managed-node-group.html
```
aws eks list-clusters --profile eksadmin
```

```
eksctl get cluster --region ap-northeast-2 --profile eksadmin
```
>AccessDenied: cloudformation:ListStacks

>refresh sso token

# Create Amazon EKS node role
* https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/create-node-role.html
* AmazonEKSNodeRole
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```
>AmazonEKSWorkerNodePolicy

>AmazonEC2ContainerRegistryReadOnly

>AmazonEKS_CNI_Policy

# YAML
```
cat << EOF > managed-node-group-payments.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: payments
  region: ap-northeast-2

managedNodeGroups:
  - name: ng1
    instanceType: t3.medium
    desiredCapacity: 2
    privateNetworking: true
    subnets:
      - EKS-subnet-private2-ap-northeast-2a
      - EKS-subnet-private5-ap-northeast-2c

vpc:
  securityGroup: sg-0ccad43e99f56ba73
  subnets:
    private:
      EKS-subnet-private2-ap-northeast-2a:
        id: subnet-00626a9db52508624
      EKS-subnet-private5-ap-northeast-2c:
        id: subnet-05412c45573ba1763

EOF

```

* assume-role: EKSClusterCreator
```
kubectl config use-context create-eks-cluster@payments.ap-northeast-2.eksctl.io
```

```
eksctl create nodegroup -f managed-node-group-payments.yaml
```

```
kubectl get nodes
```

```
unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN
```

```
kubectl config use-context arn:aws:eks:ap-northeast-2:$account:cluster/payments
```

```
kubectl get nodes
```
