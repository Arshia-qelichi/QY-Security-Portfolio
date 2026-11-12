# File Content Analysis

## Related Levels
- **Level 7** → Searching for a specific string within file contents using `grep`
- **Level 8** → Sorting file contents and identifying unique lines using `sort` and `uniq`
- **Level 9** → Extracting human-readable strings from binary data using `strings`
- **Level 17** → Comparing two files and identifying differences to locate hidden data using `diff`

## Core Idea
File content analysis involves examining data stored inside files to locate specific information, identify patterns, extract readable text, and detect differences.

Linux provides specialized tools for each aspect: `grep` for pattern matching, `sort` and `uniq` for organizing and identifying repeated data, `strings` for extracting printable character sequences, and `diff` for comparing files. Combining these tools helps reveal useful information in large, unfamiliar, or binary files.

## Key Commands & Patterns

### 1. Searching Text with grep

```bash
grep 'pattern' filename
```

- `grep` → Searches file contents for lines matching a specified pattern using Basic Regular Expressions (BRE) by default.
- `'pattern'` → The text or regular expression to search for. Single quotes prevent shell expansion of the pattern.
- `filename` → The target file to search.

### 2. Sorting and Identifying Unique Lines

```bash
sort filename | uniq -u
```

- `sort` → Sorts lines lexicographically by default, grouping identical lines together.
- `|` → Pipe operator; passes the standard output of `sort` to the standard input of `uniq`.
- `uniq -u` → Unique: prints only lines that appear exactly once in the input.

### 3. Extracting Readable Strings from Binary Data

```bash
strings filename
```

- `strings` → Extracts sequences of printable characters from a file, including binary files. GNU `strings` uses a minimum length of four characters by default.
- `filename` → The file to inspect.

### 4. Comparing Files with diff

```bash
diff file1 file2
```

- `diff` → Compares two files line by line and reports their differences in normal diff format.
- `file1` → The first file used as the comparison reference.
- `file2` → The second file being compared against the reference.

## How They Work Together

These tools provide complementary approaches to file content analysis:

1. **Search:** Use `grep` to locate specific patterns or strings within text files.
2. **Frequency Analysis:** Combine `sort` and `uniq` to organize data and identify lines that appear only once.
3. **String Extraction:** Use `strings` to extract printable character sequences from binary or mixed-content files before further inspection.
4. **Comparison:** Use `diff` to identify changes between similar files. In Bandit Level 17, the password can be found among the differing lines of two nearly identical files.

The key idea is that content analysis depends on the nature of the data. Different tools help search, organize, extract, or compare information without manually inspecting entire files.

## Important Notes
- `grep` returns entire lines containing the matching pattern, not just the matched substring. It is case-sensitive by default; use `-i` (Ignore Case) for case-insensitive searches.
- `uniq` only compares adjacent lines. Sorting the input first ensures identical lines are grouped together.
- Without options, `uniq` outputs one copy of each group of adjacent identical lines. `-u` Unique: prints only lines that occur exactly once in each group of adjacent identical lines while `-d` (Repeated) displays one copy of each line that occurs more than once.
- `strings` extracts printable character sequences; it does not decode encoded data or reconstruct the original binary structure.
- `diff` output describes the changes needed to transform `file1` into `file2`. In normal diff format, lines prefixed with `<` belong to `file1`, while lines prefixed with `>` belong to `file2`.

## Related Concepts
- [File Discovery](./02-File-Discovery.md) — Locating files before examining their contents.
- [Encoding & Compression](./04-Encoding-Compression.md) — Processing encoded or compressed data before examining its contents.
