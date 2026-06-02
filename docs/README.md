# FirstMile Documentation

This folder now contains the source for the internal mdBook handbook.

## Layout

- `book.toml`: mdBook configuration.
- `src/`: handbook source files.
- legacy Markdown files at the root of `docs/`: earlier standalone notes kept as source material during migration.

## Local build

If `mdbook` is available locally:

```bash
cd docs
mdbook build
mdbook serve
```

The generated output is written to `docs/book/`.
