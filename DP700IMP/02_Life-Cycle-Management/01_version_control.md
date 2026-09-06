# Fabric Git Integration — DP-700 Notes

## Core Idea

**Git Integration = Fabric Workspace + Git Repository**

Git provides version history, collaboration, and change tracking for supported Fabric items.

```text
Fabric Workspace ↔ Git Repository
```

---

## 1. Supported Git Systems

### Supported
- Azure DevOps
- GitHub Cloud
- GitHub Enterprise Cloud

### Not Supported
- GitHub Enterprise Server / on-premises GitHub Enterprise Server

**Memory Trick:**  
> Fabric Git = Azure DevOps or GitHub Cloud

---

## 2. Workspace-Level Git Integration

Git integration is connected to a **Fabric workspace**, not the entire tenant.

```text
Fabric
├── Sales Workspace → Git connected
├── HR Workspace → Git not connected
└── Finance Workspace → Git connected
```

**Memory Trick:**  
> Git connection = Workspace level

---

## 3. Commit

**Commit = Workspace → Git**

```text
Fabric Workspace
      ↓
    COMMIT
      ↓
      Git
```

You are saying: **"Save my Fabric changes into Git."**

**Memory Trick:**  
> Commit = Send my work to Git

---

## 4. Update

**Update = Git → Workspace**

```text
Git
 ↓
UPDATE
 ↓
Fabric Workspace
```

You are saying: **"Bring the latest Git changes into my workspace."**

**Memory Trick:**  
> Update = Get work from Git

### Commit vs Update

| Action | Direction |
|---|---|
| Commit | Workspace → Git |
| Update | Git → Workspace |

> **COMMIT = OUT**  
> **UPDATE = IN**

Commit and Update are one-directional actions; they do not happen simultaneously as one sync action.

---

## 5. Git Branch

A Git branch is an **isolated line of development**.

```text
main
├── feature/customer-pipeline
├── feature/sales-dashboard
└── feature/new-notebook
```

**Memory Trick:**  
> Branch = Separate line of development

---

## 6. Branch Out

**Branch Out creates:**
- A new Fabric workspace
- A new Git branch

```text
Existing Workspace
       │
   Branch Out
       ↓
New Workspace
       +
New Git Branch
```

Use Branch Out for significant/non-trivial feature work.

**Memory Trick:**  
> Branch Out = New Workspace + New Branch

---

## 7. Switch Branch

Switch Branch changes the Git branch used by the **same workspace**.

It does **not** create a new workspace.

```text
Same Workspace
      │
      ├── main
      └── feature-A

Switch:
main → feature-A
```

Switching branches can change/overwrite the current workspace content to match the selected branch.

**Memory Trick:**  
> Switch = Same Workspace + Different Branch

### Branch Out vs Switch Branch

| | Branch Out | Switch Branch |
|---|---|---|
| New workspace? | Yes | No |
| New Git branch? | Yes | No, uses another branch |
| Purpose | Isolated feature development | Change current workspace to another branch |
| Risk | Lower | Can overwrite workspace content |

> **BRANCH OUT = NEW**  
> **SWITCH = CHANGE**

---

## 8. Allow Contributors to Change Git Branch

Fabric has a setting:

**Allow users with at least Contributor role to change Git branch**

Enable it deliberately when the team needs it.

Why? Because switching branches can change/overwrite workspace content.

**Memory Trick:**  
> Branch Switch = Workspace Change → Control Who Can Switch

---

## 9. Conflict

A conflict occurs when the **same item** was changed in both:
- the Fabric workspace
- Git

since the last synchronization.

```text
Last Sync
    ↓
Workspace changed the pipeline
    +
Git changed the same pipeline
    ↓
CONFLICT
```

**Memory Trick:**  
> Conflict = Same item changed on both sides

---

## 10. Conflict Resolution

There are three main paths.

### Option 1 — UI Accept/Keep

Choose which version should be kept.

Use this when one version should simply replace the other.

### Option 2 — Revert

Discard an unwanted change.

```text
Your workspace change
        ↓
      REVERT
        ↓
Discard change
```

### Option 3 — Resolve in Git / PR

Use Git-side merging when **both changes are useful** and need to be reconciled.

Example:

```text
Developer A → changed schedule
Developer B → changed parameters
        ↓
Git / PR merge
        ↓
Combine the changes
```

**Memory Trick:**  
> Conflict = Both sides changed  
> Real merge = Resolve in Git/PR

---

## 11. Commit Early and Often

Avoid waiting weeks and making one huge commit.

### Bad

```text
3 weeks of work
      ↓
100 changes
      ↓
One huge commit
```

Large commits make conflicts harder to understand and resolve.

### Better

```text
Small change
   ↓
Commit

Another change
   ↓
Commit
```

Use meaningful messages.

### Good examples

```text
Add customer ingestion pipeline
```

```text
Update sales transformation logic
```

**Memory Trick:**  
> Small + meaningful commits = Easier conflict resolution

---

## 12. Supported Items

Not every Fabric item type is necessarily supported by Git integration.

The supported-items list can change over time.

Therefore:

> Always check the current supported-items list before assuming an item is version-controlled.

**Memory Trick:**  
> Supported-items list = Living document

---

## 13. Unsupported Items

If an item isn't supported by Git:

> Git ignores it.

It does **not** mean Fabric deletes it.

```text
Unsupported item
      ↓
Git doesn't track it
      ↓
Item remains in Fabric
```

**Memory Trick:**  
> Unsupported ≠ Deleted

---

## 14. Git Status

### Synced

Workspace and Git match.

```text
Workspace = Git
```

Nothing needs to be done.

### Uncommitted

The workspace has changes that haven't been committed.

```text
Workspace ≠ Git
```

Typical action: **Commit**

### Update Required

Git has changes that aren't yet in the workspace.

Typical action: **Update**

### Conflict

Both Workspace and Git changed the same item.

Typical action: **Resolve the conflict**

### Unsupported

The item isn't supported by Git integration.

It remains in Fabric but isn't version-controlled through Git.

### Git Status Memory

```text
SYNCED
   ↓
Everything matches

UNCOMMITTED
   ↓
Workspace changed
   ↓
COMMIT

UPDATE REQUIRED
   ↓
Git changed
   ↓
UPDATE

CONFLICT
   ↓
Both sides changed
   ↓
RESOLVE

UNSUPPORTED
   ↓
Git doesn't track it
```

---

## 15. Permissions

### Connect / Disconnect / Sync

Require:

> **Workspace Admin**

```text
Connect
Disconnect
Sync
   ↓
Admin
```

### Commit / Update

Require:

> **Contributor or higher**, with the necessary write access on the relevant items.

```text
Commit
Update
   ↓
Contributor+
```

**Memory Trick:**  
> Admin = Connection / Sync  
> Contributor+ = Commit / Update

---

# Complete Real-Life Example

Imagine:

```text
Fabric Workspace
       ↕
     GitHub
```

### Step 1 — Create

Create a new pipeline.

Status:

```text
UNCOMMITTED
```

### Step 2 — Commit

```text
Workspace
   ↓
COMMIT
   ↓
Git
```

Status becomes:

```text
SYNCED
```

### Step 3 — Major Feature

Use:

```text
Branch Out
```

Now you have:

```text
New Workspace
      +
New Git Branch
```

### Step 4 — Develop

Make changes and commit regularly.

### Step 5 — Review

Use the Git/PR workflow for review.

### Step 6 — Conflict

If another developer changed the same item in Git:

```text
Workspace version
       VS
Git version
```

Result: **Conflict**

### Step 7 — Resolve

Choose:

```text
UI Accept/Keep
OR
Revert
OR
Git/PR Merge
```

---

# Complete Git Flow

```text
                 FABRIC WORKSPACE
                        │
                        ↕
                   GIT INTEGRATION
                        │
                Azure DevOps / GitHub
                        │
             ┌──────────┴──────────┐
             │                     │
          COMMIT                 UPDATE
             │                     │
             ↓                     ↓
            GIT                WORKSPACE
```

For feature development:

```text
Shared Branch
     │
     │ Branch Out
     ↓
New Workspace + New Branch
     │
     ↓
Develop
     │
     ↓
Small meaningful commits
     │
     ↓
Review / PR
```

If both sides change:

```text
Workspace changed
       +
Git changed
       ↓
   CONFLICT
       ↓
 ┌─────┼─────┐
 ↓     ↓     ↓
UI   Revert  Git/PR
```

---

# DP-700 Exam Cheat Sheet

| Concept | Remember |
|---|---|
| Git Integration | Workspace-level version control |
| Supported Git | Azure DevOps + GitHub Cloud |
| GitHub Enterprise Server | Not supported |
| Commit | Workspace → Git |
| Update | Git → Workspace |
| Branch | Isolated development line |
| Branch Out | New workspace + new branch |
| Switch Branch | Same workspace + different branch |
| Switch Branch risk | Can overwrite workspace content |
| Contributor branch setting | Enable deliberately |
| Conflict | Same item changed on both sides |
| UI Accept/Keep | Choose one version |
| Revert | Discard unwanted change |
| Git/PR | Merge/reconcile changes |
| Unsupported item | Ignored, not deleted |
| Synced | Workspace = Git |
| Uncommitted | Workspace has changes |
| Update required | Git has changes |
| Conflict | Both sides changed |
| Connect/Disconnect/Sync | Admin |
| Commit/Update | Contributor+ |
| Best practice | Commit early + meaningful messages |
| Large feature | Branch Out |
| Supported items | Check current list |

---

# ⭐ 10 Things to Memorize

1. **Git Integration = Workspace + Git version control.**
2. **Supported Git = Azure DevOps or GitHub Cloud.**
3. **Commit = Workspace → Git.**
4. **Update = Git → Workspace.**
5. **Branch Out = New workspace + new branch.**
6. **Switch Branch = Same workspace + another branch.**
7. **Same item changed on both sides = Conflict.**
8. **Conflict → UI choice, Revert, or Git/PR merge.**
9. **Unsupported item = Ignored, NOT deleted.**
10. **Admin = Connect/Disconnect/Sync; Contributor+ = Commit/Update.**

## Ultimate Memory Sentence

> **"I develop in a workspace, Commit my work to Git, Update my workspace from Git, Branch Out for big features, and resolve Conflicts when both sides changed."**

## Final Memory

```text
GIT INTEGRATION = VERSION CONTROL

COMMIT = WORKSPACE → GIT

UPDATE = GIT → WORKSPACE

BRANCH OUT = NEW WORKSPACE + NEW BRANCH

SWITCH = SAME WORKSPACE + DIFFERENT BRANCH

CONFLICT = BOTH SIDES CHANGED

UNSUPPORTED = IGNORED, NOT DELETED

ADMIN = CONNECT / DISCONNECT / SYNC

CONTRIBUTOR+ = COMMIT / UPDATE

SMALL COMMITS = EASIER CONFLICTS
```
