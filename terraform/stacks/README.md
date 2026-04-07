# Stacks

Atmos stack configurations for each deployment environment.

- `dev.yaml` - Development environment
- `staging.yaml` - Staging environment
- `prod.yaml` - Production environment
- `preview.yaml` - Preview environments for pull requests
- `defaults/` - Shared component configuration imported by all stacks
  - `app.yaml` - ECS app config (local component)
  - `s3-bucket.yaml` - S3 bucket config (source-provisioned from remote GitHub module)
- `deps/` - Infrastructure dependencies (VPC, ECS cluster, EFS)
