# Components

Local Terraform/OpenTofu modules checked into the repository.

- `ecs-task/` - ECS task definition and service component

> **Note:** Not all components are local. The `s3-bucket` component is [source-provisioned](https://atmos.tools/core-concepts/components/source-provisioning) from a remote GitHub module (`github.com/cloudposse/terraform-aws-s3-bucket`) and has no local directory here. Its configuration lives in `terraform/stacks/defaults/s3-bucket.yaml`.
