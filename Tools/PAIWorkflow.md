# PAI Development Workflow

> **FOR AI AGENTS:** This document defines how to work on PAI infrastructure. Follow these workflows when fixing bugs, adding features, or customizing installations.

---

## Core Principle

**Installations are deployments of PAI, via a fork.**

```
danielmiessler/PAI (upstream)     Canonical community source
        ↓ fork
your-username/PAI (origin)        Your fork
        ↓ clone
~/proj/personal/PAI               Local working copy
        ↓ install                 (main = upstream, local = personal)
~/.claude or $PAI_DIR             Deployment
```

This architecture enables:
- **Upstream sync**: Pull community updates when ready
- **Personal changes**: Keep in fork's `local` branch
- **Contribution**: PR from fork to upstream
- **Clear tracking**: Know what's yours vs community

### Branch Strategy

| Branch | Purpose |
|--------|---------|
| `main` | Tracks `upstream/main` - community canonical |
| `local` | Personal changes - never goes upstream |
| `fix/*` | Bug fixes intended for upstream PR |
| `feat/*` | Features intended for upstream PR |

---

## File Classification

### Source-Managed (Edit in PAI, sync to installation)

| File Pattern | Location in Source |
|--------------|-------------------|
| `hooks/*.ts` | `Packs/pai-hook-system/src/*.ts` |
| `hooks/lib/*.ts` | `Packs/pai-hook-system/src/lib/*.ts` |
| `skills/*/Tools/*.ts` | `Packs/pai-*/src/skills/*/Tools/*.ts` |
| `skills/*/Workflows/*.md` | `Packs/pai-*/src/skills/*/Workflows/*.md` |

**Rule**: Always edit in PAI source first, then copy to installation.

### Local-Only (Edit in installation, never upstream)

| File | Purpose |
|------|---------|
| `.env` | API keys and secrets |
| `pai-manifest.json` | Installation state tracking |
| `skills/CORE/SKILL.md` | Personal identity |
| `skills/Art/Aesthetic.md` | Personal aesthetic |
| `MEMORY/**` | Session memory |
| `history/**` | Captured outputs |

**Rule**: Edit directly in installation, never push to PAI source.

### Template-Derived (Compare periodically)

| File | Template |
|------|----------|
| `skills/CORE/Contacts.md` | `Packs/pai-core-install/src/skills/CORE/Contacts.md` |
| `skills/CORE/CoreStack.md` | `Packs/pai-core-install/src/skills/CORE/CoreStack.md` |

**Rule**: Compare with templates periodically for new suggestions.

---

## Workflow: Fixing a Bug

### Step 1: Identify and Classify

```
1. IDENTIFY in installation (~/.claude)
   "This hook is failing because..."

2. ASK: Should this fix go upstream?
   - YES → Benefits all PAI users → Create PR
   - NO → Personal/installation-specific → Local branch only
```

### If Contributing Upstream (PR)

```bash
cd ~/proj/personal/PAI

# Start from fresh main
git checkout main
git fetch upstream
git merge upstream/main

# Create fix branch
git checkout -b fix/descriptive-name

# Make the fix in Packs/*/src/
# Copy to installation for testing
cp Packs/*/src/file ~/.claude/path/

# Test, then commit
git add . && git commit -m "fix(pack-name): description"
git push origin fix/descriptive-name

# Create PR to upstream via GitHub
# After merge: git branch -d fix/descriptive-name
```

### If Local-Only Fix

```bash
cd ~/proj/personal/PAI

# Work on local branch
git checkout local

# Make the fix in Packs/*/src/
# Copy to installation for testing
cp Packs/*/src/file ~/.claude/path/

# Test, then commit
git add . && git commit -m "fix(local): description"
git push origin local

# Update manifest drift_log
```

---

## Workflow: Adding a Feature

### If Contributing Upstream (PR)

```bash
cd ~/proj/personal/PAI

# Start from fresh main
git checkout main
git fetch upstream
git merge upstream/main

# Create feature branch
git checkout -b feat/descriptive-name

# Implement in Pack's src/ directory
# - Add all necessary files
# - Update INSTALL.md with installation steps
# - Update VERIFY.md with verification checks

# Test by installing to your installation
# Run the INSTALL.md steps, verify with VERIFY.md

# Commit and push
git add . && git commit -m "feat(pack-name): description"
git push origin feat/descriptive-name

# Create PR to upstream via GitHub
```

### If Personal Feature

```bash
cd ~/proj/personal/PAI

# Work on local branch
git checkout local

# Implement in Pack's src/ directory
# Test by installing to your installation

# Commit and push
git add . && git commit -m "feat(local): description"
git push origin local

# Update manifest with new feature
```

---

## Workflow: Syncing with Upstream

When the community (upstream) has updates you want:

```bash
cd ~/proj/personal/PAI

# 1. Update main from upstream
git checkout main
git fetch upstream
git merge upstream/main
git push origin main

# 2. Bring updates into local branch
git checkout local
git merge main
# Resolve any conflicts (your local changes vs upstream)
git push origin local

# 3. Re-install affected packs to your installation
# Copy updated files from Packs/*/src/ to ~/.claude/
```

### Conflict Resolution

When merging upstream into local:
- **Your change is better**: Keep yours, consider PR to upstream
- **Upstream is better**: Accept theirs
- **Both needed**: Merge carefully, test thoroughly

---

## Workflow: Customizing Installation

```
1. DETERMINE type of customization
   ├── Personal (identity, aesthetic, keys)
   │   └── Edit directly in installation
   │       Never push to PAI source
   │
   └── Generic improvement (would help others)
       └── Follow "Fixing a Bug" workflow
           Push to PAI source
```

---

## Manifest System

Every installation should have `pai-manifest.json` tracking:

```json
{
  "installed_packs": { ... },      // What's installed
  "source": { ... },               // Where PAI source lives
  "local_only_patterns": [ ... ],  // Files that never sync upstream
  "drift_log": [ ... ]             // Changes made and why
}
```

### Using the Manifest

**Before making changes:**
```bash
cat ~/.claude/pai-manifest.json | jq '.installed_packs'
```

**After fixing a bug:**
```bash
# Add to drift_log
{
  "date": "2026-01-11",
  "file": "hooks/lib/observability.ts",
  "issue": "Missing from installation",
  "resolution": "Fixed in PAI source, synced to installation"
}
```

---

## AI Agent Instructions

When working on PAI infrastructure:

1. **Always ask first**: "Is this a source-managed file or local-only?"

2. **For source-managed files**:
   - Locate the Pack in `~/proj/personal/PAI/Packs/`
   - Edit there first
   - Copy to installation
   - Remind user to commit

3. **For local-only files**:
   - Edit directly in installation
   - Never mention pushing to source

4. **When in doubt**:
   - Check `pai-manifest.json` for classification
   - Ask user if unclear

5. **After significant changes**:
   - Update `drift_log` in manifest
   - Suggest committing PAI source if applicable

---

## Quick Reference

| I want to... | Workflow |
|--------------|----------|
| Fix a hook bug | Edit PAI source → Copy to installation → Commit |
| Add a new skill | Create pack in PAI → Install → Commit |
| Change my identity | Edit installation directly |
| Update aesthetic | Edit installation directly |
| Check what's installed | `cat pai-manifest.json` |
| See PAI source status | `cd ~/proj/personal/PAI && git status` |

---

*Part of the [PAI (Personal AI Infrastructure)](https://github.com/danielmiessler/PAI) project.*
