# Approved Documents

**Status**: Production-Ready
**Last Updated**: November 9, 2025

---

## What's Here

This folder contains **ONLY production-ready approved documents** from all planning phases.

**✅ Approved** = Ready for implementation when credentials/resources available

---

## Quick Start

1. **New to the project?** → Read [SYSTEM-OVERVIEW.md](./SYSTEM-OVERVIEW.md)
2. **Starting implementation?** → Read the stage's APPROVED PLAN
3. **Need details?** → Check [APPROVED-DOCUMENTS-INDEX.md](./APPROVED-DOCUMENTS-INDEX.md)

---

## Structure

```
approved/
├── SYSTEM-OVERVIEW.md                  # Master overview (all stages)
├── APPROVED-DOCUMENTS-INDEX.md        # Complete index with descriptions
├── stage-a-architecture/              # Stage A approved docs
├── stage-b-skill-system/              # Stage B approved docs
└── stage-c-knowledge-base/            # Stage C approved docs
```

**Total**: 10 approved documents across 3 stages

---

## Current Stages

| Stage | Name | Status | Approved Docs |
|-------|------|--------|---------------|
| A | Architecture | ✅ Complete, in production | 1 ADR |
| B | Skill System | ✅ Planning complete | 1 plan + 3 ADRs |
| C | Knowledge Base | ⏸️ Blocked (pending credentials) | 1 plan + 3 execution docs |

---

## Rules

### ✅ What Goes Here
- Approved implementation plans (STAGE-X-APPROVED-PLAN.md)
- Architecture Decision Records (ADRs)
- Execution trackers and checklists
- Final approved deliverables

### ❌ What Stays Outside
- Research documents → `.planning/research/`
- Work-in-progress → `.planning/` (root)
- Navigation docs → `.planning/` (root)
- Archives → `.planning/.archive/`

---

## How to Add New Stages

When Stage D (or E, F, etc.) planning is complete:

1. Create stage folder: `approved/stage-d-name/`
2. Move approved plan: `STAGE-D-APPROVED-PLAN.md`
3. Move ADRs (if any): `approved/stage-d-name/decisions/`
4. Update: `APPROVED-DOCUMENTS-INDEX.md`
5. Update: `SYSTEM-OVERVIEW.md`

**Result**: Clean separation between planning and approved documents.

---

## Documentation

- **Complete Index**: [APPROVED-DOCUMENTS-INDEX.md](./APPROVED-DOCUMENTS-INDEX.md) - Detailed descriptions of all approved documents
- **Master Overview**: [SYSTEM-OVERVIEW.md](./SYSTEM-OVERVIEW.md) - Complete system architecture and stage summaries
- **Project Docs**: See `.planning/README.md` for full project documentation

---

## Implementation Workflow

When starting implementation of a stage:

```mermaid
graph LR
    A[Read Approved Plan] --> B[Check Readiness]
    B --> C[Use Execution Tracker]
    C --> D[Reference Research Docs]
    D --> E[Follow ADR Decisions]
    E --> F[Implement]
```

**Key Point**: ADR decisions are **non-negotiable** - they represent approved architectural choices.

---

## Questions?

- **What's in each stage?** → [APPROVED-DOCUMENTS-INDEX.md](./APPROVED-DOCUMENTS-INDEX.md)
- **Big picture?** → [SYSTEM-OVERVIEW.md](./SYSTEM-OVERVIEW.md)
- **How to start?** → See `.planning/START-HERE.md`
- **Technical details?** → See `.planning/research/` folder

---

**Maintained By**: Planning agents
**Updated**: After each stage approval
**Version**: 1.0.0

---

**Purpose**: Guarantee clean state for development by clearly separating approved (production-ready) documents from work-in-progress planning.
