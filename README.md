# Project 020 – Secure Terraform CI with GitHub Secrets

## Overview

This project extends the Terraform CI pipeline by securely injecting Terraform variables into GitHub Actions using GitHub Secrets.

The objective was to eliminate interactive Terraform input, avoid committing sensitive information to source control, and build a fully automated, secure Infrastructure as Code (IaC) validation pipeline.

The pipeline authenticates to AWS using GitHub OpenID Connect (OIDC), retrieves temporary AWS credentials through AWS Security Token Service (STS), injects required Terraform variables from GitHub Secrets, and validates the infrastructure without exposing sensitive information.

---

## Project Objectives

* Eliminate interactive Terraform input during CI execution.
* Securely manage Terraform variables using GitHub Secrets.
* Prevent sensitive credentials from being committed to Git.
* Continue using AWS OIDC authentication.
* Execute Terraform Plan automatically on every push.

---

## Technologies Used

* Terraform
* GitHub Actions
* GitHub Secrets
* AWS IAM
* AWS STS
* AWS OpenID Connect (OIDC)
* Amazon S3 Remote Backend
* Git

---

## CI/CD Workflow

```text
Git Push
    │
    ▼
GitHub Actions
    │
    ▼
OIDC Authentication
    │
    ▼
AWS STS Temporary Credentials
    │
    ▼
Terraform Init
    │
    ▼
Terraform Format Check
    │
    ▼
Terraform Validate
    │
    ▼
GitHub Secrets
    │
    ▼
Terraform Plan
```

---

## GitHub Secrets Used

| Secret      | Purpose           |
| ----------- | ----------------- |
| KEY_NAME    | EC2 Key Pair      |
| DB_USERNAME | Database Username |
| DB_PASSWORD | Database Password |

The workflow injects these values using Terraform's `TF_VAR_` environment variable convention.

Example:

```yaml
env:
  TF_VAR_key_name: ${{ secrets.KEY_NAME }}
  TF_VAR_db_username: ${{ secrets.DB_USERNAME }}
  TF_VAR_db_password: ${{ secrets.DB_PASSWORD }}
```

Terraform automatically maps these values to the corresponding input variables.

---

## Security Improvements

Before this project:

* Terraform requested interactive input.
* CI pipelines stalled waiting for variable input.
* Sensitive variables risked being hardcoded.

After this project:

* No interactive prompts.
* No secrets committed to Git.
* Sensitive values stored securely in GitHub Secrets.
* Terraform receives variables automatically during pipeline execution.

---

## Problems Encountered

### 1. Interactive Terraform Variables

#### Issue

Terraform Plan paused waiting for:

* key_name
* db_username
* db_password

#### Cause

GitHub Actions cannot provide interactive terminal input.

#### Solution

Configured GitHub Secrets and passed them into Terraform using the `TF_VAR_` environment variable convention.

---

### 2. Missing Required Variable

#### Issue

Terraform reported:

```text
No value for required variable:
instance_type
```

#### Cause

The variable had no default value and was not sensitive.

#### Solution

Added a Terraform default:

```hcl
variable "instance_type" {
  type    = string
  default = "t3.micro"
}
```

This removed unnecessary configuration from the CI pipeline.

---

### 3. Terraform Format Check Failure

#### Issue

GitHub Actions failed during:

```text
terraform fmt -check
```

#### Cause

Terraform code formatting did not match Terraform's standard formatting rules.

#### Solution

Executed:

```bash
terraform fmt
```

Committed the formatting changes and reran the workflow successfully.

---

## Final Pipeline Result

Successful pipeline execution:

* Checkout Repository
* Setup Terraform
* Configure AWS Credentials
* Terraform Init
* Terraform Format Check
* Terraform Validate
* Terraform Plan

Terraform completed with:

```text
No changes.
Your infrastructure matches the configuration.
```

This confirms that:

* Terraform configuration
* Remote Terraform State
* AWS Infrastructure

are fully synchronized.

---

## Lessons Learned

This project reinforced several important DevOps principles:

* Separate secrets from configuration.
* CI/CD pipelines must never require interactive input.
* GitHub Secrets provide secure variable injection.
* Terraform automatically maps `TF_VAR_` variables.
* Infrastructure should always be validated before deployment.
* Automated formatting improves code consistency across teams.

---

## Skills Demonstrated

* Infrastructure as Code (Terraform)
* Secure Secret Management
* GitHub Actions
* GitHub Secrets
* AWS OIDC Authentication
* AWS STS Temporary Credentials
* Terraform Variable Management
* Terraform CI/CD
* Infrastructure Validation
* DevOps Troubleshooting

---

## Future Improvements

* Store database credentials in AWS Secrets Manager instead of GitHub Secrets.
* Add Terraform Apply with manual approval.
* Add security scanning using Checkov or TFLint.
* Add Pull Request validation.
* Implement least-privilege IAM policies.

---

## Outcome

Successfully built a production-style Terraform CI pipeline that securely injects Terraform variables using GitHub Secrets while authenticating to AWS through GitHub OIDC and AWS STS. The pipeline validates infrastructure automatically without exposing sensitive credentials or requiring manual input.
