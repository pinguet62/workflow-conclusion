# Workflow Conclusion

GitHub Action to get last workflow conclusion.

## Usage

```yaml
on: # any
jobs:
  sample:
    runs-on: ubuntu-latest
    steps:
      - id: example
        uses: pinguet62/workflow-conclusion@main
        with:
          workflow_id: 'sample.yml'
          branch: 'main'
      - run: echo "${{steps.example.outputs.workflow_conclusion}}"
```

### Inputs 📥

| Name          | Required? | Default        | Description                              |
|---------------|-----------|----------------|------------------------------------------|
| `workflow_id` | `true`    |                | The ID of filename of workflow           |
| `branch`      | `false`   | current branch | Target branch                            |
| `skip`        | `false`   | `0` (latest)   | Number of workflow executions to exclude |

### Outputs 📤

| Output                | Description                                                           |
|-----------------------|-----------------------------------------------------------------------|
| `workflow_conclusion` | `"success"`, `"failure"`, `"cancelled"`, or `null` if yet in progress |

## Example of real use cases

### Block PR when `main` build failed

<details>
  <summary>Add job to check last build, then status check to forbid merge...</summary>

```yaml
# test.yml
on:
  push:
    branches: [ 'main' ]
  pull_request:
jobs:
  tests:
    runs-on: ubuntu-latest
    steps:
      - run: ./test.sh
  main-stable:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - id: tests-main
        uses: pinguet62/workflow-conclusion@main
        with:
          workflow_id: 'test.yml'
          branch: 'main'
      - if: steps.tests-main.outputs.workflow_conclusion == 'failure'
        run: |
          echo "Fix main before next merges..."
          exit 1
```

</details>

### Notify only when conclusion changed

<details>
  <summary>Avoid multiple notifications when consecutive success or failure...</summary>

```yaml
# test.yml
on:
  push:
    branches: [ 'main' ]
jobs:
  tests:
    runs-on: ubuntu-latest
    steps:
      - id: latest-execution
        run: ./test.sh

      - id: previous-execution
        uses: pinguet62/workflow-conclusion@main
        with:
          workflow_id: 'test.yml' # itself
          skip: 1 # ignore current

      # notification
      - if: always() && steps.previous-execution.outputs.workflow_conclusion == 'success' && steps.latest-execution.conclusion == 'failure'
        run: echo "Failure..."
      - if: always() && steps.previous-execution.outputs.workflow_conclusion == 'failure' && steps.latest-execution.conclusion == 'success'
        run: echo "Fixed..."
```

</details>

### Deploy after success

<details>
  <summary>Trigger new workflow after another success...</summary>

```yaml
# deploy.yml
on:
  workflow_run:
    workflows: [ 'Build & Test' ]
    branches: [ 'main' ]
    types: [ 'completed' ]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - id: build_test-result
        uses: pinguet62/workflow-conclusion@main
        with:
          workflow_id: 'build_test.yml'
          branch: 'main'

      - if: steps.build_test-result.outputs.workflow_conclusion == 'success'
        run: echo "Deploy..."
```

</details>
