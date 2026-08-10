You are acting as **Setup Assistant** for the SDLC Playbook (Option B)

## Your task

Create the directory structure and copy the necessary files for the Option B workflow in this project.

## Steps

### STEP 1 — Find sdlc-playbook

Search for the sdlc-playbook path in this order:
1. Check `~/.sdlc-playbook-path` (if it exists, read the path from this file)
2. Check `~/Workspace/ai/sdlc-playbook`
3. Check `~/sdlc-playbook`
4. If not found → ask the user where sdlc-playbook is located, then save that path to `~/.sdlc-playbook-path`

### STEP 2 — Create directory structure

Run the following bash commands:

```bash
mkdir -p docs/roles/po
mkdir -p docs/roles/sa/adr
mkdir -p docs/roles/lead
mkdir -p docs/roles/qa
mkdir -p docs/shared/tasks
mkdir -p .claude/commands
```

### STEP 3 — Copy files from sdlc-playbook

```bash
# Copy role instruction files
cp -r <SDLC_PLAYBOOK_PATH>/ai ./ai

# Copy slash commands (excluding setup.md itself)
cp <SDLC_PLAYBOOK_PATH>/templates/option-b/commands/po.md .claude/commands/po.md
cp <SDLC_PLAYBOOK_PATH>/templates/option-b/commands/sa.md .claude/commands/sa.md
cp <SDLC_PLAYBOOK_PATH>/templates/option-b/commands/lead.md .claude/commands/lead.md
cp <SDLC_PLAYBOOK_PATH>/templates/option-b/commands/qa.md .claude/commands/qa.md
```

Replace `<SDLC_PLAYBOOK_PATH>` with the actual path found in STEP 1.

### STEP 4 — Create initial CLAUDE.md

Create the file `CLAUDE.md` at the project root (if it doesn't already exist) with this content:

```markdown
# CLAUDE.md

You are acting as **Developer** for this project.

## Instructions
Read and follow every rule in `ai/DEV_PROJECT_INSTRUCTIONS.md`

## Your knowledge files
Read the following files in `docs/shared/` silently before starting any task:
- `tasks/Task_[ID]_[title].md` — the task received from Lead
- `TASK_LOG.md` — see completed tasks and done criteria

## Key rules
- Implement only what is specified in the task
- Update `docs/shared/TASK_LOG.md` every time a task is complete
- Do not go outside the scope of the task you received
```

### STEP 5 — Create initial TASK_LOG.md

Create the file `docs/shared/TASK_LOG.md` if it doesn't already exist:

```markdown
# Last updated: <TODAY_DATE> | Version: 1

# TASK_LOG.md

| Task ID | Title | Status | Done Criteria Met | Notes |
|---------|-------|--------|-------------------|-------|
```

### STEP 6 — Show summary

After completing the setup, display:
1. The directory structure created (use `find . -type d | sort`)
2. Files that were copied
3. Next steps for the user:

```
Setup complete ✓

Next steps:
1. Type /po to start a PO session and analyse your first feature
2. Or type /sa if you want to design the architecture first
```
