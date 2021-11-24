# Limit AWS Credentials

This is the results of an internal experiment to see how we could limit an AWS IAM user's credentials.

**What We Wanted:**
* Limit AWS credentials to a VPC through a VPC Endpoint.
* Limit AWS credentials to an IP address outside of AWS.

**Findings:**

* It's possible to limit AWS credentials to a VPC through a VPC endpoint. This is because we can filter our traffic through it and use `aws:SourceVpce` in our IAM conditional.
* It's possible to limit AWS credentials to a source ip.

**Notes:**
* VPC endpoints aren't free. You can calculate the price [here](https://aws.amazon.com/privatelink/pricing/).

## Useful Links

* https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html#policy-eval-denyallow
* https://docs.aws.amazon.com/AmazonS3/latest/userguide/example-bucket-policies-vpc-endpoint.html#example-bucket-policies-restrict-accesss-vpc-endpoint
* https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_condition-keys.html#condition-keys-sourcevpce

## Run the Test

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
