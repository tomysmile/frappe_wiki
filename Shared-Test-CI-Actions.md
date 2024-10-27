## Using Shared GitHub Actions in Frappe

GitHub Actions provide a powerful way to automate workflows in your repositories. Frappe has implemented shared workflows to streamline common tasks across its projects. Here's how to use these shared actions effectively:

### Available Shared Actions

Frappe currently offers the following shared GitHub Actions:

1. **Type Checking**
2. **Server Testing**
3. **Migration Testing**
4. **UI Testing**

### Type Checking Workflow

The type checking workflow is designed to run MyPy on your codebase. To use this workflow:

1. Create a workflow file in your repository (e.g., `.github/workflows/type-check.yml`)
2. Add the following content:

```yaml
name: Type Check

on:
  pull_request:
  workflow_dispatch:

jobs:
  typecheck:
    name: Types
    uses: frappe/frappe/.github/workflows/type-check-base.yml@develop
```

This workflow will automatically run type checking on pull requests and can be manually triggered.

### Server Testing Workflow

The server testing workflow runs your project's tests against both MariaDB and PostgreSQL. To implement this workflow:

1. Create a workflow file (e.g., `.github/workflows/tests.yml`)
2. Add the following content:

```yaml
name: Tests

on:
  pull_request:
  workflow_dispatch:

jobs:
  test:
    name: Tests
    uses: frappe/frappe/.github/workflows/tests-base.yml@develop
    with:
      enable-postgres: true
      parallel-runs: 2
      disable-socketio: true
      disable-redis-socketio: true
```

### UI Testing Workflow

To implement the UI testing workflow:

1. Create a workflow file (e.g., `.github/workflows/ui-tests.yml`)
2. Add the following content:

```yaml
name: UI

on:
  pull_request:
  workflow_dispatch:

jobs:
  test:
    name: UI Tests
    uses: frappe/frappe/.github/workflows/ui-tests-base.yml@develop
    with:
      parallel-runs: 3
```

This workflow will run UI tests using Cypress against a MariaDB database.
Thank you for providing the updated information. I'll revise the Customization Options section to accurately reflect the available options for the shared GitHub Actions workflows.

### Customization Options

The shared workflows offer several customization options:

#### Tests Base Workflow

- **fake-success**: Set to `true` to simulate a successful run without actually executing tests (default: `false`)
- **python-version**: Specify the Python version to use (default: '3.12')
- **node-version**: Set the Node.js version for the workflow (default: 20)
- **parallel-runs**: Define the number of parallel test runs (default: 2)
- **enable-postgres**: Set to `true` to run tests against both MariaDB and PostgreSQL (default: `false`)
- **enable-coverage**: Enable coverage reporting (default: `false`)

#### Migration Base Workflow

- **fake-success**: Set to `true` to simulate a successful run without actually executing the patch (default: `false`)
- **python-version**: Specify the Python version to use (default: '3.10')
- **node-version**: Set the Node.js version for the workflow (default: 20)

#### UI Tests Base Workflow

- **fake-success**: Set to `true` to simulate a successful run without actually executing UI tests (default: `false`)
- **python-version**: Specify the Python version to use (default: '3.12')
- **node-version**: Set the Node.js version for the workflow (default: 20)
- **parallel-runs**: Define the number of parallel test runs (default: 2)
- **enable-coverage**: Enable coverage reporting for UI tests (default: `false`)

To use these options, include them in the `with` section of your workflow file. For example:

```yaml
jobs:
  test:
    name: Tests
    uses: frappe/frappe/.github/workflows/tests-base.yml@develop
    with:
      python-version: '3.12'
      node-version: 20
      parallel-runs: 4
      enable-postgres: true
      enable-coverage: true
```

By customizing these options, you can tailor the shared workflows to suit the specific needs of your Frappe project while maintaining consistency across different repositories.

### Complete Example

More details can be found inside `frappe/frappe/.github/workflows`.

The following example results in a nicely organized test hierarchy, grouped into UI and Server tests (including migration and type checks).

```yaml
name: Server

on:
  repository_dispatch:
    types: [frappe-framework-change]  # this event should be captured by all frappe/* repos
  pull_request:
  workflow_dispatch:
  schedule:
    # Run everday at midnight UTC / 5:30 IST
    - cron: "0 0 * * *"

concurrency:
  group: server-${{ github.event_name }}-${{ github.event.number }}
  cancel-in-progress: true

permissions:
  # Do not change this as GITHUB_TOKEN is being used by roulette
  contents: read

jobs:
  typecheck:
    name: Types
    uses: frappe/frappe/.github/workflows/type-check-base.yml@develop

  checkrun: # decides if success can be faked
    name: Plan Tests
    runs-on: ubuntu-latest
    needs: typecheck  # type checks run first and must succeed
    outputs:
      build: ${{ steps.check-build.outputs.build }}
    steps:
      - name: Clone
        uses: actions/checkout@v4
      - name: Check if unit tests should be run
        id: check-build
        run: |
          python "${GITHUB_WORKSPACE}/.github/helper/roulette.py"
        env:
          TYPE: "server"
          PR_NUMBER: ${{ github.event.number }}
          REPO_NAME: ${{ github.repository }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  test:
    name: Tests
    uses: frappe/frappe/.github/workflows/tests-base.yml@develop
    with:
      enable-postgres: true  # This will test against both MariaDB and PostgreSQL
      parallel-runs: 2
      enable-coverage: ${{ github.event_name != 'pull_request' }}
      fake-success: ${{ needs.checkrun.outputs.build != 'strawberry' }}
    needs: checkrun  # runs only after checkrun, but runs in parallel with migratek
    secrets: inherit  # inherit secrets from the calling workflow, e.g. secrets.SENTRY_DSN

  migrate:
    name: Migration
    needs: checkrun  # runs only after checkrun, but runs in parallel with migrate
    uses: frappe/frappe/.github/workflows/patch-base.yml@develop
    with:
      python-version: '3.10'
      node-version: 20
      fake-success: ${{ needs.checkrun.outputs.build != 'strawberry' }}

  coverage:  # uploads coverage data with the secret
    name: Coverage Wrap Up
    needs: [test, checkrun]
    runs-on: ubuntu-latest
    if: ${{ github.event_name != 'pull_request' }}
    steps:
      - name: Clone
        uses: actions/checkout@v4
      - name: Download artifacts
        uses: actions/download-artifact@v4.1.8
      - name: Upload coverage data
        uses: codecov/codecov-action@v4
        with:
          name: Server
          token: ${{ secrets.CODECOV_TOKEN }}
          fail_ci_if_error: true
          verbose: true
          flags: server
  dispatch:
    runs-on: "ubuntu-latest"
    needs: [test, migrate]  # only if tests succeeded
    # and only if label 'trigger-downstream-ci' is set
    if: ${{ contains( github.event.pull_request.labels.*.name, 'trigger-downstream-ci') }}
    strategy:
      matrix:
        repo:
          # repos can subscribe by adding themselves here
          - frappe/erpnext
          - frappe/lending
          - frappe/hrms
    steps:
      - name: Dispatch Downstream CI (if supported)
        uses: peter-evans/repository-dispatch@v3
        with:
          token: ${{ secrets.CI_PAT }}
          repository: ${{ matrix.repo }}
          event-type: frappe-framework-change
          client-payload: '{"frappe_sha": "${{ github.sha }}"}'  # see above, frappe_sha is set as FRAPPE_BRANCH for downstream checkout

```