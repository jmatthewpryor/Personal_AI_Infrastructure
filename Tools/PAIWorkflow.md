# PAI Development Workflow

> **FOR AI AGENTS:** This document defines how to work on PAI infrastructure. Follow these workflows when fixing bugs, adding features, or customizing installations.

---

## Core Principle

**Installations are deployments of PAI source, not forks.**

```
PAI Source Repository        User Installation
(~/proj/personal/PAI)   →    (~/.claude or $PAI_DIR)
     Blueprint                 Deployment
```

This separation enables:
- Bug fixes flow back to source for everyone
- Personal customizations stay local
- Clear tracking of what's installed
- Bi-directional updates possible

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

```
┌─────────────────────────────────────────────────────────┐
│ 1. IDENTIFY in installation (~/.claude)                 │
│    "This hook is failing because..."                    │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ 2. LOCATE source pack                                   │
│    ~/proj/personal/PAI/Packs/pai-hook-system/src/       │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ 3. FIX in PAI source FIRST                              │
│    Edit the file in Packs/*/src/                        │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ 4. COPY fix to installation                             │
│    cp ~/proj/personal/PAI/Packs/*/src/file ~/.claude/   │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ 5. TEST in installation                                 │
│    Verify the fix works                                 │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ 6. COMMIT PAI source                                    │
│    git add/commit/push in ~/proj/personal/PAI           │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│ 7. UPDATE manifest                                      │
│    Add entry to drift_log in pai-manifest.json          │
└─────────────────────────────────────────────────────────┘
```

---

## Workflow: Adding a Feature

```
1. DESIGN in PAI source
   - Determine which pack it belongs to
   - If new pack needed, use PAIPackTemplate.md

2. IMPLEMENT in Pack's src/ directory
   - Add all necessary files
   - Update INSTALL.md with installation steps
   - Update VERIFY.md with verification checks

3. TEST by installing to user installation
   - Run the INSTALL.md steps
   - Verify with VERIFY.md

4. ITERATE until working
   - Fix issues in source
   - Re-sync to installation
   - Test again

5. COMMIT when complete
   - git add/commit/push PAI source
   - Update manifest with new pack version
```

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
