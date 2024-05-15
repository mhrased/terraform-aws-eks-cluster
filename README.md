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

### provider.tf

```
provider "aws":
This block configures the AWS provider to interact with AWS resources.
region specifies the AWS region where the resources will be provisioned. In this case, it's set to "ap-south-1", which is the Asia Pacific (Mumbai) region.
profile specifies the AWS CLI profile to use. In this case, it's set to "default", which is a commonly used profile name.

terraform:
This block specifies the Terraform configuration settings.
required_providers specifies the required providers for this Terraform configuration.
Under aws, it specifies the source and version constraints for the AWS provider.
source specifies the source of the provider. Here, it's set to "hashicorp/aws", which refers to the official AWS provider maintained by HashiCorp.
version specifies the version constraint for the provider. It's set to "~> 3.0", indicating that any version in the 3.x series is acceptable, with the expectation that the minor version should be at least 3.
```

### vpc.tf

```
Resource "aws_vpc" "aws-eks-vpc":
This block defines an AWS VPC resource.
cidr_block specifies the Classless Inter-Domain Routing (CIDR) block for the VPC's IP address range. In this case, it's set to "10.0.0.0/16", which allows for up to 65,536 IP addresses.
tags assigns tags to the VPC for identification and organization. In this case, it sets a tag with the key "Name" and the value "aws-eks-vpc".
```

### eks.tf

```
aws_iam_role resource for EKS:
This block defines an IAM role named "eks-cluster-eks" that is intended for use by Amazon EKS.
assume_role_policy specifies the trust policy that allows the EKS service to assume this role.

aws_iam_role_policy_attachment resource for attaching AmazonEKSClusterPolicy:
This block attaches the "AmazonEKSClusterPolicy" IAM policy to the IAM role created above.
policy_arn specifies the ARN (Amazon Resource Name) of the policy to attach.
role specifies the name of the IAM role to which the policy is attached.

aws_eks_cluster resource for creating the EKS cluster:
This block defines the Amazon EKS cluster itself.
name specifies the name of the EKS cluster.
role_arn specifies the ARN of the IAM role that provides permissions for the EKS cluster.
vpc_config specifies the VPC configuration for the EKS cluster, including the subnet IDs where the EKS nodes will be deployed.
depends_on ensures that the EKS cluster creation only starts after the IAM role policy attachment is complete.
```

### addons.tf

```
aws_eks_addon resource for CoreDNS:
This block defines an EKS addon for CoreDNS, which is a DNS server used for Kubernetes cluster networking.
cluster_name specifies the name of the EKS cluster to which this addon will be applied.
addon_name specifies the name of the addon, in this case, "coredns".

aws_eks_addon resource for kube-proxy:
This block defines an EKS addon for kube-proxy, which is a component responsible for handling network proxying for Kubernetes services.
cluster_name specifies the name of the EKS cluster to which this addon will be applied.
addon_name specifies the name of the addon, in this case, "kube-proxy".

aws_eks_addon resource for Amazon VPC CNI (Container Network Interface):
This block defines an EKS addon for the Amazon VPC CNI plugin, which is used for pod networking in Kubernetes clusters running on AWS.
cluster_name specifies the name of the EKS cluster to which this addon will be applied.
addon_name specifies the name of the addon, in this case, "vpc-cni".
```

### iam-oidc.tf

```
data "tls_certificate" "eks":
This block retrieves the TLS certificate associated with the OIDC issuer of the EKS cluster.
url specifies the OIDC issuer URL, which is obtained from the aws_eks_cluster resource.

resource "aws_iam_openid_connect_provider" "eks":
This block defines an IAM OIDC provider resource for the EKS cluster.
client_id_list specifies the list of allowed OAuth/OpenID Connect client IDs that can be used to authenticate to the OIDC provider. In this case, it's set to ["sts.amazonaws.com"], which is the client ID used by AWS Security Token Service (STS).
thumbprint_list specifies the list of SHA-1 thumbprints of the OIDC provider's server certificate(s). This thumbprint is computed from the TLS certificate obtained in the previous data source block.
url specifies the URL of the OIDC provider, which should match the issuer URL of the EKS cluster.
```

### internetgatwy.tf

```
resource "aws_internet_gateway" "igw":
This block defines an AWS Internet Gateway (IGW) resource.
vpc_id specifies the ID of the VPC to which the Internet Gateway will be attached. It references the VPC with the ID aws_vpc.aws-eks-vpc.id.
tags is a map of tags to apply to the Internet Gateway. In this case, it adds a tag with the key "Name" and the value "igw".
```

### nodes.tf

```
aws_iam_role resource for nodes:
This block defines an IAM role named "eks-node-group-nodes" for the EKS node group.
assume_role_policy specifies the trust policy allowing EC2 instances to assume this role.

aws_iam_role_policy_attachment resources:
Three attachments are defined to attach IAM policies to the IAM role created above:
AmazonEKSWorkerNodePolicy: Grants permissions required by Amazon EKS worker nodes to make calls to other AWS services on your behalf.
AmazonEKS_CNI_Policy: Grants permissions required by the Amazon VPC CNI plugin to function properly with Amazon EKS.
AmazonEC2ContainerRegistryReadOnly: Grants read-only access to the Amazon ECR (Elastic Container Registry) for pulling container images.

aws_eks_node_group resource for creating the EKS node group:
This block defines an EKS node group within the specified cluster.
cluster_name specifies the name of the EKS cluster.
node_group_name specifies the name of the node group.
node_role_arn specifies the IAM role ARN for the EKS nodes, which was created earlier.
subnet_ids specifies the IDs of the subnets where the EKS nodes will be launched.
capacity_type specifies the capacity type for the nodes. In this case, it's set to "ON_DEMAND".
instance_types specifies the instance types to be used for the nodes.
scaling_config specifies the scaling configuration for the node group.
update_config specifies the update configuration for the node group.
labels specifies any labels to be applied to the nodes.
depends_on ensures that the node group creation only starts after the IAM role policy attachments are complete.
```

### routes.tf

```
Resource "aws_route_table" "private":
This block defines a route table for private subnets.
vpc_id specifies the ID of the VPC to which the route table belongs.
route specifies the routes for this route table. In this case, there's a single route defined with a destination CIDR block of "0.0.0.0/0", which means all traffic not destined for the VPC's internal network will be routed elsewhere. The nat_gateway_id specifies the NAT gateway to route internet-bound traffic through.
tags applies a tag to the route table for identification.

Resource "aws_route_table" "public":
This block defines a route table for public subnets.
Similar to the private route table, it has a route for "0.0.0.0/0", but instead of a NAT gateway, it specifies an Internet Gateway (gateway_id) to route internet-bound traffic directly.
It also applies a tag for identification.

Resource "aws_route_table_association":
These blocks associate the route tables with the respective subnets.
Each association specifies a subnet ID (subnet_id) and the ID of the route table (route_table_id) to associate with that subnet.
```

### subnets.tf

```
Resource "aws_subnet" "private-ap-south-1a":
This block defines a private subnet in availability zone ap-south-1a.
vpc_id specifies the ID of the VPC to which this subnet belongs.
cidr_block specifies the CIDR block for the subnet's IP address range.
availability_zone specifies the availability zone for the subnet.
tags assigns tags to the subnet for identification and potentially for Kubernetes roles. In this case, it specifies tags related to Kubernetes, such as "kubernetes.io/role/internal-elb" and "kubernetes.io/cluster/demo".

Resource "aws_subnet" "private-ap-south-1b":
Similar to the previous subnet but for availability zone ap-south-1b.

Resource "aws_subnet" "public-ap-south-1a":
This block defines a public subnet in availability zone ap-south-1a.
map_public_ip_on_launch is set to true, which means that instances launched in this subnet will have public IP addresses automatically assigned.
Similar to the private subnets, it also has tags related to Kubernetes.

Resource "aws_subnet" "public-ap-south-1b":
Similar to the previous public subnet but for availability zone ap-south-1b.
```
