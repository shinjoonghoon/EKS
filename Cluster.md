# Security group of cluster
* https://docs.aws.amazon.com/cli/latest/reference/ec2/authorize-security-group-ingress.html
* https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/sec-group-reqs.html

```
vpcid=vpc-xxxxxxxxxxxxxxxxx
```
```
region=$(aws configure get region --profile eksadmin)
echo $vpcid
echo $region

```
* create-security-group
```
aws ec2 create-security-group --description "Security group of Payments Cluster" --group-name "eks-cluster-sg-payments" --vpc-id $vpcid --output json --region $region | jq '.[]'

```

* authorize-security-group-ingress
>source-group
```
aws ec2 authorize-security-group-ingress \
    --group-id $(aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`eks-cluster-sg-payments`)].GroupId' --output text --region $region ) \
    --protocol -1 --port -1 \
    --source-group $(aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`eks-cluster-sg-payments`)].GroupId' --output text --region $region ) \
    --region $region \
    --output json | jq '.[]'

```
* authorize-security-group-ingress
>cidr 10.0.0.0/8
```
aws ec2 authorize-security-group-ingress \
    --group-id $(aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`eks-cluster-sg-payments`)].GroupId' --output text --region $region  ) \
    --protocol tcp \
    --port 443 \
    --cidr 10.0.0.0/8 \
    --region $region \
    --output json | jq '.[]'

```

# Creating an Amazon EKS cluster
* https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/create-cluster.html
* describe-security-groups
```
aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`eks-cluster-sg-payments`)].GroupId' --output text --region $region
```

* describe-subnets
```
aws ec2 describe-subnets --filters "Name=vpc-id, Values=$vpcid"   --query "Subnets[*].{id:SubnetId,az:AvailabilityZone,subnet:Tags[?Key=='Name']|[0].Value}" --output text --region $region
```

* ClusterConfig
* https://eksctl.io/usage/schema/
>privateCluster

>skipEndpointCreation

>securityGroup
```
cat << EOF > cluster-payments.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: payments
  region: ap-northeast-2
  version: "1.27"
privateCluster:
  enabled: true
  skipEndpointCreation: true
vpc:
  securityGroup: <>
  subnets:
    private:
      EKS-subnet-private2-ap-northeast-2a:
        id: <>
      EKS-subnet-private5-ap-northeast-2c:
        id: <>
EOF

```

* dry-run
* https://eksctl.io/usage/dry-run/
```
eksctl create cluster -f cluster-payments.yaml --dry-run
```

* create cluster
```
eksctl create cluster -f cluster-payments.yaml
```

* kube/config
```
kubectl config view
```
```
aws sts get-caller-identity
```
```
eksctl get cluster --region ap-northeast-2
```

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

# Delete cluster
>assume-role: EKSClusterCreator
```
eksctl delete cluster --name payments --region ap-northeast-2
```

# (Optional) KUBECONFIG
```
export KUBECONFIG=~/.kube/config-ekssso-EKSClusterAdmin 

```
