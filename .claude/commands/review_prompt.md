# 審查提示

Generate a structured prompt to ask an external LLM to review code implementation for logical errors, missing updates, and feasibility issues.

## Usage

```
/review_prompt <description_of_work>
```

Example:
```
/review_prompt market data fetcher implementation for China stocks
```

## Instructions

When this command is invoked:

### Step 1: Gather Context

1. **Identify the work to review** from the user's description
2. **Find relevant files**:
   - Check git diff for recent changes related to the work
   - Identify modified Python scripts in tool_scripts/
   - Find any related plan files in `prompts/` folder

### Step 2: Collect Code Diffs

For each modified file:
1. Get the git diff showing the changes
2. Extract key code snippets that are critical to the implementation
3. Note the data structures, API patterns, and file I/O patterns

### Step 3: Generate the Review Prompt

Create a markdown file in `prompts/` folder with naming convention:
```
prompts/YYYYMMDD_N_review_<description>.md
```

**IMPORTANT**: When generating the prompt, you MUST:
1. Determine the actual filename first (e.g., `20260228_3_review_fetcher_impl.md`)
2. Pre-fill this filename in the "Prompt Being Reviewed" field of the response header

#### Required Sections:

```markdown
# Review Request: [Title]

## Task Objective
[Clear description of what was implemented and the goal]

## Files Modified/Created
| File | Type | Purpose |
|------|------|---------|
| ... | ... | ... |

## Code Diffs
[Include all relevant code diffs with explanations]

## Review Questions

### 1. Logical Errors
- [ ] Question 1
- [ ] Question 2

### 2. API Abstraction Compliance
- [ ] Does it follow the fetcher_base.py interface?
- [ ] Are market-specific quirks properly handled?

### 3. Data Integrity
- [ ] Are financial calculations correct?
- [ ] Are data types consistent across markets?

### 4. Security
- [ ] No API keys in code?
- [ ] No hardcoded credentials?

### 5. CLAUDE.md Compliance
- [ ] Direct dict access (no unnecessary .get())?
- [ ] Fail-fast error handling?
- [ ] No over-engineering?

## Important Instructions

### DO NOT Modify Code
**You must NOT modify any code files.** This is a review-only task.

### Response Header (MANDATORY)
**Your response MUST start with this header block:**

\```
================================================================
REVIEW RESPONSE METADATA
----------------------------------------------------------------
1. Reviewer LLM: [Your model name]
2. Prompt Being Reviewed: YYYYMMDD_N_review_<description>.md
3. Response Generated At: [YYYY-MM-DD HH:MM UTC]
================================================================
\```

### Report Language
**整份 Review 報告必須使用繁體中文撰寫。** 包括：
- Summary（摘要）
- Issues Found（發現的問題）
- Data Flow Analysis（資料流分析）
- Recommendations（建議）

唯一例外：程式碼片段、檔案路徑、變數名稱、英文專有名詞保留原文。

### Output Your Review
Write your review results to a new file in the `prompts/` folder:
`prompts/YYYYMMDD_ID_<topic>_review_result.md`
```

### Step 4: Report to User

Tell the user:
1. The review prompt file path created
2. Summary of what's included (files reviewed)
3. Suggest which LLM to use for the review
