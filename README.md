# Clean Code GitHub Actions

An opinionated practices for manageable GitHub Actions workflows

## Workflow Files

### Reusable workflow files with leading underscore

```txt
$ tree -a
├── .github
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── workflows
│       └── workflows
│           ├── _build.yml
│           ├── _container.yml
│           ├── _lint.yml
│           ├── nightly.yml
│           ├── pull_request.yml
│           └── push_containers.yml
├── src
├── dist
└── ...
```

### Visual aid for group in steps

To reduce developer's overhead, this example will create line in GitHub Actions web UI, functions as a visual aid for rather crowded steps in single job. Intentionally minified for visual in yaml. `date` command generates harmless output, potentially useful for debugging timestamp between steps.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - { name: "Section [1/2] - Prepare source", run: date }
      - uses: actions/checkout@v3
      - run: echo "processing things..."

      - { name: "Section [2/2] - Build and install", run: date }
      - run: ./configure && make && make install
```

### Adding comments in steps

It is common to having `FIXME: ...` or `TODO: ...` comments in workflow files. To leave comment to a step with minimal overhead:

```yaml
jobs:
  build:
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      # FIXME: This place requires overhead on processing indent, confusing target line/ste.
      - name: Print current date
        run: date
      - # NOTE: strictly bound to this step, allowing compact arrangement.
        name: Print current directory
        run: pwd
```

### Printing step stdout to `GITHUB_STEP_SUMMARY`

In addition to step summary([docs.github.com](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#adding-a-job-summary)), it is convinent to print to stdout at the same time.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - # Print only to summary, contents are not available in workflow run details
        name: Print github contexts 
        run: |
          echo "| key               | value                        |" >> $GITHUB_STEP_SUMMARY
          echo "| ----------------- | ---------------------------- |" >> $GITHUB_STEP_SUMMARY
          echo "| github.event_name | \`${{ github.event_name }}\` |" >> $GITHUB_STEP_SUMMARY
          echo "| github.ref        | \`${{ github.ref }}\`        |" >> $GITHUB_STEP_SUMMARY

      - # use `tee -a` to keep terminal output, multiplexing to summary. use `>>` to reduce verbosity in formatting output.
        name: Print system information 
        run: |
          echo "\`\`\` >> $GITHUB_STEP_SUMMARY
          lscpu  | tee -a $GITHUB_STEP_SUMMARY
          echo "\`\`\` >> $GITHUB_STEP_SUMMARY
```

## Syntax

### status check functions

https://docs.github.com/en/actions/learn-github-actions/expressions#status-check-functions

- Avoid to use evaluation syntax(`${{ }}`) for single status check function

```yaml
- name: Checkout
  if: always()
  uses: actions/checkout@v3
```

- Use `${{ }}` block for evaluating expression

```yaml
- name: Print exit message
  if: ${{ cancelled || failure() }}
  run: echo "Exiting workflow run..."
```

### wraping runtime env

- wrap user-defined environemtn variable with `${}`
- do not wrap runner-provided environemt variables(usually prefixed with `GITHUB`)

for example: `echo ${FOO} >> $GITHUB_STEP_SUMMARY`

### inputs and env 

When you want to configure reusable workflow to be triggered with `workflow_dispatch: {}`
it is rather convient to marshalling input with env

```yaml
on:
  workflow_dispatch:
    inputs:
      foo:
        type: string
        default: 'bar'
env:
  FOO: ${{ inputs || 'default value here' }}
```

It is more cannonical to use capital letter as a environment variable, to match with POSIX convention

#### Letter case for inputs & env

- if `inputs.FOO` directly goes into `env`, use capitals
- if `inputs.bar` requires further processing or not used for `env` nor environment variables, use lowercases

```yaml
on:
  workflow_dispatch:
    inputs:
      DOCKER_BUILDKIT:
        type: string
        default: "1"
      condition:
        type: string
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: build docker image
        if: ${{ inputs.condition == 'docker' }}
        run: docker build .
        env:
          DOCKER_BUILDKIT: ${{ inputs.DOCKER_BUILDKIT }}
```

### scope of env

When inputs or envs only used in a few steps, especially running very simillar steps multiple times,

- avoid overriding env with `echo FOO=bar >> $GITHUB_ENV`

```yaml
- name: Print input
  run: echo ${FOO} >> $GITHUB_STEP_SUMMARY
  env:
    FOO: bar

- name: Print with override
  run: echo ${FOO} >> $GITHUB_STEP_SUMMARY
  env:
    FOO: 'not bar'

