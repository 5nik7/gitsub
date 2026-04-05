# gitsub

[![shell](https://img.shields.io/badge/shell-bash-121011.svg?logo=gnu-bash)](#)
[![platform](https://img.shields.io/badge/platform-linux%20%7C%20termux%20%7C%20macOS-blue)](#)
[![git](https://img.shields.io/badge/tool-git-orange?logo=git)](#)
[![license](https://img.shields.io/badge/license-MIT-green)](#)

`gitsub` is a lightweight Bash utility for detecting whether the current Git repository is a submodule of one or more parent Git repositories higher on the same filesystem path.

It walks the full upward path, discovers every Git repository root on that path, and verifies adjacent parent/child submodule relationships. That makes nested submodule chains work correctly.

## Features

- Detect whether the inspected repo is part of a verified submodule chain
- Accept an optional path argument and default to the current working directory
- Print the immediate parent and outermost parent repository roots
- Print the verified repo chain and hop paths
- Output the chain as a shell array literal
- Print relative path views and parent/submodule path fields
- Output JSON or custom formatted strings
- Generate Bash and Zsh completions from the script itself

## Installation

Save the script as `gitsub`, then make it executable and move it somewhere in your `PATH`.

```bash
chmod +x gitsub
mv gitsub ~/.local/bin/
```

Make sure `~/.local/bin` is in your `PATH`.

## Usage

```bash
gitsub [options] [path]
```

If `path` is omitted, `gitsub` inspects the current working directory.

Exit codes:

- `0` — the inspected repo is part of a verified submodule chain
- `1` — the inspected repo is not part of a verified submodule chain
- `2` — invalid usage or the inspected path is not inside a Git repository

## Quick examples

Basic check:

```bash
gitsub
```

Inspect another path:

```bash
gitsub ~/parent_root/parent_prefix/path/submodule_repo_toplevel/subprefix/path/current
```

Status mode:

```bash
gitsub --status
```

Immediate parent and outermost parent:

```bash
gitsub --parent
gitsub --outermost
```

Chain and hop paths:

```bash
gitsub --chain
gitsub --chain-paths
```

Array output:

```bash
gitsub --array
```

Set the array in Bash:

```bash
eval "$(gitsub --array)"
printf '%s\n' "${chain[@]}"
```

Safer no-`eval` Bash alternative:

```bash
mapfile -t chain < <(gitsub --chain)
```

Zsh alternative:

```zsh
chain=("${(@f)$(gitsub --chain)}")
```

## Optional path behavior

These are equivalent when you are already in the target directory:

```bash
gitsub
gitsub .
```

You can inspect any path inside the submodule:

```bash
gitsub --cwd-prefix ~/parent_root/parent_prefix/path/submodule_repo_toplevel/subprefix/path/current
```

The inspected path is also reflected in JSON output as `inspected_path`.

## Path and field example

Given:

```text
~/parent_root/parent_prefix/path/submodule_repo_toplevel/subprefix/path/current
```

then:

```bash
gitsub -f '%D / %M / %G / %B / %W'
```

returns:

```text
~ / parent_root / parent_prefix/path / submodule_repo_toplevel / subprefix/path/current
```

Meaning:

- `%D` — dirname of outermost parent repo root
- `%M` — basename of outermost parent repo root
- `%G` — path to, but not including, the submodule basename
- `%B` — basename of the current submodule repo root
- `%W` — path from current submodule root to the inspected path

## Output modes

Plain mode:

```bash
gitsub
```

Quiet / scripting mode:

```bash
gitsub --quiet
gitsub --exists-only
```

Status mode:

```bash
gitsub --status
```

JSON mode:

```bash
gitsub --json
```

Custom formatting:

```bash
gitsub -f '%O | %G | %B | %W'
```

## Chain output

Verified chain roots:

```bash
gitsub --chain
```

Verified hop paths:

```bash
gitsub --chain-paths
```

Shell array literal:

```bash
gitsub --array
```

Example output:

```bash
chain=(
  "~/mainrepo"
  "~/mainrepo/vendor/libA"
  "~/mainrepo/vendor/libA/ext/libB"
)
```

## Format tokens

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
| `%W` | Path from current submodule root to the inspected path |
| `%L` | Verified repo chain length |
| `%%` | Literal percent sign |

## Options

| Option | Description |
|---|---|
| `-q`, `--quiet` | Print nothing; use exit status only |
| `-e`, `--exists-only` | Same as `--quiet`; useful for scripting |
| `--status` | Print `submodule` or `not-submodule` |
| `-r`, `--raw` | Print raw absolute paths instead of replacing `$HOME` with `~` |
| `-p`, `--parent` | Print the immediate parent repository root |
| `-o`, `--outermost`, `--toplevel` | Print the outermost parent repository root |
| `--chain` | Print the verified repository chain |
| `-a`, `--array` | Print the verified repository chain as a shell array literal |
| `--chain-paths` | Print the verified relative path for each chain hop |
| `-s`, `--path` | Print path according to `--relative-to` |
| `--path-parent` | Print path relative to the immediate parent repository |
| `-O`, `--path-outermost` | Print path relative to the outermost parent repository |
| `--relative-to parent\|outermost\|current` | Select how `--path` and `%S` are resolved |
| `-c`, `--current` | Print the current submodule repository root |
| `--parent-dirname` | Print dirname of outermost parent repository root |
| `--parent-basename` | Print basename of outermost parent repository root |
| `--submodule-prefix` | Print path to but not including submodule basename |
| `--submodule-basename` | Print basename of current submodule repository root |
| `--cwd-prefix` | Print path from submodule root to the inspected path |
| `-j`, `--json` | Print JSON output |
| `-f`, `--format FORMAT` | Print custom formatted output |
| `--completion bash\|zsh` | Print shell completion script |
| `-h`, `--help` | Show help text |

## Completions

Generate Bash completion:

```bash
mkdir -p ~/.local/share/bash-completion/completions
gitsub --completion bash > ~/.local/share/bash-completion/completions/gitsub
```

Generate Zsh completion:

```bash
mkdir -p ~/.local/share/zsh/site-functions
gitsub --completion zsh > ~/.local/share/zsh/site-functions/_gitsub
```

If needed, make sure your Zsh `fpath` includes that directory before `compinit`:

```zsh
fpath=(~/.local/share/zsh/site-functions $fpath)
autoload -Uz compinit
compinit
```

## Notes

- `gitsub` only checks upward along the current filesystem path
- it is intended for repos physically nested beneath parent repos
- it uses `.gitmodules` and Git gitlink entries (`160000`) to verify submodule edges
- array output is meant for Bash/Zsh-style shells

## License

MIT
