# verbaria/translate-sync

A GitHub Action that keeps a repository's translations in sync with a
[Verbaria](https://github.com/verbaria/verbaria) translation server.

On each run it:

1. Sets up **JDK 25** (configurable).
2. Downloads the **latest Verbaria CLI** release archive.
3. **Skips** if an open translation pull request already exists.
4. **Backs up** the existing `verbaria-lock.json` (if any) before pulling.
5. Runs **`verbaria pull`** to fetch the latest translations.
6. Generates a **git commit message** and a **Markdown changelog** by diffing the
   old and new `verbaria-lock.json` (via `verbaria changelog`).
7. **Does nothing** when the translations are unchanged.
8. **Opens (or updates) a pull request** with the changes.

## Usage

```yaml
# .github/workflows/translations.yml
name: Sync translations
on:
  schedule:
    - cron: "0 6 * * *"   # daily at 06:00 UTC
  workflow_dispatch: {}

permissions:
  contents: write
  pull-requests: write

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: verbaria/translate-sync@v1
        with:
          server-url: https://translate.example.org/
          username: ${{ vars.VERBARIA_USERNAME }}
          api-key: ${{ secrets.VERBARIA_API_KEY }}
```

## Inputs

| Input                  | Required | Default                              | Description |
|------------------------|----------|--------------------------------------|-------------|
| `server-url`           | yes      | —                                    | Verbaria server URL. |
| `username`             | yes      | —                                    | Verbaria username. |
| `api-key`              | yes      | —                                    | Verbaria API key (use a secret). |
| `project-config`       | no       | `verbaria.json`                      | Path to the project config. |
| `working-directory`    | no       | `.`                                  | Where `verbaria.json` lives. |
| `pull-type`            | no       | `trans`                              | `trans`, `source` or `both`. |
| `pull-args`            | no       | `""`                                 | Extra args for `verbaria pull`. |
| `client-repo`          | no       | `verbaria/verbaria`                  | Repo publishing the CLI archive. |
| `client-version`       | no       | `latest`                             | CLI release tag, or `latest`. |
| `client-download-url`  | no       | `""`                                 | Explicit `verbaria-*.tar.gz` URL (overrides repo/version). |
| `java-version`         | no       | `25`                                 | JDK version. |
| `java-distribution`    | no       | `temurin`                            | `actions/setup-java` distribution. |
| `branch`               | no       | `verbaria/translations`              | PR head branch. |
| `base-branch`          | no       | current branch                       | PR base branch. |
| `pr-title`             | no       | `chore(i18n): sync translations`     | PR title. |
| `pr-labels`            | no       | `translations`                       | Comma-separated PR labels. |
| `commit-author-name`   | no       | `verbaria-bot`                       | Commit author name. |
| `commit-author-email`  | no       | `verbaria-bot@users.noreply.github.com` | Commit author email. |
| `github-token`         | no       | `${{ github.token }}`                | Token for CLI download + PR. |

## Outputs

| Output    | Description |
|-----------|-------------|
| `changed` | `true` when translations changed and a PR was created/updated. |
| `skipped` | `true` when an open translation PR already existed. |
| `pr-url`  | URL of the created/updated pull request. |

## Notes

- The lock diff drives both the commit message and the PR body, so an empty diff
  produces no commit and no PR.
- `github-token` needs `contents: write` and `pull-requests: write`; if the CLI
  release repo is private it must also be readable by the token.
- The CLI launcher selects its JVM from `VERBARIA_JRE`, which the action points at
  the JDK installed by `actions/setup-java`.
