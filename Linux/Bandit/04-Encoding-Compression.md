# Encoding & Compression

## Related Levels

- **Level 10&#x20;**&#x20;→ Decoding Base64-encoded data to recover its original content
- **Level 11** → Applying the ROT13 substitution cipher using `tr` for character translation
- **Level 12** → Identifying and reversing multiple layers of encoding, compression, and archiving using `file`, `xxd`, and related tools

## Core Idea

Encoding, substitution ciphers, compression, and archiving are different ways of transforming or organizing data:

- **Encoding** (e.g., Base64) converts data into a different representation for storage or transmission. It is reversible and does not provide secrecy.
- **Substitution Cipher** (e.g., ROT13) replaces characters according to a fixed rule. It provides simple obfuscation, not security.
- **Compression** (e.g., gzip, bzip2) reduces data size by representing information more efficiently. It is reversible and preserves the original information.
- **Archiving** (e.g., tar) combines multiple files into a single archive without necessarily compressing them.

In security analysis, identifying the transformation applied to data is essential. The `file` command and hexadecimal inspection with `xxd` help identify file formats, especially when extensions are missing or misleading.

## Key Commands & Patterns

### 1. Base64 Decoding

```bash
base64 -d filename
```

- `base64` → Encodes binary data into a text-based representation or decodes Base64 data back into its original form.
- `-d` → Decode: converts Base64-encoded input back into its original data.

### 2. Character Translation with tr (ROT13)

```bash
tr 'A-Za-z' 'N-ZA-Mn-za-m' < filename
```

- `tr` → Translates or deletes characters from standard input.
- `'A-Za-z'` → Input character set containing uppercase and lowercase letters.
- `'N-ZA-Mn-za-m'` → Output character set defining the ROT13 mapping, where A↔N, B↔O, and so on.
- `< filename` → Redirects file content to standard input because `tr` does not accept filenames directly.

### 3. Hex Dump and Reverse with xxd

```bash
xxd filename
xxd -r hexdump.txt > restored.bin
```

- `xxd` → Creates a hexadecimal dump of a file or converts a hexadecimal dump back into binary data.
- `-r` → Revert: reconstructs binary data from a hexadecimal dump.
- In Bandit Level 12, `xxd` helps inspect file headers and identify magic bytes.

### 4. File Identification and Decompression

```bash
file filename
gzip -d file.gz
bzip2 -d file.bz2
tar -xf archive.tar
```

- `file` → Identifies a file's type by examining its contents and signatures.
- `gzip -d` → Decompresses gzip-compressed data.
- `bzip2 -d` → Decompresses bzip2-compressed data.
- `tar -xf` → Extracts files from a tar archive. `-x` means extract and `-f` specifies the archive file.

## How They Work Together

Bandit Level 12 demonstrates an iterative workflow for analyzing data that has undergone multiple transformations:

1. **Identify:** Use `file` to determine the current file format. If the result is unclear, inspect its hexadecimal representation with `xxd` to look for magic bytes.
2. **Reverse:** Apply the appropriate decoding, decompression, or extraction tool based on the identified format.
3. **Re-identify:** Run `file` on the resulting output to determine its new format.
4. **Repeat:** Continue until the final readable content is recovered.

This approach is also useful in malware analysis and digital forensics, where data may be nested inside multiple archives or transformed through different methods.

Levels 10 and 11 introduce individual transformations, while Level 12 demonstrates how to identify and reverse them as part of a larger workflow.

## Important Notes

- Base64 is encoding, not encryption. It provides no confidentiality and can be reversed without a secret key.
- ROT13 is a symmetric substitution cipher with no secret key. Applying it twice restores the original text.
- `tr` is a general-purpose character translation and deletion tool. ROT13 is just one specific character mapping it can perform.
- Magic bytes are file signatures found in file contents. Examples include `1f 8b` (gzip), `42 5a 68` (bzip2 / BZh), and `7f 45 4c 46` (ELF binary).
- `tar` is an archiving tool, not a compression tool. Compression requires an additional tool such as gzip or bzip2.
- Always run `file` again after each decoding or decompression step. The output may contain another transformation layer.

## Related Concepts

- [File Discovery](./02-File-Discovery.md) — Identifying file types and locating files before further analysis.
- [File Content Analysis](./03-File-Content-Analysis.md) — Using tools such as `strings` to extract readable content for further inspection.

