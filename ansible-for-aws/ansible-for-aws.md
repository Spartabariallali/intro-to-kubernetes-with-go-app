## Building and AWS EKS cluster with cloudformation and ansible
- Ansible includes a number of aws-specific modules, including `aws_eks_cluster` module, which can control AWS resources directly
- sometimes its not efficient to use ansible modules to directly manipulate AWS infrastructure components.
- AWS offers their own 'Infrastructure as Code' solution, `CloudFormation` and its a reliable way to define AWS infrastructure via YAML templates.
- Ansible offers a CloudFormation module
