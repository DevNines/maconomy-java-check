# Maconomy Java Check

[![Marketplace](https://img.shields.io/badge/marketplace-Maconomy_Java_Check-blue?logo=githubactions)](https://github.com/marketplace/actions/maconomy-java-check)

A GitHub Action that compile-checks Java 8 sources against a curated bundle of libraries and writes every `javac` diagnostic as a PR annotation on the offending line.

Use of this action requires a valid API key issued by DevNines.

## Usage

```yaml
name: Java Check
on: [pull_request]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: DevNines/maconomy-java-check@v1
        with:
          api-key:     ${{ secrets.JAVA_CHECK_API_KEY }}
          source-path: src/main/java
          fail-on:     error
```

A complete copy-paste workflow is in [`examples/javacheck.yml`](examples/javacheck.yml).

## Inputs

| Name | Required | Default | Description |
|---|---|---|---|
| `api-key` | yes | — | Bearer token issued by your JavaCheck operator. Store as a repo secret (recommended name: `JAVA_CHECK_API_KEY`). |
| `source-path` | no | `.` | Directory whose `.java` files should be checked. Adjust to your layout (`src/main/java`, `src`, etc.). |
| `fail-on` | no | `error` | Severity threshold that fails the step. One of `info`, `warning`, `error`. |
| `api-url` | no | (set by the action) | Override only if your operator has provisioned a non-default endpoint for you. |
| `timeout-seconds` | no | `300` | Max time to wait for the job to terminate. |

## Outputs

| Name | Description |
|---|---|
| `job-id` | The analysis job ID, useful for log correlation. |
| `findings-count` | Total findings returned (compile + analysis combined). |
| `status` | `success`, `failed`, or `timeout`. |

## Pinning

For reproducibility, pin to a release tag:

```yaml
- uses: DevNines/maconomy-java-check@v1.0.0
```

`@v1` tracks the latest `v1.x.y`. `@main` always pulls tip — convenient for early adoption, less reproducible.

## License

Proprietary — see [LICENSE](LICENSE). Use requires a valid API key issued by DevNines. Redistribution, forking for a competing service, and reverse-engineering are not permitted without prior written consent.
