# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains the documentation website for **AReaL** (Agentic Reinforcement Learning), a comprehensive framework for training large language models using reinforcement learning techniques like GRPO, PPO, DPO, and rejection sampling. The project is built with Jupyter Book (Sphinx) and provides bilingual (English/Chinese) documentation.

### Key Characteristics

- **Documentation site only**: This repo contains the published documentation, not the main AReaL source code (which lives in the `areal-project/AReaL` repo). The docs reference examples and APIs from the main project.
- **Bilingual structure**: Parallel English (`en/`) and Chinese (`zh/`) documentation trees with shared figures and static assets.
- **Auto-generated CLI docs**: The `cli_reference.md` files are generated from `generate_cli_docs.py` which introspects `areal.api.cli_args` dataclasses.
- **Jupyter Book build**: Uses `jupyter-book` (v1.x) with custom language toggle functionality.

## Repository Structure

```
areal-website/
├── en/                          # English documentation source
│   ├── _config.yml             # Jupyter Book config (English)
│   ├── _toc.yml                # Table of contents
│   ├── intro.md
│   ├── cli_reference.md        # Auto-generated CLI docs (large)
│   ├── version_history.md
│   ├── tutorial/               # Tutorials and guides
│   ├── algorithms/             # Algorithm explanations
│   ├── best_practices/         # Best practices guides
│   ├── customization/          # Customization guides
│   └── reference/              # API/reference documentation
├── zh/                         # Chinese documentation (parallel structure)
├── _static/                    # Shared static assets (JS/CSS)
│   ├── js/lang-toggle.js       # Language toggle functionality
│   └── css/lang-toggle.css     # Language toggle styling
├── figures/                    # Shared images/figures
├── build_all.sh                # Build script for both versions
├── generate_cli_docs.py        # CLI docs generator
├── requirements.txt            # Python dependencies
└── .github/workflows/          # CI/CD workflows
    └── deploy-docs.yml         # GitHub Pages deployment
```

## Development Workflow

### Building the Documentation

**Full build (both languages):**
```bash
./build_all.sh
```

This script:
1. Cleans previous builds
2. Sets up static file directories for both languages
3. Builds English version with `jupyter-book build`
4. Builds Chinese version with `jupyter-book build`
5. Flattens directory structure
6. Copies static assets and figures
7. Creates root redirect page

**Output:** `_build/en/` and `_build/zh/` directories with HTML files.

**Individual language build** (manual):
```bash
# English
uv run --with-requirements requirements.txt jupyter-book build en --all --path-output _build/en

# Chinese
uv run --with-requirements requirements.txt jupyter-book build zh --all --path-output _build/zh
```

### Regenerating CLI Reference Documentation

The CLI reference docs are auto-generated from the `areal.api.cli_args` module:

```bash
python3 generate_cli_docs.py
```

This requires:
- The `areal` package to be importable (the script adds the project root to `sys.path`)
- `mdformat` for formatting
- The main AReaL repo to be present at the expected location (parent of this repo)

**Output:** Updates `en/cli_reference.md` and `zh/cli_reference.md`.

### Local Development Server

To preview changes locally:

```bash
# English
cd en && uv run jupyter-book build . --all --path-output _build --builder html

# Or use livereload (if available)
cd en && uv run jupyter-book build . --all --path-output _build --builder html --watch
```

### Testing Changes

There are no automated tests in this repo. Verify changes by:
1. Running `./build_all.sh` to ensure the build succeeds
2. Checking that both English and Chinese versions build without errors
3. Verifying the generated HTML looks correct in `_build/`
4. For CLI doc changes, ensure `generate_cli_docs.py` runs without errors

## Common Tasks

### Edit Documentation Content

**Markdown files are in:**
- `en/` for English content
- `zh/` for Chinese content

Keep both versions in sync when updating documentation. The file structure is parallel between the two directories.

### Update CLI Reference

1. Ensure you have access to the main AReaL repository (it must be importable)
2. Run: `python3 generate_cli_docs.py`
3. Commit the updated `cli_reference.md` files

**Note:** The generator uses `mdformat` with specific options (wrap at 88 chars, GFM + tables + frontmatter extensions).

### Add a New Documentation Page

1. Create the English version: `en/path/to/page.md`
2. Create the Chinese version: `zh/path/to/page.md`
3. Add entries to both `en/_toc.yml` and `zh/_toc.yml`
4. Run `./build_all.sh` to verify

### Add or Update Figures

Place images in `figures/` directory. They're automatically copied to both language builds.

Reference in Markdown:
```markdown
![Description](../figures/image.png)
```

### Modify Build Configuration

- **Build settings:** `en/_config.yml` and `zh/_config.yml` (keep in sync)
- **Table of contents:** `en/_toc.yml` and `zh/_toc.yml`
- **Build script:** `build_all.sh`
- **Dependencies:** `requirements.txt`

### Update Language Toggle

The language toggle is implemented via:
- `_static/js/lang-toggle.js` - JavaScript logic
- `_static/css/lang-toggle.css` - Styling

Both are copied to each language's `_static/` directory during build.

## Key Architecture Details

### Language Toggle System

The bilingual setup uses:
1. **Root redirect** (`_build/index.html`): Auto-redirects to English or Chinese based on `localStorage` preference
2. **Footer links**: Each page has language switcher links in the footer (via `lang-toggle.js`)
3. **Shared static files**: Copied to each build during the build process

### CLI Documentation Generation

The `generate_cli_docs.py` script:
- Discovers all dataclasses in `areal.api.cli_args`
- Categorizes them (Core Experiment, Training, Inference, Dataset, System, Logging, Others)
- Generates a table for each class with: Parameter | Type | Default | Description
- Creates anchor links for cross-referencing
- Supports both English and Chinese output

### Jupyter Book Configuration

Key settings in `_config.yml`:
- `execute_notebooks: force` - Notebooks are re-executed on each build
- `use_issues_button` / `use_repository_button` - GitHub integration
- `extra_footerjs` / `extra_css` - Language toggle assets
- `html_extra_path` - Static files and figures

### Deployment

The GitHub Actions workflow (`deploy-docs.yml`):
- Triggers on push to `main` branch
- Builds in the `docs/` directory (note: workflow expects files in `docs/`, but repo root is the actual root)
- Deploys to GitHub Pages

**⚠️ Important:** The workflow has `cd docs` which suggests the repo might be structured as a subdirectory in the actual deployment. Verify the deployment setup matches the repo structure.

## Troubleshooting

### Build fails with import errors

Ensure dependencies are installed:
```bash
uv sync --all-extras  # Or: uv run --with-requirements requirements.txt jupyter-book --version
```

### CLI docs generation fails

The script imports `areal.api.cli_args`. This requires:
1. The main AReaL repository to be present
2. The project root to be in `sys.path` (script handles this)
3. All dependencies of the `areal` package to be installed

If the main repo isn't available, CLI doc generation will fail. In that case, update the docs manually or skip regeneration.

### Language toggle not working

Verify that:
1. `_static/js/lang-toggle.js` exists in both `en/_static/js/` and `zh/_static/js/` after build
2. The footer config includes the extra JS: `extra_footerjs: ["_static/js/lang-toggle.js"]`
3. No JavaScript errors in browser console

### Figures not displaying

Ensure figures are in the `figures/` directory and referenced with the correct path (`../figures/...` in source files). The build script copies figures to each language's build directory.

## Best Practices

1. **Keep both languages in sync**: When adding/updating content, create both English and Chinese versions
2. **Run full build before committing**: `./build_all.sh` catches structural issues
3. **Verify CLI docs**: If dataclasses changed in the main repo, regenerate CLI reference
4. **Use relative paths**: For cross-references within docs, use relative paths
5. **Follow Jupyter Book conventions**: Use proper Markdown, frontmatter where needed
6. **Test locally**: Preview changes with a local build before pushing

## Dependencies

- `jupyter-book<2` - Documentation site generator
- `matplotlib` - For any notebook-generated figures
- `numpy` - For numerical operations in notebooks
- `mdformat` - For formatting generated CLI docs (used by generate_cli_docs.py)

## Related Resources

- **Main AReaL Repository**: https://github.com/areal-project/AReaL
- **Quickstart Guide**: `en/tutorial/quickstart.md`
- **CLI Arguments Reference**: `en/cli_reference.md` (auto-generated)
- **Algorithm Documentation**: `en/algorithms/`

## Notes for Future Maintainers

- The `examples/` directory referenced in docs lives in the main AReaL repo, not here
- CLI reference docs are large (~320KB) - they're comprehensive but may load slowly
- The deployment workflow may need adjustment if the repo structure doesn't match the `cd docs` expectation
- Consider adding a pre-commit hook to verify both language versions are in sync