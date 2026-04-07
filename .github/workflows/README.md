# Workflows

GitHub Actions CI/CD pipelines. All deploy workflows use `atmos describe affected --format=matrix` to dynamically detect which components changed and dispatch parallel deploy jobs.

| Workflow | Trigger | Action |
|----------|---------|--------|
| `main-branch.yaml` | Push to `main` | Build image → Detect affected → Deploy affected components to dev → Create draft release |
| `release.yaml` | Published release | Promote image → Detect affected → Deploy to staging → Deploy to prod |
| `feature-branch.yml` | PR with `deploy` label | Build image → Detect affected → Deploy affected components to preview |
| `preview-cleanup.yml` | PR closed | Destroy preview environment |
| `validate.yml` | Pull request | Run validation checks |
| `labeler.yaml` | Pull request | Auto-label based on changed files |

## How Affected Detection Works

Each deploy workflow includes an `affected` job that runs `atmos describe affected --format=matrix`. This compares the current commit against the base to identify which component+stack pairs have changed, then outputs a JSON matrix that GitHub Actions uses to fan out parallel deploy jobs.

```mermaid
graph LR
    A[Git Diff] --> B[atmos describe affected]
    B --> C[JSON Matrix]
    C --> D[Deploy app -s dev]
    C --> E[Deploy s3-bucket -s dev]
    C --> F[Deploy ... -s ...]
```

## Main Branch Workflow

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant GH as GitHub
    participant GA as GitHub Actions
    participant ECR as AWS ECR
    participant Atmos as Atmos CLI
    participant TF as OpenTofu
    participant AWS as AWS

    Dev->>GH: Push to main
    GH->>GA: Trigger main-branch workflow
    GA->>ECR: Build & push Docker image
    ECR-->>GA: Image pushed (sha-xxx)
    GA->>Atmos: atmos describe affected --format=matrix
    Atmos-->>GA: Matrix of affected components
    par Deploy each affected component
        GA->>Atmos: atmos terraform deploy app -s dev
        Atmos->>TF: tofu apply
        TF->>AWS: Update ECS service
    and
        GA->>Atmos: atmos terraform deploy s3-bucket -s dev
        Atmos->>TF: tofu apply
        TF->>AWS: Update S3 bucket
    end
    AWS-->>GA: Deployments complete
    GA->>GH: Create draft release
```

## Release Workflow

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant GH as GitHub
    participant GA as GitHub Actions
    participant ECR as AWS ECR
    participant Atmos as Atmos CLI
    participant TF as OpenTofu
    participant AWS as AWS

    Dev->>GH: Publish release (v1.0.0)
    GH->>GA: Trigger release workflow
    GA->>ECR: Promote image tag (sha-xxx → v1.0.0)
    GA->>Atmos: atmos describe affected --format=matrix
    Atmos-->>GA: Staging matrix
    par Deploy affected to staging
        GA->>Atmos: atmos terraform deploy app -s staging
        Atmos->>TF: tofu apply
        TF->>AWS: Update staging services
    and
        GA->>Atmos: atmos terraform deploy s3-bucket -s staging
        Atmos->>TF: tofu apply
        TF->>AWS: Update staging S3
    end
    AWS-->>GA: Staging deployed
    GA->>Atmos: atmos describe affected --format=matrix
    Atmos-->>GA: Production matrix
    par Deploy affected to production
        GA->>Atmos: atmos terraform deploy app -s prod
        Atmos->>TF: tofu apply
        TF->>AWS: Update prod services
    and
        GA->>Atmos: atmos terraform deploy s3-bucket -s prod
        Atmos->>TF: tofu apply
        TF->>AWS: Update prod S3
    end
    AWS-->>GA: Production deployed
```

## Feature Branch Workflow (Preview Environments)

```mermaid
sequenceDiagram
    participant Dev as Developer
    participant GH as GitHub
    participant GA as GitHub Actions
    participant ECR as AWS ECR
    participant Atmos as Atmos CLI
    participant TF as OpenTofu
    participant AWS as AWS

    Dev->>GH: Open PR with 'deploy' label
    GH->>GA: Trigger feature-branch workflow
    GA->>ECR: Build & push Docker image
    ECR-->>GA: Image pushed
    GA->>Atmos: atmos describe affected --format=matrix
    Atmos-->>GA: Matrix of affected components
    par Deploy affected to preview
        GA->>Atmos: atmos terraform deploy app -s preview
        Atmos->>TF: tofu apply
        TF->>AWS: Create preview services
    and
        GA->>Atmos: atmos terraform deploy s3-bucket -s preview
        Atmos->>TF: tofu apply
        TF->>AWS: Create preview S3
    end
    AWS-->>GA: Preview URLs
    GA->>GH: Post preview URL to PR
    Note over Dev,GH: PR closed
    GH->>GA: Trigger preview-cleanup workflow
    GA->>Atmos: atmos terraform destroy app -s preview
    Atmos->>TF: tofu destroy
    TF->>AWS: Delete preview resources
```

## Environment Promotion Flow

```mermaid
graph LR
    A[Push to main] --> B[Build Image]
    B --> C[Detect Affected]
    C --> D[Deploy Affected to Dev]
    D --> E[Draft Release]
    E --> F{Publish Release}
    F --> G[Promote Image Tag]
    G --> H[Detect Affected]
    H --> I[Deploy Affected to Staging]
    I --> J[Detect Affected]
    J --> K[Deploy Affected to Production]

    L[PR with 'deploy' label] --> M[Build Image]
    M --> N[Detect Affected]
    N --> O[Deploy Affected to Preview]
    O --> P{PR Merged/Closed}
    P --> Q[Cleanup Preview]
```
