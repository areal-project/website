# AReaL Documentation Website

Bilingual (English / 中文) documentation for AReaL, built with [Jupyter Book](https://jupyterbook.org/).

## Structure

- `en/` — English source (Markdown + `_toc.yml` / `_config.yml`)
- `zh/` — Chinese source
- `figures/` — Shared images
- `_static/` — Language toggle JS/CSS
- `build_all.sh` — Builds both languages into `_build/`
- `generate_cli_docs.py` — Generates CLI reference from AReaL dataclasses

## Build

Requires [uv](https://github.com/astral-sh/uv).

```bash
./build_all.sh
```

Output:

- English: `_build/en/index.html`
- Chinese: `_build/zh/index.html`
- Root (auto-redirect): `_build/index.html`

## Requirements

See [requirements.txt](requirements.txt): `jupyter-book<2`, `matplotlib`, `numpy`.
