# Workflow Conclusion

GitHub Action to get last workflow conclusion.

## Usage

### Inputs 📥

| Name          | Required? | Default        | Description                              |
|---------------|-----------|----------------|------------------------------------------|
| `workflow_id` | `true`    | /              | The ID of filename of workflow           |
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
