# Terraform

Infrastructure as Code using [Atmos](https://atmos.tools) and [OpenTofu](https://opentofu.org).

- `components/` - Local Terraform/OpenTofu modules checked into the repository (e.g., `ecs-task`)
- `stacks/` - Environment-specific configurations (dev, staging, prod, preview)

This project uses two component provisioning models:

1. **Local components** under `components/` - traditional Terraform modules managed in the repo (e.g., `ecs-task` for the ECS application)
2. **Source-provisioned components** defined in stack YAML - fetched just-in-time from remote sources with no local code (e.g., `s3-bucket` from `github.com/cloudposse/terraform-aws-s3-bucket`)
