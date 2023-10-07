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

### 3. Create IAM role & Assign IAM role with Policies

```
# create our role for EKS

# create role and extract ARN
role_arn=$(aws iam create-role --role-name <custom-role-name> --assume-role-policy-document file://assume-policy.json | jq .Role.Arn | sed s/\"//g)

# attach an AWS managed policy (AmazonEKSClusterPolicy) to the IAM role
aws iam attach-role-policy --role-name <custom-role-name> --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
```

### 4. Create VPC with Subnets for Cluster

```
# download VPC CloudFormation template
curl https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-05-08/amazon-eks-vpc-sample.yaml -o vpc.yaml

# deploy VPC with Public subnets using CloudFormation
aws cloudformation deploy --template-file vpc.yaml --stack-name <custom-cf-stack>

# get stack details
# take not of vpc subnet ids
aws cloudformation list-stack-resources --stack-name <custom-cf-stack> > stack.json
```
