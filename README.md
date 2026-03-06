# ch-migrate GitHub Action

A composite GitHub Action that installs [ch-migrate](https://github.com/DRYCodeWorks/clickhouse-migrate), starts a ClickHouse server container, and runs bootstrap, migrations, and linting.

## Usage

```yaml
- uses: DRYCodeWorks/ch-migrate-action@v1
  with:
    environment: ci
    clickhouse-version: '24.8'
    lint: 'true'
```

### Full example workflow

```yaml
name: Migrations CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  migrate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: DRYCodeWorks/ch-migrate-action@v1
        with:
          environment: ci
          clickhouse-version: '24.8'
          lint: 'true'
```

Your project must include a `config.yaml` with a matching environment. For CI, use HTTP (not HTTPS) on port 8123:

```yaml
environments:
  ci:
    host: localhost
    port: 8123
    secure: false
    database: my_db
    migration_user: my_migration_user
    admin_user: default
```

### Install from local source

To test your own fork or unreleased version of ch-migrate, set `ch-migrate-version` to `.` and check out the ch-migrate repo first:

```yaml
- uses: actions/checkout@v4
- uses: DRYCodeWorks/ch-migrate-action@v1
  with:
    ch-migrate-version: '.'
    environment: ci
```

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `environment` | `ci` | ch-migrate environment name (must match `config.yaml`) |
| `clickhouse-version` | `24.8` | ClickHouse server Docker image tag |
| `python-version` | `3.12` | Python version to install |
| `ch-migrate-version` | `clickhouse-alembic` | pip install specifier (use `.` for local) |
| `lint` | `true` | Run `ch-migrate lint` after migrations |
| `bootstrap` | `true` | Run `ch-migrate bootstrap` before migrations |
| `migrate` | `true` | Run `ch-migrate up` after bootstrap |
| `clickhouse-port` | `8123` | Host port for ClickHouse HTTP interface |

## Outputs

| Output | Description |
|--------|-------------|
| `clickhouse-container-id` | Docker container ID of the running ClickHouse server |

## Engine compatibility

ClickHouse Cloud uses `SharedMergeTree` / `SharedReplacingMergeTree` engines which are not available in self-hosted ClickHouse. When writing migrations for CI, use standard engine families (`MergeTree`, `ReplacingMergeTree`, etc.) or use `{engine_suffix}` template variables to switch between Cloud and self-hosted modes.

## License

MIT
