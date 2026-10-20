# Git Internals: Content-Addressable Storage :3

Git is not merely a version control system; it is a **Content-Addressable Storage** system. Instead of tracking changes to files, Git saves **snapshots** of the entire project. The key to this is the hash: the content determines the address (ID), not the filename.

**Index:**

- [The Heart of Git ](#1-the-heart-of-git-the-git-directory)
- [Storage Optimization](#2-storage-optimization-object-sharding)
- [Conventional Commits](#conventional-commits)
- [The local lifecycle of Workflow](#3-the-local-lifecycle-workflow)
- [Automation with Hooks](#4-automation-with-hooks)
- [Technical Plumbing Commands](#5-technical-plumbing-commands)
- [Configure SSH_KEY in your Linux](SSH_KEY.md)
- [Github Actions](GITHUB_ACTIONS.md)

### Conventional Commits

- feat: feat(web): add project social links.
- fix: fix(workflow): fixed permission of writing in gh-pages.
- docs: docs(web): update menu nav.
- style: style(frontend): adjust color of the profile.
- refactor: refactor(oop): improve the cicle of validation.
- test: test(oop): add cases of class car.
- chore: chore(ci): create a workflow of automatic test.

### 1. The Heart of Git: The `.git` Directory

The `.git` folder is the object database where all project history and metadata are stored.

#### A. The Object Database (`objects/`)

This is where Git stores all content using the **SHA-1 algorithm**, which generates a unique 40-character hexadecimal ID based on the file's content.

- **Blobs (Binary Large Objects):** Store the raw content of a file. Blobs do not store the filename or permissions—only the data.
- **Trees:** Store the directory structure. A tree maps filenames and permissions to the hashes of their corresponding blobs or other sub-trees.
- **Commits:** Store the snapshot metadata: the author, the committer, the date, the commit message, and the hash of the **root Tree** representing the project state at that moment. They also point to the parent commit(s), creating the history chain.

#### B. References and Pointers

- **`refs/`:** These are pointers to specific commit hashes. For example, a branch (like `main` or `gh-pages`) is simply a text file in `refs/heads/` containing the hash of the latest commit on that branch.
- **`HEAD`:** A file that indicates the current active checkout (which branch or commit you are currently on).
  - **Example:** `snatbep [.git] $ cat HEAD` $\rightarrow$ `ref: refs/heads/main`

#### C. The Index File (Staging Area)

The **Index** is a binary file that acts as a "draft" for the next snapshot. When you run `git add`, Git:

1. Creates a blob in `objects/` for the content.
2. Updates the index file to record that this file (with its name) should be included in the next commit.

---

### 2. Storage Optimization: Object Sharding

To prevent the filesystem from slowing down due to thousands of files in a single directory, Git uses **sharding**.

Git takes the first **2 characters** of the SHA-1 hash to create a directory and stores the remaining **38 characters** as the filename.

**Example Process:**

```bash
snatbep [Git] $ git hash-object objects_blob_snapshot
# Result: f93ecf51e852619d25434a58a28af755d06b3c92
```

**Physical Storage Path:**
`objects/f9/3ecf51e852619d25434a58a28af755d06b3c92`

_Note: Because Git is immutable, modifying a file creates a completely new hash and thus a new object in a potentially different shard folder._

---

### 3. The Local Lifecycle (Workflow)

The movement of a file through the Git stages:

1. **Untracked:** Files in the working directory that have not been added to the index yet.
2. **Modified:** Files that were already tracked by Git but have changes in the working directory that are not yet staged.
3. **Staged:** Files that have been added to the index (`git add`). Their content is already saved as a blob in `objects/`.
4. **Committed:** Changes are permanently registered in the local repository. A commit object is created, and the branch pointer (`refs/heads/main`) is updated.
5. **Remote:** The local commits are pushed to a host (e.g., GitHub) via `git push`.

---

### 4. Automation with Hooks

Hooks are scripts that Git executes before or after specific events (like committing). They are used to enforce professional standards and semantic rules.

**Example: Setting up a commit-message hook**

```bash
cd .git/hooks
mv commit-msg.sample commit-msg
chmod +x commit-msg
```

If the `commit-msg` hook detects a bad format or an empty message, it will abort the commit process.

---

### 5. Technical Plumbing Commands

These commands allow you to interact directly with the object database:

| Command                                             | Description                                                     |
| :-------------------------------------------------- | :-------------------------------------------------------------- |
| `git cat-file -p <hash>`                            | Pretty-print the content of any object (blob, tree, or commit). |
| `git ls-tree <hash>`                                | List the contents of a Tree object.                             |
| `git rev-parse HEAD`                                | Returns the exact SHA-1 hash of the current commit.             |
| `git verify-pack -v .git/objects/pack/pack-xxx.idx` | Analyzes compressed packfiles (used by `git gc`).               |

---

### Commit Correction

| Command                               | Description                                                              |
| :------------------------------------ | :----------------------------------------------------------------------- |
| `git commit --amend --no-edit`        | Update the last commit with staged changes without changing the message. |
| `git commit --amend -m "New Message"` | Rewrite the last commit message.                                         |
| `git reset --hard HEAD~1`             | Permanently discard the last commit and all its changes.                 |

---

## 6. Branching and Merging

### The Nature of Branches

A branch is not a folder or a copy of the code; it is merely a **pointer to a commit**.

#### Special Case: The Orphan Branch (`gh-pages`)

An orphan branch is a branch that starts with no history. It is ideal for documentation or static sites.

**Workflow for `gh-pages`:**

```bash
git checkout --orphan gh-pages   # Create branch with no parent
git rm -rf .                    # Clear working directory
git checkout main -- Web/       # Pull only specific files from main
git add Web/
git commit -m "Deploy static content"
git push -u origin gh-pages
```

#### Merge Strategies

- **Fast-Forward:** Occurs when the target branch has no new commits since the feature branch started. Git simply moves the pointer forward.
- **Three-Way Merge (Manual):** Occurs when both branches have diverged. Git finds the common ancestor and creates a new "Merge Commit" to join the two histories.

---

## 7. Collaborative Workflow: The Pull Request (PR)

A PR is essentially a request to merge one branch into another, usually after a code review.

**Professional PR Workflow (using SSH Aliases):**

1. **Fork** the original repository.
2. **Clone** using the alias: `git clone git@github.com-uni:username/repo.git`
3. **Configure local identity:** `git config --local user.email "uni@email.com"`
4. **Setup remotes:**
   - `origin`: Your fork.
   - `upstream`: The original project.
5. **Feature Cycle:**
   ```bash
   git checkout -b feature/new-component
   # ... make changes ...
   git add .
   git commit -m "feat: add new component"
   git push origin feature/new-component
   ```
6. **Sync and Cleanup:**
   ```bash
   git checkout main
   git pull upstream main        # Keep your local main updated
   git push origin main         # Update your fork
   git branch -D feature/new-component # Delete local branch
   ```

##### References

- [Jon Mircha - Git Guide](https://jonmircha.com/git#github-pages)
- [Book Git](https://git-scm.com/book/es/v2)
- [Doc Github](https://docs.github.com/es)
- [Pull Request](https://docs.github.com/en/pull-requests/reference/pull-requests)
