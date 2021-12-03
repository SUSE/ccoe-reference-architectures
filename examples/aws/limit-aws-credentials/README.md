# Limit AWS Credentials

Sometimes, you have a project where you need static AWS credentials. The problem is that when you generate them, they can be used anywhere and become risks. This example helps lower that risk by restricting where they can be used.

**Warning:** If you can avoid generating AWS credentials, please do so. Whenever you generate credentials, you pick up a security debt (rotating credentials, preventing leakage, etc...). It's preferred to [assume a role](https://docs.aws.amazon.com/STS/latest/APIReference/API_AssumeRole.html), use an [instance profile on ec2](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2_instance-profiles.html), or attach a [role to your service account](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) in kubernetes.

**Our Options:**
* Limit AWS credentials to a VPC through a VPC Endpoint.
* Limit AWS credentials to an IP address outside of AWS.

**Notes:**
* If you plan on restricting AWS credentials to a VPC, you should be aware that VPC endpoints aren't free. You can calculate their price [here](https://aws.amazon.com/privatelink/pricing/).

## The Experiment

In this example, we're restricting the usage of `ec2`. We do this by using a `deny` policy. If a request meets the requirements for a `deny`, it stops there without evaluating anything else. If the request passes, it can evaluate any `allow`s that you give it.

**The Policies:**

1. Deny all requests not coming through the VPC, and if it is, allow `ec2:Describe*`.
2. Deny all requests not coming through an external ip address, and if it is, allow `ec2:Describe*`.

(see [main.tf](./terraform/main.tf) for more information)

**The Setup:**

1. Your local machine as the external ip address. This should be able to use `aws ec2 describe-instances` with the credentials for the source ip, but not the credentials for the vpc.
2. An AWS instance inside a VPC with a VPC Endpoint. This should be able to use `aws ec2 describe-instances` with the credentials for the vpc, but not the credentials for the source ip.

### Run the Test

1. Create the file `terraform/terraform.tfvars`. You can see an example at `terraform/terraform.tfvars.example`.
   1. `vpc_id` is the ID of the VPC that you would like to test with.
   2. `subnet_id` is the ID of the subnet that you would like to test with.
   3. `local_ip` is your current public ip address.
2. Apply the terraform code:
   ```bash
   cd terraform
   terraform init
   terraform apply
   ```
3. Go to the AWS console and generate IAM credentials for the users `experiment-allow-vpce` and `experiment-allow-source-ip`.
4. On your local machine:
   1. Export the generated IAM credentials for the user `experiment-allow-source-ip` to your shell.
   2. Run `aws ec2 describe-instances --region=eu-central-1`. It should work.
   3. Export the generated IAM credentials for the user `experiment-allow-vpce` to your shell.
   4. Run `aws ec2 describe-instances --region=eu-central-1`. It shouldn't work.
5. SSH into the test server. Terraform should have given you its DNS name in its output.
   1. Export the generated IAM credentials for the user `experiment-allow-source-ip` to your shell.
   2. Run `aws ec2 describe-instances --region=eu-central-1`. It shouldn't work.
   3. Export the generated IAM credentials for the user `experiment-allow-vpce` to your shell.
   4. Run `aws ec2 describe-instances --region=eu-central-1`. It should work.

## Useful Links

* [Policy Evaluation Logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html#policy-eval-denyallow)
* [Policy Condition - VPC Endpoint](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourcevpce)
* [Policy Condition - Source IP](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourceip)
* [Example policy for restricting a s3 bucket to a vpc endpoint](https://docs.aws.amazon.com/AmazonS3/latest/userguide/example-bucket-policies-vpc-endpoint.html#example-bucket-policies-restrict-accesss-vpc-endpoint)
