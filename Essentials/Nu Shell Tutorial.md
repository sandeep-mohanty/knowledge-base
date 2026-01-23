# Comprehensive Nushell Tutorial

## Introduction to Nushell

Nushell (nu) reimagines shell scripting with structured data pipelines, treating command outputs as tables or lists rather than text streams. Written in Rust, it offers cross-platform support, rich type awareness, and modern programming features while maintaining Unix pipe philosophy. Unlike bash or zsh, nu parses outputs into typed values like strings, ints, filesizes, enabling powerful filtering, sorting, and transformations.

This tutorial covers installation, basics, advanced data handling, scripting, configuration, and best practices with extensive examples.

## Installation

Install the latest Nushell (version 0.103+ as of 2026) via package managers or binaries.

### Linux (Ubuntu/Debian)
```bash
curl -fsSL https://apt.fury.io/nushell/gpg.key | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/fury-nushell.gpg
echo "deb https://apt.fury.io/nushell/ /" | sudo tee /etc/apt/sources.list.d/fury.list
sudo apt update
sudo apt install nushell
```
Verify: `nu --version`

### macOS (via Homebrew)
```bash
brew install nushell
```

### Windows (Winget)
```powershell
winget install Nushell
```

### Cargo (Rust users)
```bash
cargo install nushell
```

Launch with `nu`. Exit with `exit`.

## Quick Tour: Structured Data Pipelines

Nushell commands output *tables* of structured data, not plain text.

### Listing Files
```nu
ls
```
Output:
```
╭────┬──────────────┬──────┬──────────┬──────────────╮
│ #  │ name         │ type │ size     │ modified     │
├────┼──────────────┼──────┼──────────┼──────────────┤
│ 0  │ file1.txt    │ file │ 1.2 KiB  │ 1 day ago    │
│ 1  │ dir1         │ dir  │   4.0 KiB │ 2 days ago   │
╰────┴──────────────┴──────┴──────────┴──────────────╯
```
Columns have types (e.g., `size` is `filesize`).[1]

### Sorting and Filtering
Sort largest files first:
```nu
ls | sort-by size | reverse
```

Filter large files:
```nu
ls | where size > 10kb
```

Running processes:
```nu
ps | where status == Running
```

## Basic Commands and Navigation

- `pwd`: Print working directory
- `ls <path>`: List contents (`ls -f` for full info)
- `cd <path>`: Change directory
- `mkdir <name>`: Create directory
- `touch <file>`: Create empty file
- `cat <file>`: View file content (structured if possible)

History: Up/Down arrows or `history`.

Tab completion: Partial commands/files auto-complete.

### Help System
```nu
help ls          # Full help
ls --help        # Short help
help --find size # Search commands
help commands    # All commands table
help commands | explore  # Interactive explorer
```
`explore` lets you drill into nested tables with Enter/Esc.

## Data Manipulation

Nu excels at table operations.

### Select and Get Columns
```nu
ls | select name size type  # Specific columns
ls | get name.0            # Cell path (first name)
```

### Filter with Where
```nu
ls | where type == dir and size > 1kb
ps | where cpu > 0.1
```

### Aggregate
```nu
ls | math sum size      # Total size
ls | group-by type | each { |it| $it | math sum size }
```

### Joins and Updates
```nu
ls | upsert newcol (ls target | get name)
```

### Convert Formats
```nu
ls | to csv    # CSV output
ls | to json   # JSON
ls | to md     # Markdown table
```

## Variables and Types

Variables start with `$`.

```nu
let x = 42              # Immutable int
mut y = [1,2,3]         # Mutable list
const PI = 3.14         # Constant

$x = "hello"            # Strings
$env.PWD                # Environment vars
```

Types: int, float, string, bool, list<>, table<cols>, filesize, duration, etc. Use `describe` to check.

Lists:
```nu
[1,2,3] | math sum
["a","b"] | each { |it| $it + "!" }
```

Records (tables as values):
```nu
{ name: "Alice", age: 30 }
```

## Control Flow

### Loops
```nu
for i in 1..5 {
  print $"Number: ($i)"
}

repeat 3 { print "Hi" }
```

### Conditionals
```nu
if $x > 10 {
  print "Big"
} else {
  print "Small"
}

match $status {
  "Running" => { "Active" }
  _ => { "Idle" }
}
```

## Custom Commands and Modules

Define commands:
```nu
def greet [name] {
  print $"Hello ($name)!"
}
greet Bob
```

With params/types:
```nu
def add [a: int, b: int] {
  $a + $b
}
```

Modules: Save in `.nu` files.
```nu
# mymodule.nu
export def greet [] { print "Hi from module!" }
use mymodule.nu *
```

## Scripting

Shebang: `#!/usr/bin/env nu`

Example script `backup.nu`:
```nu
#!/usr/bin/env nu

def main [dir: string] {
  let timestamp = (date now | format date '%Y%m%d')
  mkdir $"backup_($timestamp)"
  ls $dir | where type == file | each { |it|
    cp $it.name $"backup_($timestamp)/"
  }
  print "Backup complete!"
}
```
Run: `chmod +x backup.nu; ./backup.nu docs`

Error handling:
```nu
try {
  open missing.txt
} catch {
  print "File not found: $err.message"
}
```

## Configuration

Config at `~/.config/nushell/config.nu`:
```nu
$env.config = {
  color: { primary: "#00ff00" }
  key_bindings: [
    { name: ctrl_c, mods: [], chars: [\u{03}] }
  ]
  table: { mode: compact }
}
```

Custom prompts in `env.nu`.

## Plugins

Extend with Rust plugins (e.g., `nu_plugin_formats`).

Install: `register plugin.wasm`

Example: `inc` for incrementing versions.

## Advanced Topics

### Parallelism
```nu
ls *.log | par-each { |it| wc $it.name } | flatten
```

### Custom Errors
```nu
error make {msg: "Custom error"}
```

### External Commands
```nu
^git status  # Pipe-friendly external
```

### Working with JSON/CSV
```nu
open data.json | from json | where age > 25 | to csv | save output.csv
```

## Common Workflows

Disk usage:
```nu
ls $HOME | sort-by size --reverse | each { |it| $it | upsert pct (($it.size / (ls $HOME | math sum size).0) * 100) }
```

Kill processes:
```nu
ps | where name == node | get pid | each { |it| kill -9 $it }
```

## Best Practices

- Pipe everything: Leverage structured data.
- Use `help commands | explore` to discover.
- Script idempotently with checks.
- Modularize with defs/modules.
- Test pipelines incrementally.

## Resources

- Official Book: https://www.nushell.sh/book/
- GitHub: https://github.com/nushell/nushell
- Discord: nushell.sh community
- nu_scripts: Shared scripts repo

Practice in `nu` REPL. Experiment with `ls | explore`!
