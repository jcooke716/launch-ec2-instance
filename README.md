# launch-ec2-instance

## Project overview

A Terraform and AWS CodeBuild lifecycle lab for a single Apache web server on EC2. The project combines instance provisioning, startup automation, remote state, and CI job scripting.

## Infrastructure and bootstrap

[ec2.tf](ec2.tf) defines:

- An AWS provider in `us-east-1` using the `terraform-user` profile.
- An S3 backend for Terraform state.
- The region's default VPC and a default subnet.
- A security group allowing HTTP and SSH.
- An Amazon Linux 2 AMI lookup and a `t2.micro` EC2 instance.
- An output that constructs the instance's public HTTP URL.

[install_techmax.sh](install_techmax.sh) is supplied as EC2 user data. It installs Apache, downloads the [Techmax website template](https://github.com/azeezsalu/techmax), copies its assets into the document root, and enables the web service.

## CI configuration

| File | Responsibility |
| --- | --- |
| [cicd/buildspec.yml](cicd/buildspec.yml) | Defines CodeBuild install, preparation, and build commands. |
| [cicd/install-terraform.sh](cicd/install-terraform.sh) | Installs Terraform through the HashiCorp Amazon Linux RPM repository. |
| [cicd/configure-named-profile.sh](cicd/configure-named-profile.sh) | Configures an AWS profile from CI environment variables. |
| [cicd/apply-terraform.sh](cicd/apply-terraform.sh) | Initializes Terraform and performs the selected lifecycle action. |

**The checked-in CI lifecycle action is currently `terraform destroy -auto-approve`. The apply command is commented out.** Despite its filename, `apply-terraform.sh` is presently configured for teardown and must be reviewed before any build job is run.

## Configuration requirements

The current configuration expects an existing S3 state bucket, an EC2 key pair, and the named AWS profile. The CI profile script reads `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`, and `PROFILE_NAME`; the selected profile must match the Terraform provider and backend.

For an initial configuration review:

```bash
terraform init -backend=false
terraform validate
terraform fmt -check
```

Before deploying, adapt the backend, key pair, and network rules to the target account. Confirm that the bootstrap environment supplies `wget` and `unzip`; the script invokes both without installing them explicitly.

## Portfolio focus

The boundary between infrastructure provisioning and operating-system bootstrap, plus the configuration of an infrastructure CI job. The website is template content; the delivery and instance lifecycle configuration are the focus of this repository.
