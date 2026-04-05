# gitsub

[![shell](https://img.shields.io/badge/shell-bash-121011.svg?logo=gnu-bash)](#)
[![platform](https://img.shields.io/badge/platform-linux%20%7C%20termux%20%7C%20macOS-blue)](#)
[![git](https://img.shields.io/badge/tool-git-orange?logo=git)](#)
[![license](https://img.shields.io/badge/license-MIT-green)](#)

`gitsub` is a lightweight Bash utility for detecting whether the **current Git repository** is a **submodule of a parent Git repository** somewhere above it on the same filesystem path.

It is built for shell workflows, scripts, prompts, and repo debugging, with support for:

- plain yes/no output
- quiet boolean checks
- explicit status output
- JSON output
- custom format strings
- relative-path selection
- parent/submodule path field extraction
- built-in Bash and Zsh completion generation

---

## Features

- Detect whether the current repo is a submodule of a parent repo
- Print the **immediate parent** repo
- Print the **outermost** repo in a nested submodule chain
- Print paths relative to:
  - the immediate parent repo
  - the outermost parent repo
  - the current repo
- Print path components for the outermost parent repo and current submodule repo:
  - parent dirname
  - parent basename
  - submodule prefix
  - submodule basename
  - current working directory prefix inside the submodule
- Pretty-print `$HOME` as `~` by default
- Output JSON for scripting
- Output custom formatted strings with tokens
- Generate completions for Bash and Zsh from the script itself

---

## Why use `gitsub`?

When you are deep inside a nested repo layout, it is not always obvious whether the repo you are in is:

- a standalone repo, or
- a submodule tracked by a repo above it

`gitsub` answers that directly and gives you useful context for automation and shell UX.

---

## Installation

Save the script as `gitsub`, then make it executable and move it somewhere in your `PATH`.

```bash
chmod +x gitsub
mv gitsub ~/.local/bin/
```

Make sure `~/.local/bin` is in your `PATH`.

---

## Quick start

Basic check:

```bash
gitsub
```

Output:

```text
yes
```

Boolean / scripting mode:

```bash
gitsub --exists-only
```

Status output:

```bash
gitsub --status
```

Output:

```text
submodule
```

Get the immediate parent repo:

```bash
gitsub --parent
```

Get the outermost parent repo:

```bash
gitsub --outermost
```

Get the path relative to the outermost parent:

```bash
gitsub --path-outermost
```

---

## Demo

Given this layout:

```text
~/parent_root/parent_prefix/path/submodule_repo_toplevel/subprefix/path/current
```

Assume:

- `parent_root` is the outermost parent repo
- `submodule_repo_toplevel` is the current submodule repo root
- `current` is your current working directory inside the submodule

Then:

```bash
gitsub -f '%D / %M / %G / %B / %W'
```

returns:

```text
~ / parent_root / parent_prefix/path / submodule_repo_toplevel / subprefix/path/current
```

Meaning:

- `%D` → dirname of outermost parent repo root
- `%M` → basename of outermost parent repo root
- `%G` → path to, but not including, the submodule basename
- `%B` → basename of current submodule repo root
- `%W` → path from current submodule root to `$PWD`

---

## Usage

```bash
gitsub [options]
```

Default behavior:

- prints `yes` if the current repo is a submodule of a parent repo
- prints `no` otherwise

Exit codes:

- `0` → current repo is a submodule of a parent repo
- `1` → current repo is not a submodule of a parent repo
- `2` → not inside a Git repo, or invalid usage

---

## Output modes

### Plain mode

```bash
gitsub
```

Outputs:

- `yes`
- `no`

### Quiet mode

```bash
gitsub --quiet
```

Prints nothing and uses the exit code only.

### Exists-only mode

```bash
gitsub --exists-only
```

Same behavior as `--quiet`, but reads more clearly in scripts.

Example:

```bash
if gitsub --exists-only; then
  echo "inside a submodule"
fi
```

### Status mode

```bash
gitsub --status
```

Outputs one of:

```text
submodule
```

or

```text
not-submodule
```

### JSON mode

```bash
gitsub --json
```

Example output:

```json
{
  "is_submodule": true,
  "current": "~/parent_root/parent_prefix/path/submodule_repo_toplevel",
  "parent": "~/parent_root/parent_prefix/path",
  "outermost": "~/parent_root",
  "parent_dirname": "~",
  "parent_basename": "parent_root",
  "submodule_prefix": "parent_prefix/path",
  "submodule_basename": "submodule_repo_toplevel",
  "cwd_prefix": "subprefix/path/current",
  "path": "submodule_repo_toplevel",
  "path_parent": "submodule_repo_toplevel",
  "path_outermost": "parent_prefix/path/submodule_repo_toplevel",
  "relative_to": "parent"
}
```

---

## Path behavior

By default, displayed repo paths are prettified so paths under `$HOME` are shown with `~`.

Example:

```text
/data/data/com.termux/files/home/code/mainrepo
```

becomes:

```text
~/code/mainrepo
```

To disable that:

```bash
gitsub --raw
```

### `--path` behavior

`--path` follows the `--relative-to` mode.

Default:

```bash
gitsub --path
```

Same as:

```bash
gitsub --path --relative-to parent
```

Other modes:

```bash
gitsub --path --relative-to outermost
gitsub --path --relative-to current
```

With `current`, the output is:

```text
.
```

### Explicit path flags

Immediate-parent-relative path:

```bash
gitsub --path-parent
```

Outermost-relative path:

```bash
gitsub --path-outermost
```

Short alias:

```bash
gitsub -O
```

---

## Parent and submodule path fields

These flags expose the exact pieces from your nested layout.

### Parent dirname

```bash
gitsub --parent-dirname
```

Example output:

```text
~
```

### Parent basename

```bash
gitsub --parent-basename
```

Example output:

```text
parent_root
```

### Submodule prefix

Path from the outermost parent repo root to, but not including, the current submodule basename.

```bash
gitsub --submodule-prefix
```

Example output:

```text
parent_prefix/path
```

### Submodule basename

```bash
gitsub --submodule-basename
```

Example output:

```text
submodule_repo_toplevel
```

### Current working directory prefix inside the submodule

```bash
gitsub --cwd-prefix
```

Example output:

```text
subprefix/path/current
```

---

## Format strings

`gitsub` supports custom formatted output using `--format`.

Example:

```bash
gitsub --format '%D / %M / %G / %B / %W'
```

Possible output:

```text
~ / parent_root / parent_prefix/path / submodule_repo_toplevel / subprefix/path/current
```

### Format tokens

| Token | Meaning |
|---|---|
| `%P` | Immediate parent repository root |
| `%O` | Outermost parent repository root |
| `%S` | Path relative to the reference selected by `--relative-to` |
| `%T` | Path relative to the outermost parent repository |
| `%C` | Current submodule repository root |
| `%D` | Dirname of outermost parent repository root |
| `%M` | Basename of outermost parent repository root |
| `%G` | Path from outermost parent root to, but not including, the submodule root basename |
| `%B` | Basename of current submodule repository root |
| `%W` | Path from current submodule root to `$PWD` |
| `%%` | Literal percent sign |

Example:

```bash
gitsub -f 'parent=%O dirname=%D base=%M prefix=%G submodule=%B cwd=%W'
```

---

## Examples

Basic detection:

```bash
gitsub
```

Boolean check:

```bash
if gitsub --exists-only; then
  echo "yes"
fi
```

Explicit status text:

```bash
gitsub --status
```

Print current repo root:

```bash
gitsub --current
```

Print immediate parent:

```bash
gitsub --parent
```

Print outermost parent:

```bash
gitsub --outermost
```

Print path relative to immediate parent:

```bash
gitsub --path
```

Print path relative to outermost parent:

```bash
gitsub --path-outermost
```

Print parent dirname and basename:

```bash
gitsub --parent-dirname
gitsub --parent-basename
```

Print submodule prefix and basename:

```bash
gitsub --submodule-prefix
gitsub --submodule-basename
```

Print current working directory prefix inside submodule:

```bash
gitsub --cwd-prefix
```

Custom format:

```bash
gitsub -f '%D / %M / %G / %B / %W'
```

JSON with outermost-relative selection:

```bash
gitsub --json --relative-to outermost
```

Generate Bash completion:

```bash
gitsub --completion bash
```

Generate Zsh completion:

```bash
gitsub --completion zsh
```

---

## Shell completions

`gitsub` can generate its own completion files.

### Bash

```bash
mkdir -p ~/.local/share/bash-completion/completions
gitsub --completion bash > ~/.local/share/bash-completion/completions/gitsub
```

If needed, source it from `~/.bashrc`:

```bash
source ~/.local/share/bash-completion/completions/gitsub
```

### Zsh

```bash
mkdir -p ~/.local/share/zsh/site-functions
gitsub --completion zsh > ~/.local/share/zsh/site-functions/_gitsub
```

Make sure your `~/.zshrc` includes this before `compinit`:

```zsh
fpath=(~/.local/share/zsh/site-functions $fpath)
autoload -Uz compinit
compinit
```

---

## Options

| Option | Description |
|---|---|
| `-q`, `--quiet` | Print nothing; use exit status only |
| `-e`, `--exists-only` | Same as `--quiet`; useful for scripting |
| `--status` | Print `submodule` or `not-submodule` |
| `-r`, `--raw` | Print raw absolute paths instead of replacing `$HOME` with `~` |
| `-p`, `--parent` | Print the immediate parent repository root |
| `-o`, `--outermost`, `--toplevel` | Print the outermost parent repository root |
| `-s`, `--path` | Print path according to `--relative-to` |
| `--path-parent` | Print path relative to the immediate parent repository |
| `-O`, `--path-outermost` | Print path relative to the outermost parent repository |
| `--relative-to parent\|outermost\|current` | Select how `--path` and `%S` are resolved |
| `-c`, `--current` | Print the current submodule repository root |
| `--parent-dirname` | Print dirname of outermost parent repository root |
| `--parent-basename` | Print basename of outermost parent repository root |
| `--submodule-prefix` | Print path to but not including submodule basename |
| `--submodule-basename` | Print basename of current submodule repository root |
| `--cwd-prefix` | Print path from submodule root to current directory |
| `-j`, `--json` | Print JSON output |
| `-f`, `--format FORMAT` | Print custom formatted output |
| `--completion bash\|zsh` | Print shell completion script |
| `-h`, `--help` | Show help text |

---

## How it works

`gitsub` walks upward from the current repo root and checks parent repos on the same path.

It detects submodule relationships using:

- `.gitmodules`
- Git gitlink entries (`160000`) as a fallback

That makes it useful even when `.gitmodules` alone is not enough.

---

## Notes

- `gitsub` only checks upward along the current filesystem path
- it is intended for repos physically nested beneath parent repos
- it does not try to discover unrelated repos elsewhere on disk
- it is designed to stay lightweight and dependency-minimal

---

## License

MIT
