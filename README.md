# CCoE Reference Architectures

This repository serves as a collection of experiments from the CCoE (Cloud Center of Excellence) Team.

## AWS

| Focus | Project | Description | Use Case |
| --- | --- | --- | --- |
| Security - AWS Credentials| [Limit AWS Credentials](examples/aws/limit-aws-credentials) | Limits AWS credentials to an AWS VPC or IP outside of AWS. | Sometimes, you have a project where you need static AWS credentials. The problem is that when you generate them, they can be used anywhere and become risks. This example helps lower that risk by restricting where they can be used.  |
