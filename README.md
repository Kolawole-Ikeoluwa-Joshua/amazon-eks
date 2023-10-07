# Provision Amazon EKS K8s
Two ways to provision EKS Cluster in AWS
- AWS CLI https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html
- EKS CLI (newer) https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html

## AWS CLI

### Getting Started:

### 1. Create Free Tier Account

### 2. Configure AWS CLI

```
# Run Amazon CLI
docker run -it --rm -v ${PWD}:/work -w /work --entrypoint /bin/sh amazon/aws-cli:2.0.17

# switch directories
cd ./amazon

# install utilities
yum install jq gzip nano tar git

# Login to AWS
# Access your "My Security Credentials" section in your profile. 
# Create an access key

aws configure
```

### 3. Create IAM role

```
# create our role for EKS

# create role and extract ARN
role_arn=$(aws iam create-role --role-name test-cluster-eks-role --assume-role-policy-document file://assume-policy.json | jq .Role.Arn | sed s/\"//g)

# attach an AWS managed policy (AmazonEKSClusterPolicy) to the IAM role 
aws iam attach-role-policy --role-name test-cluster-eks-role --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy


```

4. Assign IAM role with Policies
5. Create VPC with 3 Subnets
6. Create EKS 0.1 USD per hour
7. Deploy EC2 in subnet & connect to EKS

### Bonus:

8. Deploy using EKS CLI
