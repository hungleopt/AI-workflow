# Git Hooks

Go to the root directory of your repository and run the following command to set the Git hooks path to the `.git-hooks` folder:

```bash wrap
git config core.hooksPath .git-hooks
```

If you're on Linux or macOS, make sure the hook scripts are executable by running:

```bash wrap
chmod +x .git-hooks/commit-msg
chmod +x .git-hooks/pre-commit
chmod +x .git-hooks/pre-push
```

## What This Enables

Once configured, the following Git hooks will be enforced:

1. Commit messages must follow one of these formats:
  
   - Match `<type>: #<ticket-id> - <short-description>`.  
     Example: *feat: FMI-12345 - header implementation*

   - Match `<type>(<scope>): #<ticket-id> - <short-description>`.  
     Example: *fix(hero): FMI-12345 - extra gap at the bottom*

   - Start with `Merge` (case insensitive).  
     Example: *Merge from development*

   Available types: `feat`, `fix`, `build`, `chore`, `ci`, `docs`, `style`, `refactor`, `perf`, `test`, `revert`, `security`, `deps` and `wip`.  
   For more information, see [Conventional Commits](https://www.conventionalcommits.org/) and Angular's [Commit Message Guidelines](https://github.com/angular/angular/blob/main/contributing-docs/commit-message-guidelines.md).

1. Remote branches must follow one of these naming conventions:

   - Match `<type>/<ticket-number>-<short-description>`  
     Available types: `features`, `bugfixes`, `ci` and `refactor`.  
     Example: *features/FMI-12345-header-implementation*

   - One of `inte`, `inte2`, `prep`, `prep2`, `prod` or `prod2`.

   - One of `development`, `develop`, `develop2`, or `master`.

## Types

Here are the common types used in Git conventional commit messages:

Primary Types:

- `feat` - a new feature for the user
- `fix` - a bug fix for the user
- `docs` - documentation only changes
- `style` - formatting, missing semi-colons, etc. (no code change)
- `refactor` - refactoring production code (no new features or bug fixes)
- `perf` - performance improvements
- `test` - adding or refactoring tests (no production code change)
- `build` - changes to build system or dependencies
- `ci` - changes to CI configuration files and scripts
- `chore` - updating grunt tasks, etc. (no production code change)

Additional Types (sometimes used):

- `revert` - reverting a previous commit
- `security` - security fixes or improvements
- `deps` - dependency updates (alternative to using as a scope)
- `wip` - work in progress (typically not merged to main)

Breaking Changes:

- Add `!` after the type to indicate breaking changes: `feat!:` or `fix!:`
- Or add `BREAKING CHANGE:` in the commit body

## Bypass

Incase there is a need to use other commit message, then follow these steps:

1. Stage the files needed for the commit.

1. Bypass the **commit-msg** hook by running the folling command:

   ```bash wrap
   git commit -m '<your message here>' --no-verify
   ```

   `--no-verify` flag tells git to skip the check.

1. Bypass the **pre-push** hook by executing the following command:

   ```bash wrap
   git push origin development:remote-branch --no-verify
   ```

