# aws-nuke Configuration in Innovation Sandbox on AWS

**Config file:** [nuke-config.yaml](./nuke-config.yaml)

**Source:** [`source/infrastructure/lib/components/config/nuke-config.yaml`](https://github.com/aws-solutions/innovation-sandbox-on-aws/blob/main/source/infrastructure/lib/components/config/nuke-config.yaml)

## How It's Used

The YAML file is stored in **AWS AppConfig** as a configuration profile. At runtime, the CodeBuild buildspec fetches it via `appconfigdata get-latest-configuration`, then `sed` replaces three placeholders and dynamically injects regions from SSM Parameter Store:

| Placeholder | Replaced With |
|---|---|
| `%HUB_ACCOUNT_ID%` | The hub/management account ID |
| `%CLEANUP_ACCOUNT_ID%` | The target sandbox account ID |
| `%CLEANUP_ROLE_NAME%` | The IAM role name used for cleanup in the sandbox account |

Regions are read from an SSM Parameter (`isbManagedRegions`) and injected into the `regions:` block dynamically.

## Key Sections

### 1. `settings` — Disable Deletion Protection

Disables deletion/termination protection on 13 resource types before nuking them:

- **CloudFormationStack** — Disables deletion protection, creates a role to delete the stack.
- **CognitoUserPool** — Disables deletion protection.
- **DSQLCluster** — Disables deletion protection.
- **DynamoDBTable** — Disables deletion protection.
- **EC2Image** — Includes disabled/deprecated AMIs, disables deregistration protection.
- **EC2Instance** — Disables stop and deletion protection.
- **ELBv2** — Disables deletion protection.
- **LightsailInstance** — Force-deletes add-ons.
- **NeptuneCluster** — Disables deletion protection.
- **NeptuneInstance** — Disables cluster and instance deletion protection.
- **NeptuneGraph** — Disables deletion protection.
- **QLDBLedger** — Disables deletion protection.
- **QuickSightSubscription** — Disables termination protection.
- **RDSInstance** — Disables deletion protection.
- **S3Bucket** — Bypasses governance retention and removes object legal holds.

### 2. `resource-types.excludes` — Skipped Resources

Three resource types are excluded from nuking:

| Resource | Reason |
|---|---|
| `S3Object` | Optimization — lets the `S3Bucket` handler delete all objects in bulk instead of one-by-one |
| `ConfigServiceConfigurationRecorder` | Org-managed AWS Config resource |
| `ConfigServiceDeliveryChannel` | Org-managed AWS Config resource |

### 3. `blocklist` — Account Protection

The hub account ID is blocklisted to prevent aws-nuke from ever running against it.

### 4. `accounts.filters` — Protected Resources

Filters protect infrastructure that must survive cleanup:

| Filter Target | Pattern(s) | Purpose |
|---|---|---|
| **ISB StackSets** | `StackSet-Isb-*` | Protects sandbox account stack set instances |
| **Cleanup IAM Role** | `%CLEANUP_ROLE_NAME%` | Prevents aws-nuke from deleting the role it's using |
| **Organizations Role** | `OrganizationAccountAccessRole` | Preserves org management access |
| **StackSets Execution Roles** | `stacksets-exec-*` | Preserves CloudFormation StackSets execution roles |
| **IAM Identity Center (SSO)** | `AWSReservedSSO_*`, `AWSSSO` | Preserves SSO roles and SAML providers |
| **AWS Control Tower** | `aws-controltower-*`, `AWSControlTower` | Preserves Control Tower trails, EventBridge rules, IAM roles, Lambda functions, log groups, SNS topics/subscriptions |
| **OS Packages** | `pkg-*`, `G*` | Preserves base AMI packages |
