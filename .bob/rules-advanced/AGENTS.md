# AGENTS.md - Advanced Mode Rules

This file provides guidance to agents when working with code in this repository.

## Demo-Specific Coding Patterns

**4-Folder Structure Enforcement:**
- When creating new demos, MUST create all 4 folders: `prompt-templates/`, `input-documents/`, `optional-generated-content/`, `README.md`
- Prompt files MUST be numbered: `01-name.md`, `02-name.md` (not `1-name.md`)
- Never create PowerPoint files - use Markdown with screenshots instead

**Skills Implementation:**
- Skills live in `skills/` subdirectories within demo folders
- Each skill MUST have `SKILL.md` with YAML frontmatter followed by Markdown instructions
- Skills are executed sequentially - order matters (see [`tekton-devops/input-documents/hello-world-tekton/AGENTS.md`](tekton-devops/input-documents/hello-world-tekton/AGENTS.md))

**Web Interface Pattern (Northwind Demo):**
- Static HTML with embedded JavaScript - no build tools or external file loading
- Documentation content MUST be embedded in JavaScript objects, not loaded from files
- Modal popups use inline content to avoid CORS issues
- Print functionality opens new window with formatted content

**File Naming Conventions:**
- SQL documentation: `Q{N}_Documentation.md` (e.g., `Q1_Documentation.md`)
- No spaces in filenames - always use hyphens
- Prompt templates: descriptive names with numbers (e.g., `01-install-httpd-prompt.md`)

**MCP and Browser Tools Available:**
- This mode has access to MCP servers and browser automation capabilities
- Use these tools when demos require external integrations or web interactions