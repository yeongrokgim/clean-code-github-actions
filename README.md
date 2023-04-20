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

