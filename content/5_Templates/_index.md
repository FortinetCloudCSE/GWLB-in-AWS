---
title: "Templates"
menuTitle: "Templates"
weight: 50
---

It is best practice to use Infrastructure as Code (IaC) templates to deploy FortiGates & GWLB in AWS as there are quite a bit of components that make up the entire solution.  These can be used to deploy a new VPC.  You can also integrate with a new or existing Transit Gateway as well.

Reference the CloudFormation and Terraform templates in the Github repos below and reference the quick start guides for how to use these for a deployment.

{{% notice note %}}
You will need administrator privileges to run these templates as they are creating IAM roles, policies, and other resources that are required for the solution and automating deployment.
{{% /notice %}}

**fortigate-aws-gwlb-cloudformation**
  - [**Repo**](https://github.com/FortinetCloudCSE/fortigate-aws-gwlb-cloudformation)
  - [**Quick Start**](https://fortinetcloudcse.github.io/fortigate-aws-gwlb-cloudformation/)

**fortigate-aws-gwlb-terraform**
  - [**Repo**](https://github.com/FortinetCloudCSE/fortigate-aws-gwlb-terraform)
  - [**Quick Start**](https://fortinetcloudcse.github.io/fortigate-aws-gwlb-terraform/)