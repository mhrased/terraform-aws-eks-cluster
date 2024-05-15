# terraform-aws-eks-cluster

A sample repository to create EKS on AWS using Terraform.

### Install AWS CLI

Need to install AWS CLI as we will use the AWS CLI (`aws configure`) command to connect Terraform with AWS.

Install AWS Cli on MacOS

```
brew install awscli
```

### Install Terraform

MacOS

```
brew tap hashicorp/tap
```

```
brew install hashicorp/tap/terraform
```

```
brew update
```

```
terraform -help
```

For Other OS one can follow terraform installation documentation link provided below:

```
https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli
```

### Connect Terraform with AWS

Its very easy to connect Terraform with AWS. Run

```
aws configure
```

command and provide the AWS Security credentials.

### To initialize the terraform use command

Run

```
terraform init
```

This will initialize the terraform environment for you and download the modules, providers and other configuration required.

### Validate terraform script

Run

```
terraform fmt
```

to formate.

Run

```
terraform validate
```

to validate terraform script.

### Optionally review the terraform configuration

Run

```
terraform plan
```

to see what resources will terraform create, modify, or delete.

### Finally, Apply terraform configuation to create EKS cluster with VPC

Run

```
terraform apply
```

to deploy the ESK on AWS.

### Destroy Resources

Run

```
terraform destroy
```

### Terraform Script Structure

    ### For Providers Script `versions.tf`
    ### For VPC Script `vpc.tf`
    ### For EKS CLuster `eks-cluster.tf`
    ### For Variables `variables.tf`
    ### For Security Group Script [inbound && outbound rules] `security-groups.tf`
    ### For Show Outputs in terminals `outputs.tf`

### Note

```
Need to Add ARN Manually to access EKS Cluster
```
