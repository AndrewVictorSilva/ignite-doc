# Infraestrutura com Terraform

Bem-vindo à seção de criação da infraestrutura virtual na AWS com Terraform do nosso guia.

Nesta seção, abordaremos passo a passo como configurar a infraestrutura necessária para o nosso cluster EKS (Elastic Kubernetes Service). 

A infraestrutura que iremos criar envolve elementos cruciais, incluindo:

- VPC (Virtual Private Cloud)
- Subnets
- Tabelas de roteamento
- NAT gateways
- Cluster EKS
- Nós de computação (compute nodes)
- Addons de rede, como CNI (Container Network Interface), Kube Proxy, EBS CSI Driver e CoreDNS

Com a utilização do Terraform, uma poderosa ferramenta de infraestrutura como código, seremos capazes de provisionar toda essa infraestrutura de forma automatizada e repetível. Isso nos permitirá criar e gerenciar nosso ambiente de maneira eficiente e escalável.

Ao final desta seção, você terá uma infraestrutura sólida e pronta para implantar e executar aplicativos em contêineres no nosso cluster EKS.

### Virtual Private Cloud (VPC)
``` tf title="vpc.tf"
 provider "aws"{
        region = "região"
    }

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}

resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  instance_tenancy     = "default"
  enable_dns_hostnames = true

  tags = {
    "Name" = "main"
  }
}

terraform {
  backend "s3" {
    bucket  = "bucket_name"
    key     = "folder/terraform.tfstate"
    region  = "region"
    profile = "profile_name"
  }
}

```

### Internet Gateway
``` tf title="igw.tf"
    resource "aws_internet_gateway" "igw" {
        vpc_id = aws_vpc.main.id

        tags = {
            Name = "igw"
        }
    }
```

### Range de IP's

|    Subnet             |      CIDR         |  Endereços Disponíveis  |
|-----------------------|-------------------|-------------------------|
| private-us-east-1a     | 10.0.0.0/19      | 8190                    |
| private-us-east-1b    | 10.0.32.0/19     | 8190                    |
| public-us-east-1a     | 10.0.64.0/19     | 8190                    |
| public-us-east-1b     | 10.0.96.0/19     | 8190                    |


### Subnets
``` tf title="subnets.tf"
resource "aws_subnet" "private-us-east-1a" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.0.0/19"
  availability_zone = "us-east-1a"

  tags = {
    "Name"                            = "private-us-east-1a"
    "kubernetes.io/role/internal-elb" = "1"
    "kubernetes.io/cluster/demo"      = "shared"
  }
}
resource "aws_subnet" "private-us-east-1b" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.32.0/19"
  availability_zone = "us-east-1b"

  tags = {
    "Name"                            = "private-us-east-1b"
    "kubernetes.io/role/internal-elb" = "1"
    "kubernetes.io/cluster/demo"      = "shared"
  }
}

resource "aws_subnet" "public-us-east-1a" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.64.0/19"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true

  tags = {
    "Name"                       = "public-us-east-1a"
    "kubernetes.io/role/elb"     = "1"
    "kubernetes.io/cluster/demo" = "shared"
  }
}

resource "aws_subnet" "public-us-east-1b" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.96.0/19"
  availability_zone       = "us-east-1b"
  map_public_ip_on_launch = true

  tags = {
    "Name"                       = "public-us-east-1b"
    "kubernetes.io/role/elb"     = "1"
    "kubernetes.io/cluster/demo" = "shared"
  }
}
```

### NAT Gateway
``` tf title="nat.tf"
resource "aws_eip" "nat" {
  vpc = true

  tags = {
    Name = "nat"
  }
}

resource "aws_nat_gateway" "nat" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public-us-east-1a.id

  tags = {
    Name = "nat"
  }

  depends_on = [aws_internet_gateway.igw]
}
```

### Routes (Route Table)
``` tf title="routes.tf"
resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  route = [
    {
      cidr_block                 = "0.0.0.0/0"
      nat_gateway_id             = aws_nat_gateway.nat.id
      carrier_gateway_id         = ""
      destination_prefix_list_id = ""
      egress_only_gateway_id     = ""
      gateway_id                 = ""
      instance_id                = ""
      ipv6_cidr_block            = ""
      local_gateway_id           = ""
      network_interface_id       = ""
      transit_gateway_id         = ""
      vpc_endpoint_id            = ""
      vpc_peering_connection_id  = ""
      core_network_arn           = ""
    },
  ]

  tags = {
    Name = "private"
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route = [
    {
      cidr_block                 = "0.0.0.0/0"
      gateway_id                 = aws_internet_gateway.igw.id
      nat_gateway_id             = ""
      carrier_gateway_id         = ""
      destination_prefix_list_id = ""
      egress_only_gateway_id     = ""
      instance_id                = ""
      ipv6_cidr_block            = ""
      local_gateway_id           = ""
      network_interface_id       = ""
      transit_gateway_id         = ""
      vpc_endpoint_id            = ""
      vpc_peering_connection_id  = ""
      core_network_arn           = ""
    },
  ]

  tags = {
    Name = "public"
  }
}

resource "aws_route_table_association" "private-us-east-1a" {
  subnet_id      = aws_subnet.private-us-east-1a.id
  route_table_id = aws_route_table.private.id
}

resource "aws_route_table_association" "private-us-east-1b" {
  subnet_id      = aws_subnet.private-us-east-1b.id
  route_table_id = aws_route_table.private.id
}

resource "aws_route_table_association" "public-us-east-1a" {
  subnet_id      = aws_subnet.public-us-east-1a.id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "public-us-east-1b" {
  subnet_id      = aws_subnet.public-us-east-1b.id
  route_table_id = aws_route_table.public.id
}

```

### Elastic Kubernets Service (EKS)
``` tf title="eks.tf"
resource "aws_iam_role" "demo" {
  name = "eks-cluster-demo"

  assume_role_policy = <<POLICY
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "eks.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
POLICY
}

resource "aws_iam_role_policy_attachment" "demo-AmazonEKSClusterPolicy" {
  for_each = toset([
    "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy",
    "arn:aws:iam::aws:policy/AmazonEKSServicePolicy",
  ])
  role       = aws_iam_role.demo.name
  policy_arn = each.value
}

resource "aws_eks_cluster" "demo" {
  name     = "demo"
  role_arn = aws_iam_role.demo.arn

  vpc_config {
    subnet_ids = [
      aws_subnet.private-us-east-1a.id,
      aws_subnet.private-us-east-1b.id,
    ]
  }

  depends_on = [
    aws_iam_role_policy_attachment.demo-AmazonEKSClusterPolicy,
  ]
}

```

### Cluster Nodes
``` tf title="nodes.tf"
resource "aws_iam_role" "nodes" {
  name = "eks-node-group-nodes"

  assume_role_policy = jsonencode({
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
    }]
    Version = "2012-10-17"
  })
}

resource "aws_iam_role_policy_attachment" "nodes-AmazonEKSWorkerNodePolicy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy"
  role       = aws_iam_role.nodes.name
}

resource "aws_iam_role_policy_attachment" "nodes-AmazonEKS_CNI_Policy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy"
  role       = aws_iam_role.nodes.name
}

resource "aws_iam_role_policy_attachment" "nodes-AmazonEC2ContainerRegistryReadOnly" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"
  role       = aws_iam_role.nodes.name
}

resource "aws_eks_node_group" "private-nodes" {
  cluster_name    = aws_eks_cluster.demo.name
  node_group_name = "private-nodes"
  node_role_arn   = aws_iam_role.nodes.arn

  subnet_ids = [
    aws_subnet.private-us-east-1a.id,
    aws_subnet.private-us-east-1b.id
  ]

  capacity_type  = "ON_DEMAND"
  instance_types = ["t3.medium"]

  scaling_config {
    desired_size = 2
    max_size     = 3
    min_size     = 1
  }

  update_config {
    max_unavailable = 1
  }

  labels = {
    role = "general"
  }

  tags = {
    "kubernetes.io/cluster/demo" = "shared"
  }

  depends_on = [
    aws_iam_role_policy_attachment.nodes-AmazonEKSWorkerNodePolicy,
    aws_iam_role_policy_attachment.nodes-AmazonEKS_CNI_Policy,
    aws_iam_role_policy_attachment.nodes-AmazonEC2ContainerRegistryReadOnly,
    aws_eks_addon.kube-proxy,
    aws_eks_addon.vpc-cni,
  ]
}

```

### Addons
``` tf title="addons.tf"


resource "aws_eks_addon" "vpc-cni" {
  cluster_name = aws_eks_cluster.demo.name
  addon_name   = "vpc-cni"
}

resource "aws_eks_addon" "kube-proxy" {
  cluster_name = aws_eks_cluster.demo.name
  addon_name   = "kube-proxy"
}

resource "aws_eks_addon" "aws-ebs-csi-driver" {
  cluster_name             = aws_eks_cluster.demo.name
  addon_name               = "aws-ebs-csi-driver"
  depends_on               = [aws_eks_node_group.private-nodes]
  service_account_role_arn = aws_iam_role.AmazonEKS_EBS_CSI_DriverRole.arn
}

resource "aws_eks_addon" "coredns" {
  cluster_name = aws_eks_cluster.demo.name
  addon_name   = "coredns"
  depends_on   = [aws_eks_node_group.private-nodes]
}
```

### Roles/ Authorization
``` tf title="oidc.tf"
resource "aws_iam_openid_connect_provider" "demo" {
  client_id_list  = ["sts.amazonaws.com"]
  thumbprint_list = data.tls_certificate.thumb.certificates[*].sha1_fingerprint
  url             = aws_eks_cluster.demo.identity.0.oidc.0.issuer
}
data "tls_certificate" "thumb" {
  url = aws_eks_cluster.demo.identity[0].oidc[0].issuer
}

data "aws_iam_policy_document" "aws-ebs-csi-driver-trust-policy" {
  statement {
    actions = ["sts:AssumeRoleWithWebIdentity"]
    effect  = "Allow"

    condition {
      test     = "StringEquals"
      variable = "${replace(aws_iam_openid_connect_provider.demo.url, "https://", "")}:aud"
      values   = ["sts.amazonaws.com"]
    }

    condition {
      test     = "StringEquals"
      variable = "${replace(aws_iam_openid_connect_provider.demo.url, "https://", "")}:sub"
      values   = ["system:serviceaccount:kube-system:ebs-csi-controller-sa"]
    }

    principals {
      identifiers = ["${aws_iam_openid_connect_provider.demo.arn}"]
      type        = "Federated"
    }
  }
}

resource "aws_iam_role" "AmazonEKS_EBS_CSI_DriverRole" {
  assume_role_policy = data.aws_iam_policy_document.aws-ebs-csi-driver-trust-policy.json
  name               = "AmazonEKS_EBS_CSI_DriverRole"
}

resource "aws_iam_role_policy_attachment" "ebs_csi_controller_policy" {
  policy_arn = "arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy"
  role       = aws_iam_role.AmazonEKS_EBS_CSI_DriverRole.name
}

```


### Load Balancer Controller

Aqui está um examplo de que o Terraform pode ser usado para executar o charts de Helm.
Criamos o chart usando as permissõesc criadas anteriormente e dinamicamente alocamos "trust relationship" na role que será usada na service account.

``` tf title="helm.tf"
provider "helm" {
  kubernetes {
    host                   = aws_eks_cluster.demo.endpoint
    cluster_ca_certificate = base64decode(aws_eks_cluster.demo.certificate_authority[0].data)
    exec {
      api_version = "client.authentication.k8s.io/v1beta1"
      args        = ["eks", "get-token", "--cluster-name", aws_eks_cluster.demo.id]
      command     = "aws"
    }
  }
}

resource "helm_release" "aws-load-balancer-controller2" {
  name = "aws-load-balancer-controller"

  repository = "https://aws.github.io/eks-charts"
  chart      = "aws-load-balancer-controller"
  namespace  = "kube-system"
  version    = "1.5.4"

  set {
    name  = "clusterName"
    value = aws_eks_cluster.demo.id
  }

  set {
    name  = "image.tag"
    value = "v2.5.3"
  }

  set {
    name  = "serviceAccount.name"
    value = "aws-load-balancer-controller"
  }

  set {
    name  = "serviceAccount.annotations.eks\\.amazonaws\\.com/role-arn"
    value = aws_iam_role.aws_load_balancer_controller.arn
  }

  depends_on = [
    aws_eks_node_group.private-nodes,
    aws_iam_role_policy_attachment.aws_load_balancer_controller_attach
  ]
}
```
