```
export PROXY_IP=[YOUR PROXY IP ADDRESS]
export HTTP_PROXY=http://$PROXY_IP:3128
export HTTPS_PROXY=http://$PROXY_IP:3128
export NO_PROXY=169.254.169.254,*.ap-northeast-2.eks.amazonaws.com,oidc.eks.ap-northeast-2.amazonaws.com
export AWS_CA_BUNDLE=/home/ec2-user/squid.pem
```
