# Account Recycling Solution — Implementation Tasks

Implementation tasks derived from the [README.md — What You Need for Your Own Account Recycling Solution](./README.md#what-you-need-for-your-own-account-recycling-solution).

---

## Phase 1: AWS Organizations & OU Structure

- [ ] **1.1** Set up AWS Organizations (or confirm existing org).
- [ ] **1.2** Create Organizational Units: `Active`, `Cleanup`, `Available`, `Quarantine`.
- [ ] **1.3** Author Service Control Policies (SCPs) for the `Cleanup` OU to restrict actions during cleanup.
- [ ] **1.4** Author SCPs for sandbox OUs to prevent users from creating hard-to-delete resources.
- [ ] **1.5** Attach SCPs to the appropriate OUs.

## Phase 2: IAM Role Chain

- [ ] **2.1** Create an **Intermediate IAM Role** in the hub account with a trust policy allowing CodeBuild to assume it.
- [ ] **2.2** Create a **Cleanup IAM Role** in each sandbox account with broad list/delete permissions.
- [ ] **2.3** Configure the Cleanup Role's trust policy to allow the Intermediate Role to assume it.
- [ ] **2.4** Add the Cleanup Role to the aws-nuke config filters so it is never deleted.

## Phase 3: Container Image

- [ ] **3.1** Write a Dockerfile (Amazon Linux 2023 minimal base) installing: aws-nuke (pinned version + checksum), AWS CLI, jq, sed.
- [ ] **3.2** Create an ECR repository (public or private).
- [ ] **3.3** Build and push the Docker image to ECR.
- [ ] **3.4** Set up a CI pipeline to rebuild the image on aws-nuke version updates.

## Phase 4: aws-nuke Configuration

- [ ] **4.1** Write the `nuke-config.yaml` template with placeholders (`%HUB_ACCOUNT_ID%`, `%CLEANUP_ACCOUNT_ID%`, `%CLEANUP_ROLE_NAME%`).
- [ ] **4.2** Define the account blocklist (production accounts).
- [ ] **4.3** Define resource filters to protect: cleanup role, org-managed resources, Control Tower resources, SSO resources, baseline infrastructure.
- [ ] **4.4** Configure `settings` block to disable deletion protection on relevant resource types.
- [ ] **4.5** Configure `resource-types.excludes` for optimization (e.g., S3Object, ConfigService resources).
- [ ] **4.6** Store the config in **AWS AppConfig** (or S3) as an externally managed configuration profile.
- [ ] **4.7** Store the managed regions list in **SSM Parameter Store**.

## Phase 5: CodeBuild Project

- [ ] **5.1** Create the CodeBuild project using the ECR Docker image.
- [ ] **5.2** Write the `buildspec.yaml` that:
  - Fetches nuke config from AppConfig.
  - Injects runtime values via `sed`.
  - Dynamically populates regions from SSM Parameter.
  - Assumes the Intermediate Role, then the Cleanup Role (role chaining).
  - Runs `aws-nuke nuke` with flags: `--no-dry-run --no-alias-check --force --profile cleanup --log-format=json`.
- [ ] **5.3** Set CodeBuild timeout to 60 minutes.
- [ ] **5.4** Configure CodeBuild environment variables (HUB_ACCOUNT_ID, INTERMEDIATE_ROLE_ARN, CLEANUP_ROLE_NAME, APPCONFIG IDs, SSM param ARN).
- [ ] **5.5** Configure CloudWatch Logs for CodeBuild output.

## Phase 6: Step Functions Orchestration

- [ ] **6.1** Create the **Initialize Cleanup Lambda** that:
  - Checks if cleanup is already in progress (concurrency guard).
  - Fetches global config (retry counts, wait durations).
  - Performs pre-cleanup actions.
- [ ] **6.2** Build the **Account Cleaner Step Function** state machine with:
  - Initialize counters (`succeeded=0`, `failed=0`).
  - Invoke Initialize Cleanup Lambda.
  - "Cleanup Already In Progress?" choice state.
  - Start CodeBuild (sync invocation).
  - On success: increment success counter → check if enough passes → wait → re-run or emit success.
  - On failure: increment failure counter, reset success counter → check if too many failures → wait → retry or emit failure.
- [ ] **6.3** Configure Step Function timeout to 12 hours.
- [ ] **6.4** Make retry parameters configurable: `numberOfSuccessfulAttemptsToFinishCleanup` (default: 3), `numberOfFailedAttemptsToCancelCleanup`, `waitBeforeRerunSuccessfulAttemptSeconds`, `waitBeforeRetryFailedAttemptSeconds`.

## Phase 7: EventBridge Integration

- [ ] **7.1** Define the `CleanAccountRequest` event schema.
- [ ] **7.2** Create an EventBridge rule to trigger the Step Function on `CleanAccountRequest`.
- [ ] **7.3** Define `AccountCleanupSuccessful` and `AccountCleanupFailure` event schemas.
- [ ] **7.4** Create EventBridge rules to route success/failure events to downstream consumers (OU movement, notifications).

## Phase 8: Account Lifecycle Management

- [ ] **8.1** Create a DynamoDB table to track account states: `Active`, `Cleanup`, `Available`, `Quarantine`.
- [ ] **8.2** Write Lambda functions to move accounts between OUs based on cleanup results.
- [ ] **8.3** Implement quarantine handling workflow for admin investigation and remediation.
- [ ] **8.4** Implement drift detection: periodic checks that DB state matches actual OU placement.

## Phase 9: Operational Concerns

- [ ] **9.1** Set up notifications (SNS → Email/Slack) for cleanup success, failure, and quarantine events.
- [ ] **9.2** Configure CloudWatch Logs retention and log group structure for audit/compliance.
- [ ] **9.3** Set up cost monitoring (AWS Budgets or Cost Anomaly Detection) to trigger cleanup at budget thresholds.
- [ ] **9.4** Verify concurrency control: the Initialize Cleanup Lambda must prevent duplicate runs on the same account.

## Phase 10: Testing & Validation

- [ ] **10.1** Dry-run aws-nuke against a test sandbox account and review the resource scan output.
- [ ] **10.2** Run a full cleanup cycle end-to-end on a test account.
- [ ] **10.3** Validate that all filtered resources survive cleanup.
- [ ] **10.4** Test failure scenarios: CodeBuild timeout, role assumption failure, max failures exceeded → quarantine.
- [ ] **10.5** Test concurrency guard: trigger two cleanups on the same account simultaneously.
- [ ] **10.6** Validate OU movement and DB state consistency after cleanup.
