```markdown
# Special Filenames

## Related Levels
- **Level 1** → Accessing a file named `-`
- **Level 2** → Accessing filenames that contain spaces
- **Level 3** → Finding and reading hidden files inside `inhere`

## Core Idea
Not every filename can be used directly in the shell.  
Some names have special meaning to commands, while others need quoting or escaping to be treated as a single argument.  
Understanding these cases prevents common errors when working with unusual filenames in real environments.

## Key Commands & Patterns

### 1. File named `-`
Many commands interpret a standalone `-` as standard input or output instead of a filename.

```bash
cat ./-
```

`./-` forces the command to treat `-` as a filename in the current directory.

### 2. Filenames containing spaces
The shell splits arguments on unquoted spaces by default.

```bash
cat "spaces in this filename"
# or
cat spaces\ in\ this\ filename
```

- `"..."` keeps the entire filename as one argument.
- `\` escapes a space so it is not treated as an argument separator.

### 3. Hidden files
Filenames starting with `.` are hidden by convention and are not shown by a normal `ls`.

```bash
ls -a          # show all entries, including hidden ones
ls -la         # long listing + hidden entries
```

- `-a` includes hidden entries.
- `-l` shows detailed file information.

## How They Work Together

A typical directory inspection sequence:

```bash
ls -la
cat ./-
cat "file with spaces"
```

This allows you to:
- Inspect all directory entries, including hidden files
- Read a file whose name is a single dash
- Read a file whose name contains spaces

## Important Notes
- A leading dot hides a file by convention; it does not protect or encrypt it.
- `./-` makes the filename unambiguous for commands that treat a standalone `-` specially.
- Quoting protects spaces and other shell-sensitive characters from being interpreted by the shell.
- `ls -a` also shows `.` (current directory) and `..` (parent directory).
- The meaning of `-` depends on the command; not every tool treats it as standard input or output.

## Related Concepts
- **File Discovery** → Tools like `find` locate files based on their properties.
```
