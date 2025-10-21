# Delete Workflow Runs v2.1.0

A GitHub Action to delete workflow runs in a repository. This Action uses JavaScript and interacts with the GitHub API to manage workflow runs efficiently.

## Features

- Deletes workflow runs based on retention period and minimum runs to keep.
- Supports filtering by workflow name, filename, state, or run conclusion.
- Includes a dry-run mode to simulate deletions without making changes.
- Skips runs linked to active branches or pull requests (optional).
- Optimized to avoid uploading `node_modules` by bundling code with `@vercel/ncc`.

## Inputs

### 1. `token`

- **Required**: Yes
- **Default**: `${{ github.token }}`
- The GitHub token for authentication. Use `github.token` for the current repository (requires `actions: write` and `contents: read` permissions) or a Personal Access Token (PAT) with `repo` scope for other repositories.

### 2. `repository`

- **Required**: Yes
- **Default**: `${{ github.repository }}`
- The repository name in `{owner}/{repo}` format.

### 3. `retain_days`

- **Required**: Yes
- **Default**: `30`
- Number of days to retain workflow runs before deletion.

### 4. `keep_minimum_runs`

- **Required**: Yes
- **Default**: `6`
- Minimum number of runs to keep per workflow.

### 5. `delete_workflow_pattern`

- **Required**: No
- Target workflows by name or filename. Omit to target all workflows.

### 6. `delete_workflow_by_state_pattern`

- **Required**: No
- Filter workflows by state (comma-separated): `active`, `deleted`, `disabled_fork`, `disabled_inactivity`, `disabled_manually`.

### 7. `delete_run_by_conclusion_pattern`

- **Required**: No
- Filter runs by conclusion (comma-separated): `action_required`, `cancelled`, `failure`, `skipped`, `success`.

### 8. `dry_run`

- **Required**: No
- **Default**: `false`
- Simulate deletions and log actions without performing them.

### 9. `check_branch_existence`

- **Required**: No
- **Default**: `false`
- Skip deletion if the run is linked to an existing branch (excludes `main`).

### 10. `check_pullrequest_exist`

- **Required**: No
- **Default**: `false`
- Skip deletion if the run is linked to a pull request.

## Setup

1. Ensure the repository has a `package.json` with dependencies and a build script using `@vercel/ncc`.
2. Run `npm install` and `npm run build` to generate `dist/index.js`.
3. Commit the `dist/` folder, but exclude `node_modules/` using `.gitignore`.
4. Use the Action in your workflow as shown below.

## Examples

### Scheduled Workflow

Run monthly to delete old workflow runs:

```yaml
name: Delete old workflow runs
on:
  schedule:
    - cron: "0 0 1 * *" # Monthly at 00:00 on the 1st
jobs:
  delete-runs:
    runs-on: ubuntu-latest
    permissions:
      actions: write
      contents: read
    steps:
      - name: Delete workflow runs
        uses: Mattraks/delete-workflow-runs@v2
        with:
          token: ${{ github.token }}
          repository: ${{ github.repository }}
          retain_days: 30
          keep_minimum_runs: 6
```

### Manual Workflow

Trigger manually with customizable inputs:

```yaml
name: Delete old workflow runs
on:
  workflow_dispatch:
    inputs:
      days:
        description: "Days to retain runs"
        required: true
        default: "30"
      minimum_runs:
        description: "Minimum runs to keep"
        required: true
        default: "6"
      delete_workflow_pattern:
        description: "Workflow name or filename (omit for all)"
        required: false
      delete_workflow_by_state_pattern:
        description: "Workflow state: active, deleted, disabled_fork, disabled_inactivity, disabled_manually"
        required: false
        default: "ALL"
        type: choice
        options:
          - "ALL"
          - active
          - deleted
          - disabled_inactivity
          - disabled_manually
      delete_run_by_conclusion_pattern:
        description: "Run conclusion: action_required, cancelled, failure, skipped, success"
        required: false
        default: "ALL"
        type: choice
        options:
          - "ALL"
          - "Unsuccessful: action_required,cancelled,failure,skipped"
          - action_required
          - cancelled
          - failure
          - skipped
          - success
      dry_run:
        description: "Simulate deletions"
        required: false
        default: "false"
        type: choice
        options:
          - "false"
          - "true"
jobs:
  delete-runs:
    runs-on: ubuntu-latest
    permissions:
      actions: write
      contents: read
    steps:
      - name: Delete workflow runs
        uses: Mattraks/delete-workflow-runs@v2
        with:
          token: ${{ github.token }}
          repository: ${{ github.repository }}
          retain_days: ${{ github.event.inputs.days }}
          keep_minimum_runs: ${{ github.event.inputs.minimum_runs }}
          delete_workflow_pattern: ${{ github.event.inputs.delete_workflow_pattern }}
          delete_workflow_by_state_pattern: ${{ github.event.inputs.delete_workflow_by_state_pattern }}
          delete_run_by_conclusion_pattern: >-
            ${{
              startsWith(github.event.inputs.delete_run_by_conclusion_pattern, 'Unsuccessful:') &&
              'action_required,cancelled,failure,skipped' ||
              github.event.inputs.delete_run_by_conclusion_pattern
            }}
          dry_run: ${{ github.event.inputs.dry_run }}
```

### GitHub Enterprise

For GitHub Enterprise, specify the API base URL:

```yaml
jobs:
  delete-runs:
    runs-on: ubuntu-latest
    permissions:
      actions: write
      contents: read
    steps:
      - name: Delete old workflow runs
        uses: Mattraks/delete-workflow-runs@v2
        with:
          token: ${{ secrets.PAT_TOKEN }}
          baseUrl: https://github.mycompany.com/api/v3
          repository: mycompany/myrepo
          retain_days: 30
          keep_minimum_runs: 6
```

## Development

To build the Action:

1. Install dependencies: `npm install`
2. Build the Action: `npm run build`
3. Commit the `dist/` folder to the repository.

The `node_modules` folder is excluded via `.gitignore` to reduce repository size.

## License

This project is licensed under the [MIT License](LICENSE).
