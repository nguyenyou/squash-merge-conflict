# Why Squash and Merge Causes Conflicts on Rebase

## The Analogy: Photocopying vs Rewriting

Imagine you write a 3-page essay (commits 1, 2, 3). You submit it to your teacher (PR).

**Regular merge** = Teacher staples your original pages into the class binder
**Squash merge** = Teacher *rewrites* your 3 pages into 1 summary page, throws away your originals

Later, you try to "update" your original pages with the class binder. Git sees:
- Your originals: page 1, page 2, page 3
- The binder: a *different* summary page

Git doesn't know they're the same content because they have **different commit hashes**.

---

## The Diagram: What Actually Happened

```
BEFORE squash merge:

    main:     A (Initial commit: "# squash-merge-conflict")
              |
              +--→ B (commit 1: "1")
                   |
                   C (commit 2: "2")
                   |
                   D (commit 3: "3")   ← feature-1 branch


AFTER squash merge on GitHub:

    main:     A ──→ S (squash commit: "3")   ← NEW commit hash f9f6fec
              |
              +--→ B ──→ C ──→ D             ← feature-1 still has ORIGINAL commits
                                               (99ba434, 38916d9, 3e0ef34)


WHEN you rebase feature-1 onto main:

    Git tries to replay B, C, D on top of S:

    main:     A ──→ S (contains: "# squash-merge-conflict" → "3")
                    |
                    +--→ B' (tries to apply: "# squash-merge-conflict" → "1")
                         ↑
                         CONFLICT! Base doesn't match!
```

---

## Step-by-Step: What Git Sees During Rebase

1. **Git starts rebasing** commit B (99ba434) onto S (f9f6fec)

2. **Commit B says**: "Change line 1 from `# squash-merge-conflict` to `1`"

3. **But S already changed** line 1 from `# squash-merge-conflict` to `3`

4. **Git is confused**:
   - "I need to change `# squash-merge-conflict` to `1`"
   - "But I can't find `# squash-merge-conflict` - I only see `3`"
   - **CONFLICT!**

---

## The Gotcha: Same Content, Different Identity

The cruel irony: **S and D have the exact same file content** (`3`), but Git doesn't compare content - it compares **commit ancestry**.

```
S (f9f6fec) ≠ D (3e0ef34)   ← Different hashes = different commits to Git
     ↓              ↓
   "3"            "3"        ← Same content, but Git doesn't care!
```

Git uses SHA hashes (commit IDs) to track history. Squash creates a **new commit** with a **new hash**, even though the content is identical. To Git, they're strangers.

---

## The Solution

After squash merge, **delete your feature branch** and create a new one from main:

```bash
# Don't do this:
git checkout feature-1
git fetch origin
git rebase origin/main   # CONFLICT!

# Do this instead:
git checkout main
git pull origin main
git branch -D feature-1   # Delete old feature branch
# Start fresh if you need to continue work
```

Or use **regular merge** instead of squash merge if you plan to continue working on the branch.

---

## Your Repo State

```
main (origin):      97d8b97 ──→ f9f6fec (squash: "Feature 1 (#1)")
                        |
feature-1 (local):      +──→ 99ba434 ──→ 38916d9 ──→ 3e0ef34
                             (commit 1)   (commit 2)   (commit 3)
```

The commits `99ba434`, `38916d9`, `3e0ef34` are now "orphaned" - their changes already exist in `f9f6fec` but Git doesn't know that because squash created a brand new commit.
