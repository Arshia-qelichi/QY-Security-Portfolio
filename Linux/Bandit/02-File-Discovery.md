# File Discovery

## Related Levels
- **Level 4** → Identifying human-readable files among binary data using `file`
- **Level 5** → Finding files by type, exact size, and non-executable status
- **Level 6** → System-wide search by owner, group, and size with error suppression

## Core Idea
File discovery in Linux involves two distinct skills: identifying what a file is (content/type analysis) and locating files based on their properties (metadata filtering).

The `file` command identifies file types by examining their contents, while `find` locates files using conditions such as type, size, ownership, and permissions.

## Key Commands & Patterns

### 1. Content Identification

```bash
file ./*
```

- `file` → Identifies file types by examining content signatures, not just filename extensions.
- `./*` → Matches non-hidden entries in the current directory. Prefixing the pattern with `./` prevents filenames beginning with `-` from being interpreted as command-line options.
- This pattern is non-recursive; it does not search inside subdirectories.

### 2. Metadata Filtering with find

```bash
find . -type f -size 1033c ! -executable
```

- `-type f` → Type: Regular File; matches regular files, excluding directories and symbolic links.
- `-size 1033c` → Size filter; `c` specifies bytes. The file must be exactly 1033 bytes.
- `! -executable` → Logical NOT + Executable test; matches files the current user cannot execute. The test uses `access()` rather than checking permission bits alone.
- Implicit AND → Adjacent conditions are combined using logical AND unless `-o` (OR) is specified.

### 3. System-Wide Search with Error Handling

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```

- `/` → Root directory; starts the search from the filesystem root.
- `-user bandit7` → Owner: matches files owned by this username or corresponding UID.
- `-group bandit6` → Group: matches files belonging to this group name or corresponding GID.
- `-size 33c` → Matches files exactly 33 bytes in size.
- `2>/dev/null` → Redirects Standard Error (stderr, file descriptor 2) to `/dev/null`, suppressing error messages while keeping standard output visible.

## How They Work Together

The Bandit levels demonstrate two complementary approaches to file discovery:

- **Level 4:** Identify files by examining their content and determining which ones contain human-readable data.
- **Level 5:** Narrow down candidates using metadata conditions instead of inspecting every file manually.
- **Level 6:** Apply multiple ownership and size conditions across the filesystem.

An important distinction is that file accessibility and content type are different concepts. For example, `-readable` checks whether the current user can read a file; it does not determine whether its contents are human-readable. A readable file may still contain binary data.

## Important Notes

- `-executable` differs from `-perm -u+x`: the former checks whether the current user can execute a file, while the latter checks whether the owner's execute permission bit is set.
- `./*` does not match hidden files (names beginning with `.`) under the default shell glob behavior.
- For recursive file type identification, combine `find` with `file`:

```bash
find . -type f -exec file -- {} +
```

- `-size` supports comparison prefixes:
  - `+100c` → Greater than 100 bytes.
  - `-100c` → Less than 100 bytes.
  - `100c` → Exactly 100 bytes.
- Without a size suffix, GNU `find` uses 512-byte units and rounds file sizes up to the next unit.
- Suppressing stderr does not guarantee that every directory was successfully searched; inaccessible directories may be skipped.

## Related Concepts

- [Special Filenames](./01-Special-Filenames.md) — Understanding how shell expansion and special filenames affect file access.
- [File Content Analysis](./03-File-Content-Analysis.md) — The next step after discovery: examining and comparing file contents using `grep`, `strings`, and `diff`.
