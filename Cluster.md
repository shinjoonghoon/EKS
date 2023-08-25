# Security group of cluster
* https://docs.aws.amazon.com/cli/latest/reference/ec2/authorize-security-group-ingress.html
* https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/sec-group-reqs.html

```
vpcid=vpc-0e5dcdc77bf5fcc86
region=$(aws configure get region  )
echo $vpcid
echo $region

```

```
aws ec2 create-security-group --description "Security group of Payments Cluster" --group-name "eks-cluster-sg-payments" --vpc-id $vpcid --output json | jq '.[]'
```

```
aws ec2 authorize-security-group-ingress \
    --group-id $(aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`eks-cluster-sg-payments`)].GroupId' --output text  ) \
    --protocol -1 --port -1 \
    --source-group $(aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`eks-cluster-sg-payments`)].GroupId' --output text  ) \
    --output json | jq '.[]'
```

```
aws ec2 authorize-security-group-ingress \
    --group-id $(aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`eks-cluster-sg-payments`)].GroupId' --output text  ) \
    --protocol tcp \
    --port 443 \
    --cidr 10.0.0.0/8 \
    --output json | jq '.[]'



# Creating an Amazon EKS cluster
* https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/create-cluster.html
* https://eksctl.io/usage/schema/

```
aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`eks-cluster-sg-payments`)].GroupId' --output text
```

```
aws ec2 describe-subnets --filters "Name=vpc-id, Values=$vpcid"   --query "Subnets[*].{id:SubnetId,az:AvailabilityZone,subnet:Tags[?Key=='Name']|[0].Value}" --output text
```

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
#eks-cluster-sg-[Cluster Name]
  securityGroup: sg-0eb70e4b38716fba7
  subnets:
    private:
      EKS-subnet-private2-ap-northeast-2a:
        id: "subnet-00936ff6a3197cfaf"
      EKS-subnet-private5-ap-northeast-2c:
        id: "subnet-0be3341072b0df062"
EOF

```

