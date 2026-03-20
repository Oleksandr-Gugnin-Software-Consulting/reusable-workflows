# reusable-workflows

Reusable GitHub Actions workflows derived from the CI/CD groundwork in Customer-Demos.

## Included workflows

- `reusable-detect-changes.yml`: categorizes changed files into `core`, `tests`, `docs`, `docker`, and `ci` buckets.
- `reusable-python-job.yml`: runs an arbitrary Python install and job command on a single configured Python version.
- `reusable-docker-build.yml`: builds a Docker image, runs a smoke container, verifies an HTTP endpoint, and reports image size.
- `reusable-deploy-command.yml`: checks out a repository, prepares an SSH key when provided, and executes a deployment or maintenance command.

`reusable-python-job.yml` can consume an inherited `OPENAI_API_KEY` secret and exposes it to the job environment for live LLM test runs.

## Typical usage

```yaml
jobs:
  tests:
    uses: Oleksandr-Gugnin-Software-Consulting/reusable-workflows/.github/workflows/reusable-python-job.yml@v1
    with:
      job_name: Test suite
      python_versions_json: '["3.11"]'
      install_command: |
        python -m pip install --upgrade pip
        python -m pip install -e ".[dev]"
      run_command: python -m pytest -q
```

```yaml
jobs:
  llm-backend:
    uses: Oleksandr-Gugnin-Software-Consulting/reusable-workflows/.github/workflows/reusable-python-job.yml@v1
    with:
      job_name: Live LLM backend tests
      install_command: |
        python -m pip install --upgrade pip
        python -m pip install -e ".[dev]"
      run_command: python -m pytest -q tests/llm_backend --run-llm-backend -n auto
    secrets: inherit
```

```yaml
jobs:
  deploy:
    uses: Oleksandr-Gugnin-Software-Consulting/reusable-workflows/.github/workflows/reusable-deploy-command.yml@v1
    with:
      job_name: Deploy production
      runner_labels_json: '["self-hosted", "linux", "mixfm2-deploy"]'
      command: ./scripts/deploy.sh
    secrets:
      ssh_private_key: ${{ secrets.DEPLOY_SSH_PRIVATE_KEY }}
```

`reusable-deploy-command.yml` runs on `ubuntu-latest` by default. To target a self-hosted runner, pass `runner_labels_json` as a JSON array of runner labels.