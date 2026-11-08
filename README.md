# Multi-Cloud Infrastructure as Code Labs

Infrastructure as code templates I wrote, debugged, and deployed across **AWS**, **Microsoft Azure**, and **Google Cloud** as part of CYB260L (Introduction to Cloud Solutions Lab) at ECPI University.

Each template was validated and deployed in a live cloud lab environment during the course. The commit history shows how each one evolved from a starter or broken template into a working deployment.

> **Note:** After the course ended, I revised the EC2 and RDS templates (key pair parameter, RDS security group, subnet group, and Secrets Manager password). These revisions were validated with [cfn-lint](https://github.com/aws-cloudformation/cfn-lint) but have not been redeployed, since the lab environment is no longer available.

## Repository Structure

```
aws/     AWS CloudFormation templates (YAML)
azure/   Azure Resource Manager (ARM) templates (JSON)
gcp/     Google Cloud Deployment Manager configs (YAML + Jinja)
```

## AWS (CloudFormation)

| Template | What it deploys |
|---|---|
| `aws/ec2-working.yaml` | Amazon Linux 2 EC2 instance (t2.micro) using the latest AMI from SSM Parameter Store, a 20 GB gp2 EBS volume, and a public IP. Takes an existing EC2 key pair as a parameter. |
| `aws/vpc-working.yaml` | Custom VPC (10.0.0.0/16) in us-west-2 with three public subnets across three Availability Zones, an internet gateway, a route table, and a default route. |
| `aws/rds-vpc.yaml` | The VPC above plus a MySQL RDS instance (db.t3.micro) in a DB subnet group spanning all three AZs. *(Post-course revisions validated with cfn-lint; see note above.)* |

**Security decisions in the RDS template**

- The master password is generated and stored by **AWS Secrets Manager** (`ManageMasterUserPassword: true`) instead of being written in the template.
- The database is **not publicly accessible**.
- A dedicated security group allows MySQL (port 3306) **only from inside the VPC** (10.0.0.0/16).

## Azure (ARM Templates)

| Template | What it deploys |
|---|---|
| `azure/vm1-azure.json` | App Service Plan (S1) and a Web App with fixed resource names. |
| `azure/vm2-azure.json` | The same App Service deployment, refactored to use **parameters and variables** so the app name and location can be set at deploy time. |

## Google Cloud (Deployment Manager)

| Template | What it deploys |
|---|---|
| `gcp/gcp-vpc.yaml` | Main config that imports the two Jinja templates below. |
| `gcp/calab-vpc.jinja` | VPC network with auto-created subnetworks. |
| `gcp/calab-bucket.jinja` | Cloud Storage bucket in us-central1. |

## Troubleshooting Highlights

Problems I ran into and fixed while getting these templates to deploy:

- **YAML indentation errors** that caused CloudFormation template validation to fail.
- **Incorrect resource type names** (typos in `AWS::` resource types) that stopped stacks from creating.
- **Wrong KeyName reference** in the EC2 template, which pointed at the AWS account ID instead of an EC2 key pair. Fixed by adding a `KeyName` parameter of type `AWS::EC2::KeyPair::KeyName`.
- **RDS networking:** the RDS instance originally had a placeholder security group and no subnet group, so it would not deploy into the custom VPC. Fixed by defining a security group and DB subnet group inside the template.
- **Hardcoded database credentials:** replaced with a password managed by Secrets Manager.

## How to Deploy

**AWS**
```bash
aws cloudformation validate-template --template-body file://aws/vpc-working.yaml
aws cloudformation create-stack --stack-name lab-vpc --template-body file://aws/vpc-working.yaml
```

**Azure**
```bash
az deployment group create --resource-group <your-rg> --template-file azure/vm2-azure.json --parameters webAppName=<name>
```

**Google Cloud**
```bash
gcloud deployment-manager deployments create lab-deployment --config gcp/gcp-vpc.yaml
```

## Skills Demonstrated

Infrastructure as code, AWS CloudFormation, Azure Resource Manager, Google Cloud Deployment Manager, VPC and subnet design, EC2 and EBS, Amazon RDS, AWS Secrets Manager, security groups, template validation, and troubleshooting.
