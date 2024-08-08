---
title: Hosting VSCode Server on EC2 and CloudFront Distribution
author: haimtran
date: 08/08/2024
---

## Deploy

To deploy the stack, you can use CLI as the below command or use the AWS CloudFormation console.

```bash
aws cloudformation create-stack \
 --stack-name vscode-server-stack \
 --template-body file://vscode-server-stack.yaml \
 --capabilities CAPABILITY_NAMED_IAM
```

The vscode-server-stack.yaml stack will:

- Create an EC2 instance and install the [code server](https://github.com/coder/code-server) using UserData.
- Expose the code server via a CloudFront distribution.

## Parameters

You can change the following parameters by editing the vscode-server-stack.yaml.

- EC2 instance type to host the code server (default t2.medium)
- Amazon Machine Image (AMI) (default [latest amazon linux 2023 in us-west-2](https://docs.aws.amazon.com/linux/al2023/ug/ec2.html))
- Code server [release version](https://github.com/coder/code-server/releases) (default [4.9.11](https://github.com/coder/code-server/releases/download/v4.91.1/code-server-4.91.1-linux-amd64.tar.gz))
- Select an existing VPCID and SubnetID (otherwise default VPC is selected)

You can change configuration of the code server by editting config.yaml at the below path.

```bash
/home/ec2-user/.config/code-server/config.yaml
```

Here is a sample content of config.yaml with authentication disabled.

```yaml
bind-addr: 0.0.0.0:8080
auth: none
password: 8766aa4b66cf555763e9564d
cert: false
```

## Access Code Server

**Option 1.** The code server has been exposed via a CloudFront distribution. You can find the https endpoint in the CloudFormation console output after deploy 2-cloudfront-distribution.yaml.

**Option 2.** Use AWS System Manager (SSM) and port forwarding to forward the code server to local host as the following command. In this case, there are prerequisites:

- Setup [aws-cli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-quickstart.html)
- Setup AWS credentials for a profile in ~/.aws/credentials
- Replace INSTANCE_ID, PROFILE_NAME, REGION into the below command

```bash
aws ssm start-session \
--target <INSTANCE_ID> \
--document-name AWS-StartPortForwardingSessionToRemoteHost \
--parameters "{\"portNumber\":[\"8080\"],\"localPortNumber\":[\"8080\"],\"host\":[\"172.31.19.133\"]}" \
--profile <PROFILE_NAME>\
--region <REGION>
```
