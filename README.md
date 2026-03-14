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
~/src/mainrepo
~/src/mainrepo/vendor/libA
~/src/mainrepo/vendor/libA/ext/libB
```

Assume:

- `mainrepo` tracks `vendor/libA` as a submodule
- `libA` tracks `ext/libB` as a submodule

If your current directory is anywhere inside:

```text
~/src/mainrepo/vendor/libA/ext/libB
```

Then:

```bash
gitsub
```

returns:

```text
yes
```

```bash
gitsub --parent
```

returns:

```text
~/src/mainrepo/vendor/libA
```

```bash
gitsub --outermost
```

returns:

```text
~/src/mainrepo
```

```bash
gitsub --path
```

returns:

```text
ext/libB
```

```bash
gitsub --path-outermost
```

returns:

```text
vendor/libA/ext/libB
```

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
  "current": "~/src/mainrepo/vendor/libA/ext/libB",
  "parent": "~/src/mainrepo/vendor/libA",
  "outermost": "~/src/mainrepo",
  "path": "ext/libB",
  "path_parent": "ext/libB",
  "path_outermost": "vendor/libA/ext/libB",
  "relative_to": "parent",
  "name": "libB"
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

## Format strings

`gitsub` supports custom formatted output using `--format`.

Example:

```bash
gitsub --format '%N: %S -> %P'
```

Possible output:

```text
libB: ext/libB -> ~/src/mainrepo/vendor/libA
```

Using outermost-relative mode:

```bash
gitsub --relative-to outermost --format '%N: %S -> %O'
```

### Format tokens

| Token | Meaning |
|---|---|
| `%P` | Immediate parent repository root |
| `%O` | Outermost parent repository root |
| `%S` | Path relative to the reference selected by `--relative-to` |
| `%T` | Path relative to the outermost parent repository |
| `%C` | Current repository root |
| `%N` | Basename of the current repository root |
| `%%` | Literal percent sign |

Example:

```bash
gitsub -f 'name=%N parent=%P outermost=%O path=%S outer=%T'
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

Print current repo name:

```bash
gitsub --name
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

Select `--path` behavior:

```bash
gitsub --path --relative-to parent
gitsub --path --relative-to outermost
gitsub --path --relative-to current
```

Custom format:

```bash
gitsub -f '%N | %S | %P'
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
| `-c`, `--current` | Print the current repository root |
| `-n`, `--name` | Print the basename of the current repository root |
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
