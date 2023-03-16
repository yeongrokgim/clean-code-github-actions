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
