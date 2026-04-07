# CLAUDE.md

Example deploying a self-contained application and its S3 backing service using Atmos and OpenTofu.
The `app` component is a local ECS task module; the `s3-bucket` component is source-provisioned from a remote GitHub module (JIT fetched, no local code).

## Quick Reference

```bash
# Local development
atmos up                                    # Start app locally with Podman Compose
atmos down                                  # Stop local app

# Deploy with Atmos
atmos terraform plan app -s dev             # Plan app changes for dev
atmos terraform plan s3-bucket -s dev       # Plan S3 bucket changes for dev
atmos terraform deploy app -s dev           # Deploy app to dev
atmos terraform deploy s3-bucket -s dev     # Deploy S3 bucket to dev
atmos terraform deploy app -s staging       # Deploy app to staging
atmos terraform deploy app -s prod          # Deploy app to production

# Get deployment URL
atmos terraform output app -s dev --skip-init -- -raw url

# Detect affected components (used by CI)
atmos describe affected --format=matrix
```

## Project Structure

- `app/` - Go web application (see `app/README.md`)
- `terraform/components/ecs-task/` - Local ECS task Terraform component
- `terraform/stacks/defaults/app.yaml` - ECS app config (local component)
- `terraform/stacks/defaults/s3-bucket.yaml` - S3 bucket config (source-provisioned, no local component)
- `terraform/stacks/` - Environment configurations
- `.atmos.d/commands.yaml` - Custom Atmos commands
- `.github/workflows/` - CI/CD pipelines using `atmos describe affected --format=matrix`
