# Service Terraform

This terraform must be run to provision the ECS Fargate service and task definition used to run the Github Audit Tool.

The IaC is separated from the storage terraform so that the file store can persist if the service is destroyed.  This gives greater flexibility, e.g allowing the service to be destroyed to save costs without losing the data that has been output by the tool.

## Prerequisites

The service terraform is bootstrapped with a separate terraform state key so that both S3 and Service terraform state files are separated.

## Apply the Terraform

### Update Existing Service

If you are just upgrading the application logic then only the service terraform needs to be run.

When upgrading:

- Ensure the container_ver variable is set to the appropriate version in the relevant env tfvar file
- Ensure the force_deployment flag is set to true in the relevant env tfvar file
- Ensure the terraform is configured to point at the appropriate backend
- Ensure the terraform plan and apply select the appropriate environment variable file

```bash
terraform init -backend-config=env/prod/backend-prod.tfbackend -reconfigure

terraform validate

terraform plan -var-file=env/prod/prod.tfvars

terraform apply -var-file=env/prod/prod.tfvars
```

### Provision Service from Scratch

Apply the S3 bucket storage terraform first

```bash
cd terraform/storage 

terraform init -backend-config=env/prod/backend-prod.tfbackend -reconfigure

terraform validate

terraform plan -var-file=env/prod/prod.tfvars

terraform apply -var-file=env/prod/prod.tfvars
```

Apply the Authentication terraform second

```bash
cd terraform/authentication 

terraform init -backend-config=env/prod/backend-prod.tfbackend -reconfigure

terraform refresh -var-file=env/prod/prod.tfvars

terraform validate

terraform plan -var-file=env/prod/prod.tfvars

terraform apply -var-file=env/prod/prod.tfvars
```

Apply the terraform for this service

```bash
cd terraform/service 

terraform init -backend-config=env/prod/backend-prod.tfbackend -reconfigure

terraform validate

terraform plan -var-file=env/prod/prod.tfvars

terraform apply -var-file=env/prod/prod.tfvars
```

## Resources

The IaC creates the following resources:

- Target Group
- Application Load Balancer service rule to authenticate via Cognito
- Application Load Balancer service rule to bypass Cognito authentication for certain URLs
- Route 53 Record to map service URL through to Application Load Balancer
- Security Group to apply to Service restricting traffic on designated ports
- ECS Task Definition
- ECS Service  
