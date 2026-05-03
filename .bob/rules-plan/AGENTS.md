# AGENTS.md - Plan Mode Rules

This file provides guidance to agents when working with code in this repository.

## Planning for Demo Creation

**Demo Structure Requirements:**
- MUST plan for 4 folders: `prompt-templates/`, `input-documents/`, `optional-generated-content/`, `README.md`
- Prompt templates numbered sequentially: `01-`, `02-`, `03-` (with leading zeros)
- README documents the journey, not just the outcome
- No PowerPoint files - plan for Markdown + screenshots

**Skills-Based Demo Planning:**
- Skills are demo-specific, not repository-wide
- Each skill needs `SKILL.md` with YAML frontmatter + Markdown instructions
- Plan skill execution order carefully - they run sequentially
- Document dependencies between skills (see [`tekton-devops/input-documents/hello-world-tekton/AGENTS.md`](tekton-devops/input-documents/hello-world-tekton/AGENTS.md))

**Architecture Considerations:**
- Each demo is independent - no shared dependencies between demos
- Static HTML demos (like Northwind) must embed all content in JavaScript
- Skills-based demos require careful ordering and dependency management
- Branch strategy: Plan for users to create branches before starting

**Documentation Planning:**
- SQL query docs need 7 sections: Summary, Tables, Logic, Columns, Use Case, Assumptions, Tests
- Test cases: PT-01 to PT-05 (positive), EC-01 to EC-04 (edge cases)
- Each test case needs validation SQL
- Web interfaces should use modals for documentation display

**File Organization:**
- No spaces in filenames - use hyphens
- Consistent naming: `Q{N}_Documentation.md` for SQL docs
- Prompt templates: descriptive names with numbers