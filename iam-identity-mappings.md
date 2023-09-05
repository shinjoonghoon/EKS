# Manage IAM users and roles
* https://eksctl.io/usage/iam-identity-mappings/
```
ssorole=$(aws sts get-caller-identity --query Arn --output text --profile eksadmin | cut -d/ -f2)
account=$(aws sts get-caller-identity --query Account --output text --profile eksadmin)
cluster=$(aws eks list-clusters --query clusters --output text)
echo $ssorole
echo $account
echo $cluster

```

```
kubectl get cm -n kube-system aws-auth -o yaml

```

```
eksctl create iamidentitymapping \
 --cluster ${cluster} \
 --arn arn:aws:iam::${account}:role/${ssorole} \
 --username cluster-admin \
 --group system:masters \
 --region ap-northeast-2

```

```
kubectl get cm -n kube-system aws-auth -o yaml
```

```
unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN

```

```
aws eks update-kubeconfig --name ${cluster} \
 --profile eksadmin
```
>AccessDeniedException: eks:DescribeCluster

```
kubectl get nodes

```
