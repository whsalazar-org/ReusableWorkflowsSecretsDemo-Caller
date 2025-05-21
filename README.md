# GitHub Actions: Reusable Workflows & Secrets Demo

This repository, `whsalazar-org/ReusableWorkflowsSecretsDemo---Caller`, demonstrates two methods for passing secrets to reusable GitHub Actions workflows:

1.  **Explicitly Passing Secrets**: The calling workflow specifies exactly which secrets are passed to the reusable workflow and under what name the reusable workflow should expect them.
2.  **Inheriting Secrets**: The calling workflow uses the `secrets: inherit` keyword, making all of its accessible secrets available to the reusable workflow under their original names.

## Repository Structure

*   **`.github/workflows/basic.yml`**: This is the main "caller" workflow. It defines two jobs, each calling a different reusable workflow to showcase the two secret-passing methods.
*   **`.github/workflows/reusable-workflow-explicit.yml`**: A reusable workflow designed to receive a secret that is explicitly passed by the caller. It declares the secret it expects in its `on.workflow_call.secrets` section.
*   **`.github/workflows/reusable-workflow-inherit.yml`**: A reusable workflow that accesses secrets made available by the caller through `secrets: inherit`. It does not declare specific secrets in its `on.workflow_call.secrets` section for this purpose.

## How It Works

### 1. Caller Workflow (`basic.yml`)

This workflow is triggered on `push` or `pull_request` to the `main` branch, or manually via `workflow_dispatch`.

It contains two distinct jobs:

*   **`call-reusable-explicitly-passing-secrets`**:
    *   Calls `./.github/workflows/reusable-workflow-explicit.yml`.
    *   Uses the `secrets` block to explicitly pass a secret:
        ```yaml
        secrets:
          EXPECTED_SECRET_NAME: ${{ secrets.PASSED_SECRET_VALUE }}
        ```
    *   This means the secret named `PASSED_SECRET_VALUE` in *this* repository's secrets will be available as `EXPECTED_SECRET_NAME` within the `reusable-workflow-explicit.yml` workflow.

*   **`call-reusable-inheriting-secrets`**:
    *   Calls `./.github/workflows/reusable-workflow-inherit.yml`.
    *   Uses `secrets: inherit`:
        ```yaml
        secrets: inherit
        ```
    *   This makes any secrets available to the `basic.yml` workflow (like `A_SECRET_FROM_CALLER_REPO` or `ANOTHER_INHERITED_SECRET` defined in this repository's secrets) also available to `reusable-workflow-inherit.yml` under their original names.

### 2. Reusable Workflow - Explicit Secrets (`reusable-workflow-explicit.yml`)

*   Triggered by `workflow_call`.
*   Declares the secret it expects:
    ```yaml
    on:
      workflow_call:
        secrets:
          EXPECTED_SECRET_NAME:
            description: 'A secret that is explicitly passed by the caller workflow.'
            required: true
    ```
*   It then attempts to access `${{ secrets.EXPECTED_SECRET_NAME }}`.

### 3. Reusable Workflow - Inherit Secrets (`reusable-workflow-inherit.yml`)

*   Triggered by `workflow_call`.
*   It does **not** define a `secrets` block in its `on.workflow_call` trigger for the inherited secrets.
*   It can directly attempt to access secrets that were available to the caller, for example, `${{ secrets.A_SECRET_FROM_CALLER_REPO }}`.

## Setup and Usage

1.  **Fork/Clone this Repository**: Get a copy of this repository.
2.  **Define Repository Secrets**:
    Navigate to your repository's `Settings` > `Secrets and variables` > `Actions`. Create the following repository secrets:
    *   `PASSED_SECRET_VALUE`: (e.g., `explicit_secret_data_123`) - This will be passed explicitly.
    *   `A_SECRET_FROM_CALLER_REPO`: (e.g., `inherited_secret_alpha_456`) - This will be accessed via `secrets: inherit`.
    *   `ANOTHER_INHERITED_SECRET`: (e.g., `inherited_secret_beta_789`) - Another secret to demonstrate inheritance.
3.  **Trigger the Workflow**:
    *   Push a change to the `main` branch.
    *   Create a Pull Request targeting the `main` branch.
    *   Manually trigger the `ReusableWorkflow-Caller-Demo` workflow from the "Actions" tab in your repository.
4.  **Observe the Results**:
    Check the logs for the `ReusableWorkflow-Caller-Demo` workflow run. You will see output from both reusable workflows indicating whether they successfully accessed the secrets.

## Key Takeaways

*   **Explicit Passing (`secrets: ...`)**:
    *   **Pros**: More secure and clear, as the reusable workflow explicitly states what secrets it needs. The caller decides which of its secrets map to the reusable workflow's expectations. Prevents accidental exposure of unnecessary secrets.
    *   **Cons**: Can be more verbose if many secrets need to be passed.
*   **Inheriting Secrets (`secrets: inherit`)**:
    *   **Pros**: Convenient for passing many secrets without listing them individually. Good for trusted reusable workflows that might need access to a broad set of environment-specific secrets.
    *   **Cons**: Less explicit; the reusable workflow has potential access to all secrets available to the caller, which might be more than it strictly needs. Requires careful consideration of the trust level for the reusable workflow.

This demonstration should help you understand how to securely and effectively manage secrets when working with reusable workflows in GitHub Actions.
