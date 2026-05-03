---
name: skill-init
description: Initialize a directory with AI-agnostic agent files. Use when the user asks to run "/init" in a directory or to initialize AI-agnostic files for a project.
---

# skill-init

This skill instructs the agent on how to initialize a repository to be AI-agnostic while still providing strong instructions and project context.

When triggered, execute the following steps in the current workspace directory:

1. **Create `AGENTS.md`:**
   - Create a file named `AGENTS.md` in the root directory.
   - If there is an existing `GEMINI.md` that contains project-specific information, rename it to `AGENTS.md`. If there is no existing file, create a new `AGENTS.md` and populate it with a basic template for project instructions (e.g., "Project Overview", "Setup", "Architecture", "Conventions").

2. **Create a Pointer `GEMINI.md`:**
   - Create a new file named `GEMINI.md` in the root directory.
   - Write the following exact content into `GEMINI.md` to point Gemini to the agnostic file:
     ```markdown
     # AI Agent Context

     The contextual instructions for AI agents working in this repository are located in [AGENTS.md](./AGENTS.md).

     Agents should read `AGENTS.md` to understand the project architecture, conventions, and build processes.
     ```

3. **Verify:**
   - Run `git status` to show the newly created files to the user and ask if they would like to commit them.