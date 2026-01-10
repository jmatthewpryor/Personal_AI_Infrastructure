# PAI Architecture Understanding

> **AUTO-LOAD**: This document defines the relationship between PAI source and your installation. Reference this when making any changes to hooks, skills, or infrastructure.

---

## Core Mental Model

**Your installation is a deployment of PAI, not a fork.**

```
PAI Source                    Your Installation
{{PAI_SOURCE_PATH}}    →      {{PAI_DIR}}
     (Blueprint)              (Deployment)
```

This is like source code → production deployment:
- You don't edit production directly
- You edit source, commit, then deploy
- Production can have local configuration that doesn't go back to source

---

## Key Locations

| Location | Purpose | Git Tracked |
|----------|---------|-------------|
| `{{PAI_SOURCE_PATH}}` | Open source PAI project (canonical source) | Yes - push to GitHub |
| `{{PAI_DIR}}` | Your installation (personal deployment) | No - derived from source |

---

## File Classification

### Source-Managed Files (Edit in PAI source, sync to installation)

These files come from PAI packs and should be edited in source first:

- `hooks/*.ts` - Hook implementations
- `hooks/lib/*.ts` - Hook libraries
- `skills/*/Tools/*.ts` - Skill tools
- `skills/*/Workflows/*.md` - Workflow templates
- `settings.json` hooks configuration

**Workflow**: Fix in PAI source → Copy to installation → Commit PAI source

### Local-Only Files (Edit in installation, never sync upstream)

These files are personal and should never go back to PAI source:

- `.env` - Personal API keys and secrets
- `skills/CORE/SKILL.md` - Personal identity configuration
- `skills/Art/Aesthetic.md` - Personal aesthetic preferences
- `MEMORY/` - Session memory and learnings
- `history/` - Captured outputs
- `pai-manifest.json` - Local installation state

**Workflow**: Edit directly in installation, never push to PAI

### Template-Derived Files (Compare to canonical for ideas)

These start from PAI templates but are personalized:

- `skills/CORE/Contacts.md` - Personal contacts
- `skills/CORE/CoreStack.md` - Personal stack preferences

**Workflow**: Compare periodically with PAI templates for new suggestions

---

## Decision Workflows

### When Finding a Bug

```
1. Identify the issue in your installation
2. Ask: "Is this a pack bug or local misconfiguration?"
3. If pack bug:
   a. Locate source pack in {{PAI_SOURCE_PATH}}/Packs/
   b. Fix in PAI source FIRST
   c. Copy fix to installation
   d. Commit PAI source to git
   e. Test in installation
4. If local misconfiguration:
   a. Fix directly in installation
   b. No source changes needed
```

### When Adding Features

```
1. Design in PAI source
2. Implement in appropriate Pack's src/ directory
3. Update INSTALL.md if needed
4. Sync to installation for testing
5. Iterate until working
6. Commit PAI source
```

### When Customizing

```
1. Ask: "Is this personal or would others benefit?"
2. If personal (identity, aesthetic, keys):
   a. Edit directly in installation
   b. Never push to PAI source
3. If generic improvement:
   a. Follow bug fix workflow
   b. Consider if it should be a new pack
```

---

## Manifest System

The file `pai-manifest.json` tracks:

- **installed_packs**: Which packs are installed and their versions
- **source_location**: Path to PAI source
- **local_only_patterns**: Files that should never sync upstream
- **last_sync**: When installation was last synced from source
- **drift_detection**: Files that have diverged from source

---

## Quick Reference

| I want to... | Do this... |
|--------------|------------|
| Fix a hook bug | Edit in `PAI/Packs/*/src/`, copy to installation |
| Change my identity | Edit `skills/CORE/SKILL.md` directly |
| Add a new skill | Create in `PAI/Packs/`, install to installation |
| Update my aesthetic | Edit `skills/Art/Aesthetic.md` directly |
| Check what's installed | Read `pai-manifest.json` |
| See if I'm behind source | Compare manifest versions to PAI source |

---

## Important Reminders

1. **Never edit pack code directly in installation** - Always fix in PAI source first
2. **The manifest tracks truth** - Check it to understand installation state
3. **Local files are sacred** - `.env`, identity, aesthetic are personal
4. **Commit source changes** - Bug fixes should go back to the open source project
5. **Test in installation** - Always verify changes work in the actual deployment

---

*Installed from PAI Core Install pack. Customize this understanding as needed.*
