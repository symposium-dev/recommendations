# Symposium Recommendations

This repository contains the recommended agent mods for [Symposium](https://github.com/symposium-dev/symposium).

Symposium fetches these recommendations on startup to suggest relevant mods for your workspace. Recommendations are matched based on workspace characteristics like the presence of specific files or dependencies.

## Structure

```
recommendations/
├── sparkle.toml           # Single-file recommendation
├── cargo.toml
├── rust-analyzer.toml
└── my-mod/                # Directory-based recommendation
    └── config.toml
```

Each recommendation is either:
- A `.toml` file directly in `recommendations/` (e.g., `recommendations/sparkle.toml`)
- A directory containing `config.toml` (e.g., `recommendations/my-mod/config.toml`)

CI validates each file and publishes a concatenated `recommendations.toml` to GitHub Pages.

## Recommendation Format

Each file contains a single `[recommendation]` block:

```toml
# recommendations/example.toml

[recommendation]
source.cargo = { crate = "example-mod" }
when.file-exists = "Cargo.toml"
```

### Sources

- `source.cargo` - Install from crates.io via `cargo install`
- `source.npx` - Install from npm via `npx`
- `source.pipx` - Install from PyPI via `pipx`

### Conditions

- `when.file-exists` - Recommend when a file exists in the workspace
- `when.files-exist` - Recommend when all listed files exist
- `when.using-crate` - Recommend when a Rust crate is a dependency
- `when.using-crates` - Recommend when all listed crates are dependencies
- `when.any` - Recommend when any nested condition matches
- `when.all` - Recommend when all nested conditions match

Mods without conditions are always recommended.

## Contributing

To add a recommendation for your mod:

1. Create a new file in `recommendations/` (e.g., `recommendations/your-mod.toml`)
2. Add your recommendation in the format shown above
3. Submit a pull request

CI will validate your recommendation format automatically.

## License

This repository is licensed under the Apache License 2.0. See [LICENSE.txt](LICENSE.txt) for details.
