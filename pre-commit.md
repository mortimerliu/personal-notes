# `pre-commit`

## Users

### Installation

```bash
pip install pre-commit
pre-commit --version
```

### Enable `pre-commit` for a project

1. Create `.pre-commit-config.yaml` to configure hooks to use under the root directory of the project
   1. [All options](https://pre-commit.com/index.html#plugins)
   2. [Supported hooks](https://pre-commit.com/hooks.html)
   ```yaml
   repos:
   -   repo: https://github.com/pre-commit/pre-commit-hooks
       rev: v2.3.0
       hooks:
       -   id: check-yaml
       -   id: end-of-file-fixer
       -   id: trailing-whitespace
   -   repo: https://github.com/psf/black
       rev: 22.10.0
       hooks:
       -   id: black
   -   repo: https://github.com/example/repo
       rev: v1.1.0
       hooks:
       -   id: example-id
           args: [-example args] # optional
   ```
2. Install git hook by running the following command under the project directory. This will enable running `pre-commit` automatically on `git commit`
   ```bash
   pre-commit install
   ```

To update `pre-commit` hooks:
```bash
pre-commit autoupdate
```

Options: 

* `--bleeding-edge`: update to the bleeding edge of the default branch instead of the latest tagged version (the default behaviour).
* `--freeze`: Store "frozen" hashes in `rev` instead of tag names.
* `--repo REPO`: Only update this repository. This option may be specified multiple times.

To run all `pre-commit` on all files:

```bash
pre-commit run --all-files
```

## Developers

`pre-commit` currently supports hooks written in many languages. As long as your git repo is an installable package (gem, npm, pypi, etc.) or exposes an executable, it can be used with `pre-commit`. Each git repo can support as many languages/hooks as you want.

The hook must exit nonzero on failure or modify files.

### `.pre-commit-hooks.yaml`

A git repo containing `pre-commit` plugins must contain a `.pre-commit-hooks.yaml` file

Example:

```
- id: leetcode-readme
  name: leetcode-readme
  description: Auto-generate README.md for [LeetCode](https://github.com/mortimerliu/LeetCode)
  entry: leetcode-readme
  language: python
  types: [python]
```

### Python

The hook repository must be installable via `pip install .` (usually by either `setup.py` or `pyproject.toml`). The installed package will provide an executable that will match the `entry` – usually through `console_scripts` or `scripts` in `setup.py`.

A simple example hook: [leetcode-readme](https://github.com/mortimerliu/leetcode-readme)

### Developing hooks interactively

Since the `repo` property of `.pre-commit-config.yaml` can refer to anything that `git clone ...` understands, it's often useful to point it at a **local directory** while developing hooks.

`pre-commit try-repo` streamlines this process by enabling a quick way to try out a repository:

```bash
# ../hook-repo is the path to the local repo
# foo is the hook id
pre-commit try-repo ../hook-repo foo --verbose --all-files
```

note: you may need to provide `--commit-msg-filename` when using this command with hook types `prepare-commit-msg` and `commit-msg`.

a commit is not necessary to `try-repo` on a local directory. `pre-commit` will clone any tracked uncommitted changes.