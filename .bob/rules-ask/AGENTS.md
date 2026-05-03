# AGENTS.md - Ask Mode Rules

This file provides guidance to agents when working with code in this repository.

## Repository Context for Questions

**Demo Collection Structure:**
- This is NOT a single application - each subdirectory is an independent demo
- When answering questions, clarify which demo the user is asking about
- Demos span multiple technologies: Ansible, Tekton, Quarkus, React, SQL, etc.

**Documentation Locations:**
- Main README: Overview of all demos and getting started
- Demo-specific READMEs: Step-by-step journey for each demo
- `prompt-templates/`: Contains the exact prompts used with Bob
- `TECHNICAL_SPEC.md`: Detailed specification for Northwind SQL demo

**Skills-Based Demos:**
- Skills are NOT global - they're demo-specific in `skills/` subdirectories
- Each skill has `SKILL.md` with YAML frontmatter + instructions
- Skills must be executed in order (see [`tekton-devops/input-documents/hello-world-tekton/AGENTS.md`](tekton-devops/input-documents/hello-world-tekton/AGENTS.md))

**Common Confusion Points:**
- `bob-get-started/` contains example apps, not Bob itself
- `demo-template/` is a template, not a working demo
- `optional-generated-content/` may be empty if output isn't needed for demo understanding
- Branch strategy: Users create branches before demos, main is clean starting point

**File Naming Patterns:**
- Prompt templates: `01-descriptive-name-prompt.md` (numbered with leading zero)
- SQL docs: `Q{N}_Documentation.md` format
- No spaces in filenames - hyphens only