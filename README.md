# Maconomy Java Check

[![Marketplace](https://img.shields.io/badge/marketplace-Maconomy_Java_Check-blue?logo=githubactions)](https://github.com/marketplace/actions/maconomy-java-check)

A GitHub Action that compile-checks Java 8 sources against a curated bundle of libraries, then writes every `javac` diagnostic as a PR annotation on the offending line.

The heavy lifting (sandbox container, bundled JARs, analysis tool) runs on a hosted backend at `https://maconomyaction.devnines.com`. This action is the thin client: it tar-gzips your source, POSTs it, polls until the job is terminal, and renders findings as `::error` / `::warning` / `::notice` annotations.

## Usage

```yaml
name: Java Check
on: [pull_request]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: DevNines/javacheck-action@v1
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
| `api-url` | no | `https://maconomyaction.devnines.com` | Override only if you point at a non-default JavaCheck deployment. |
| `timeout-seconds` | no | `300` | Max time to wait for the job to terminate. |

## Outputs

| Name | Description |
|---|---|
| `job-id` | The analysis job ID, useful for log correlation. |
| `findings-count` | Total findings returned (compile + analysis combined). |
| `status` | `success`, `failed`, or `timeout`. |

## What it does

1. Tar-gzips `source-path` (excluding `.git/`, `node_modules/`, `target/`).
2. POSTs the tarball to `${api-url}/jobs` with the bearer token.
3. Polls `${api-url}/jobs/${jobId}` every 3 seconds until terminal.
4. Annotates every finding on the offending file/line in the PR.
5. Sets the step's exit status based on `fail-on`.

If the deployed JavaCheck endpoint is unreachable, the run fails with `API returned <status>: <body>` and the workflow goes red.

## Pinning

For reproducibility, pin to a release tag:

```yaml
- uses: DevNines/javacheck-action@v1.0.0
```

`@v1` tracks the latest `v1.x.y`. `@main` always pulls tip — convenient for early adoption, less reproducible.

## Source layout

The runtime artefact (`dist/index.js`) is the production-minified bundle, committed because GitHub Actions runs it directly. The action source proper lives in the operator's private repo (`ShadyNagy/MaconomyGitHubAction` under `java-check-action/action/`). This directory mirrors the public release for Marketplace consumption — when shipping a new version, the contents of `java-check-action/action/` here get copied to the root of `DevNines/javacheck-action` and tagged.

## License

Proprietary — see [LICENSE](LICENSE). Use of this action requires a valid API key issued by DevNines for the JavaCheck service. Redistribution, forking for a competing service, and reverse-engineering are not permitted without prior written consent.
