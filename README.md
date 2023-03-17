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
