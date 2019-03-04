# Creating a Kubernetes Cluster on AWS

![kubernetes_aws](./images/kops-aws-1.png)

This is a tutorial and a step by step walkthrough for setting up a kubernetes cluster on AWS using kops.


## Prerequisites

1. An AWS account

2. IAM Access Key ID & Secret

3. Kubectl Installed

4. Kops Installed

5. AWS CLI Installed

6. Jq Installed

## Create an Amazon S3 bucket for the Kubernetes state store

Create an S3 bucket:

```
aws s3api create-bucket --bucket k8s.yourdomainname.com --create-bucket-configuration LocationConstraint=us-east-1
```

Version your bucket-in-case you need to revert or recover a previous version:

```
aws s3api put-bucket-versioning --bucket k8s.yourdomainname.com --versioning-configuration Status=Enabled
```

For convenience, you can also define **KOPS_STATE_STORE** environment variable pointing to the S3 bucket:

```
export KOPS_STATE_STORE=s3://k8s.yourdomainname.com
```

## DNS Configuration

Generate a Route 53 hosted zone using the AWS CLI:

```
ID=$(uuidgen) && \
aws route53 create-hosted-zone \
--name k8s.yourdomainname.com \
--caller-reference $ID \
| jq .DelegationSet.NameServers
```

This gives the output shown below, add the output you get similar to the below to your DNS records:

```
[
    "ns-94.awsdns-11.com",
    "ns-1962.awsdns-s3.co.uk",
    "ns-838.awsdns-40.net",
    "ns-1107.awsdns-10.org"
]
```