# 除錯提示

Generate a structured debug prompt to discuss complex bugs or architectural issues with another LLM.

## Usage

```
/debug_prompt
```

### Recommended Workflow

1. **First, describe the problem in conversation** - paste errors, mention files, explain what you tried
2. **Then call `/debug_prompt`** - the command gathers all context from the conversation

### Context Sources

The command gathers from:
1. **Recent conversation** - errors, file paths, attempted fixes mentioned
2. **Relevant code** - files mentioned or related to the issue
3. **Git status** - recently modified files that might be related

## When to Use

- Complex bugs in tool_scripts that require deep investigation
- API integration issues (yfinance, twstock, AKShare)
- SQLite schema or data flow problems
- Financial calculation discrepancies
- Issues requiring analysis of multiple interconnected files

## Output Location

Save to `prompts/YYYYMMDD_N_debug_description.md`

## Instructions

When this command is invoked:

### Step 1: Gather Context

Collect the following information from the conversation and codebase:

1. **Problem Summary**: What is failing? Observable symptoms?
2. **Error Messages**: Complete error output with stack traces
3. **Relevant Files**: Which files are involved?
4. **What Was Tried**: Previous attempts and why they failed

### Step 2: Build the Debug Prompt

Generate a markdown file with this structure.

**IMPORTANT**: When generating the prompt, you MUST:
1. Determine the actual filename first (e.g., `20260228_2_debug_fetcher_timeout.md`)
2. Pre-fill this filename in the "Prompt Being Reviewed" field of the response header
3. Do NOT leave it as a placeholder

```markdown
# Debug: [Brief Problem Description]

## Response Header (MANDATORY)

**Your response MUST start with this header block:**

\```
================================================================
DEBUG RESPONSE METADATA
----------------------------------------------------------------
1. Responder LLM: [Your model name, e.g., gemini-2.5-pro / claude-opus-4 / gpt-5]
2. Prompt Being Reviewed: YYYYMMDD_N_debug_<description>.md
3. Response Generated At: [YYYY-MM-DD HH:MM UTC]
================================================================
\```

**Note**: Field 2 ("Prompt Being Reviewed") is pre-filled with the actual filename of this prompt.

## Problem Summary
[What is failing, observable symptoms]

## Error Message
\```
[Complete error output]
\```

## Folder Structure
\```
[Relevant directory tree]
\```

## Environment Info
- OS: [...]
- Python: [...]
- Key packages: [yfinance/twstock/AKShare versions]

## Current Code
**File**: `path/to/file.py` (lines X-Y)
\```python
[Relevant code snippet]
\```

## What Happens
1. [Step 1]
2. [Step 2]
3. [Failure occurs]

## What I've Tried
### Attempt 1: [Description]
\```python
[Code tried]
\```
**Why this doesn't work**: [Factual observation, not speculation]

## Questions
1. [Specific question about behavior]
2. [Question about architectural pattern]

## Expected Behavior
[What should happen instead]

## Additional Context
[Logs, dependencies, system info]
```

### Step 3: Report

Tell the user:
1. The debug prompt file path
2. Summary of what's included
3. Suggest which LLM to use (Claude for code analysis, Gemini for broad knowledge)

---

## CRITICAL RULES

1. **DO NOT include guesses or assumptions about the root cause**
2. **DO NOT propose solutions or fixes in the debug prompt**
3. **ONLY provide factual information**: code, errors, observations, and questions
4. The goal is to gather complete context for another LLM to analyze independently
