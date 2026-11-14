# Git Concepts & Techniques

## Related Levels

- **Level 27** → Cloning a remote repository over SSH and inspecting its contents
- **Level 28** → Recovering removed information from commit history
- **Level 29** → Finding data in a non-default branch
- **Level 30** → Inspecting a tag that references an object outside the working tree
- **Level 31** → Staging an ignored file, committing, and pushing to a remote repository

## Core Idea

A Git repository stores more than the current version of a project. Commits, branches, and tags provide references to objects in Git's object database, including content that may no longer appear in the current working tree.

For security work, this matters because deleting a secret from the latest revision does not necessarily remove it from commit history, another branch, or a tagged object. Auditing a repository means inspecting these references, not just its visible files.

## Key Commands & Patterns

### 1. Cloning Over SSH

```bash
git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo
cd repo
```

- `ssh://user@host:port/path` → Git transport URL using SSH authentication.
- These levels are solved **from your own machine**, not from inside the Bandit server. Git must therefore be installed locally.
- `git clone` copies the repository and its reachable history and objects, and registers the source remote as `origin`.

### 2. Inspecting Commit History

```bash
git log
git log -p
git show <commit>
```

- `git log` → Lists commits with their author, date, and message; it does not display file contents.
- `-p` (`--patch`) → Also displays the changes introduced by commits.
- `git show <commit>` → Displays information and changes associated with a specific commit.
- In Level 28, the current README had the password replaced. The previous commit's diff still contained the original value, so `git log -p` revealed it while plain `git log` did not.

### 3. Exploring Branches

```bash
git branch -a
git checkout dev
```

- `git branch` → Lists local branches; the current branch is marked with `*`.
- `-a` (`--all`) → Lists both local and remote-tracking branches.
- `origin/dev` → A remote-tracking branch: a local reference to the `dev` branch on the remote named `origin`.
- `git checkout dev` → If no local `dev` branch exists and Git finds a unique matching remote-tracking branch, it creates a local branch and sets up tracking automatically.
- `git switch dev` → Modern alternative for switching branches.
- In Level 29, the default branch indicated that passwords were not available in the production branch. The `dev` branch contained the required information.

### 4. Inspecting Tags

```bash
git tag
git show secret
```

- `git tag` → Lists tags in the repository.
- `git show <tag>` → Displays the tag and the object it references.
- A tag is a named reference. Lightweight tags point directly to an object, while annotated tags are separate Git objects containing metadata and a message.
- In Level 30, the `secret` tag points to a **blob**, not a commit. Therefore, `git show secret` directly displays content that does not exist as a file in the current working tree.

### 5. Staging, Committing, and Pushing

```bash
cat .gitignore
echo 'May I come in?' > key.txt
git add -f key.txt
git commit -m "Add key.txt"
git push origin master
```

- `.gitignore` → Defines patterns for files Git should normally leave untracked. In this level, the pattern is `*.txt`.
- `git add` → Stages changes for the next commit. Ignored files are normally excluded.
- `-f` (`--force`) → Allows adding a file despite matching ignore rules.
- `git commit` → Records the staged changes in the local repository.
- `-m` (`--message`) → Supplies the commit message directly instead of opening an editor.
- `git push origin master` → Sends local commits to the `master` branch on the remote named `origin`.
- These are three distinct operations: staging selects changes, committing records them locally, and pushing transfers them to the remote repository.

## How They Work Together

Each level exposes a different place where repository content can exist:

1. **Working tree (27):** The files currently checked out in the repository.
2. **History (28):** Content recorded in earlier commits, accessible through commit diffs.
3. **Other branches (29):** Separate development lines that may contain different file versions.
4. **Tags (30):** Named references that can point to objects outside the files currently checked out.
5. **Remote interaction (31):** Changes submitted to the server, where server-side hooks can process them.

The progression moves from inspecting visible content to examining retained repository data and finally submitting changes to a remote repository.

## Important Notes

- Removing a secret from the latest version does not remove it from earlier commits. Proper removal may require rewriting history and coordinating the change with repository users.
- A branch normally moves as new commits are added; a tag is intended to remain attached to a specific object.
- A tag can reference a blob, tree, or commit; it does not necessarily point to a commit.
- `.gitignore` prevents accidental tracking of untracked files. It does not stop tracking files already in the index and is not a security control.
- `git add -f` overrides ignore rules for the specified path.
- A commit remains local until it is pushed to a remote repository.
- Remote repositories can run server-side hooks when receiving a push. In Level 31, the server processes the submitted content and returns a response.
- Credentials should never be committed, even to private repositories, because repository history can preserve them and clones may carry that history.

## Related Concepts

- [SSH Authentication](./05-SSH-Authentication.md) — Git over SSH uses the SSH transport and authentication model.
- [File Discovery](./02-File-Discovery.md) — Inspecting hidden entries such as `.git` and `.gitignore`.
- [File Content Analysis](./03-File-Content-Analysis.md) — Searching and comparing recovered file contents.
