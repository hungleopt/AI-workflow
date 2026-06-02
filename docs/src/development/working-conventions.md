# Working Conventions

This chapter migrates the current contributor workflow notes into the handbook structure.

## Git hooks

Configure the repository to use the shared hooks:

```bash
git config core.hooksPath .git-hooks
```

On Linux or macOS, make the hook scripts executable:

```bash
chmod +x .git-hooks/commit-msg
chmod +x .git-hooks/pre-commit
chmod +x .git-hooks/pre-push
```

## Commit messages

Allowed message shapes include:

- `<type>: #<ticket-id> - <short-description>`
- `<type>(<scope>): #<ticket-id> - <short-description>`
- merge commits beginning with `Merge`

Common types include `feat`, `fix`, `docs`, `refactor`, `test`, `build`, `ci`, `chore`, `perf`, `security`, and `deps`.

## Branch naming

Expected remote branch formats include:

- `<type>/<ticket-number>-<short-description>` where the documented types include `features`, `bugfixes`, `ci`, and `refactor`
- fixed environment branches such as `inte`, `inte2`, `prep`, `prep2`, `prod`, and `prod2`
- baseline branches such as `development`, `develop`, `develop2`, or `master`

## Bypass flow

The legacy docs document bypassing hooks with `--no-verify`. That remains possible, but it should be treated as an exception rather than normal workflow.

## Documentation note

The existing repository uses both `develop` and environment branches in different contexts. Keep the branch name exactly as implemented in the relevant pipeline instead of normalizing it in docs.
