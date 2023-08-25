# Install Tools
* https://kubernetes.io/docs/tasks/tools/
* https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
* https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/install-kubectl.html
* https://docs.aws.amazon.com/ko_kr/eks/latest/userguide/eksctl.html

* jq
```
sudo yum install jq
```

* aws cli
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
<!-- sudo ./aws/install -->
sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update
ls -l /usr/local/bin/aws
echo "complete -C '/usr/local/bin/aws_completer' aws" >> ~/.bash_profile
source ~/.bashrc

```

* kubectl
```
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.27.1/2023-04-19/bin/linux/amd64/kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
sudo chmod +x /usr/local/bin/kubectl
kubectl version

```

* eksctl
```
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo mv /tmp/eksctl /usr/local/bin

```

* autocomplete
```
echo "source <(kubectl completion bash)" >> ~/.bashrc
echo "source <(eksctl completion bash)" >> ~/.bashrc
source ~/.bashrc

```

# Security group of endpoints
* https://docs.aws.amazon.com/vpc/latest/privatelink/create-interface-endpoint.html
* create-security-group
```
aws ec2 create-security-group --description "Security group of endpoints" --group-name "Security group of endpoints" --vpc-id $vpcid --output json   | jq '.[]'

```

* authorize-security-group-ingress
```
aws ec2 authorize-security-group-ingress \
    --group-id $(aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`Security group of endpoints`)].GroupId' --output text  ) \
    --protocol -1 --port -1 --cidr 0.0.0.0/0 \
    --output json | jq '.[]'

```

* describe-subnets
```
sbs=$(aws ec2 describe-subnets   --filters "Name=vpc-id,Values=$vpcid" "Name=tag:Endpoints,Values=true" --region ap-northeast-2 --query 'Subnets[].SubnetId' --output text)
echo $sbs

```

* describe-route-tables
```
rts=$(aws ec2 describe-route-tables   --filters "Name=vpc-id,Values=$vpcid" "Name=tag:Endpoints,Values=true" --region ap-northeast-2 --query 'RouteTables[].RouteTableId' --output text)
echo $rts

```

* s3 vpc endpoint of type gateway
```
aws ec2 create-vpc-endpoint \
    --region $region \
    --vpc-endpoint-type Gateway \
    --vpc-id  $vpcid \
    --route-table-ids $rts \
    --service-name com.amazonaws.$region.s3 \
    --output json | jq '.[]'

```

* s3 vpc endpoint of type interface
```
myarray=(
com.amazonaws.$region.s3
)
for v in "${myarray[@]}"; do
    aws ec2 create-vpc-endpoint \
    --region $region \
    --vpc-id  $vpcid \
    --subnet-ids $sbs \
    --vpc-endpoint-type Interface \
    --service-name  $v \
    --security-group-ids $(aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`Security group of endpoints`)].GroupId' --output text  ) \
    --output json | jq '.[]'
done

```

* VPC interface endpoint for any AWS services that Pods need access to
* https://docs.aws.amazon.com/eks/latest/userguide/private-clusters.html
```
myarray=(
com.amazonaws.$region.appmesh-envoy-management
com.amazonaws.$region.cloudformation
com.amazonaws.$region.ec2
com.amazonaws.$region.ec2messages
com.amazonaws.$region.ecr.api
com.amazonaws.$region.ecr.dkr
com.amazonaws.$region.eks
com.amazonaws.$region.elasticloadbalancing
com.amazonaws.$region.kms
com.amazonaws.$region.logs
com.amazonaws.$region.ssm
com.amazonaws.$region.ssmmessages
com.amazonaws.$region.sts
com.amazonaws.$region.xray
)
for v in "${myarray[@]}"; do
    aws ec2 create-vpc-endpoint \
    --region $region \
    --vpc-id  $vpcid \
    --subnet-ids $sbs \
    --vpc-endpoint-type Interface \
    --service-name  $v \
    --security-group-ids $(aws ec2 describe-security-groups --query 'SecurityGroups[?(VpcId==`'$vpcid'` && GroupName==`Security group of endpoints`)].GroupId' --output text  ) \
    --output json | jq '.[]'
done

```

* describe-vpc-endpoints
```
aws ec2 describe-vpc-endpoints --filters "Name=vpc-id,Values=$vpcid" --region ap-northeast-2 --query 'VpcEndpoints[].[State,ServiceName]' --output text
```

