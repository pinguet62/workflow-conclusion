# Workflow Conclusion

GitHub Action to get last workflow conclusion.

## Usage

### Inputs 📥

| Name          | Required? | Default        | Description                              |
|---------------|-----------|----------------|------------------------------------------|
| `workflow_id` | `true`    |                | The ID of filename of workflow           |
| `branch`      | `false`   | current branch | Target branch                            |
| `skip`        | `false`   | `0` (latest)   | Number of workflow executions to exclude |

### Outputs 📤

| Output                | Description                                                         |
|-----------------------|---------------------------------------------------------------------|
| `workflow_conclusion` | `"success"`, `"failure"`, `"cancelled"`, or null if yet in progress |

## Real use case

### Deploy after success

Trigger or Block workflow when another failed.

```yaml
on:
  workflow_run:
    workflows: [ 'Build & Test' ]
    types: [ 'completed' ]
jobs:
  publish:
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

### Single notification, when status changed

Avoid multiple notifications when consecutive success or failure.

```yaml
# test.yml
on:
  push:
jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - id: previous-execution
        uses: pinguet62/workflow-conclusion@main
        with:
          workflow_id: 'test.yml' # itself
          skip: 1 # ignore current
      - id: latest-execution
        run: test.sh
      - if: always() && steps.previous-execution.outputs.workflow_conclusion == 'failure' && steps.latest-execution.conclusion == 'success'
        run: echo "Fixed..."
      - if: always() && steps.previous-execution.outputs.workflow_conclusion == 'success' && steps.latest-execution.conclusion == 'failure'
        run: echo "Failure..."
```
