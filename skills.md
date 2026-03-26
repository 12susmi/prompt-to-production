# UC-0B — skills.md
## Skills: Policy Summariser Agent

### Skill 1 — Document Ingestion
- **Input**: Plain-text `.txt` policy file
- **Action**: Read entire file; detect section headings and numbered clauses using regex patterns
- **Output**: Structured list of `(section_id, heading, body)` tuples
- **Guard**: Raise an error if the file is empty or unreadable

### Skill 2 — Clause Inventory
- **Input**: Structured clause list from Skill 1
- **Action**: Build an explicit inventory of every numbered clause found in the source document
- **Output**: `clause_inventory[]` — a checklist used for completeness verification
- **Guard**: Log a warning if fewer than 3 clauses are detected (likely a parsing error)

### Skill 3 — Faithful Summarisation (LLM call)
- **Input**: Full document text + clause inventory
- **Action**: Call the LLM with a strict prompt that includes:
  - The full source text
  - The clause inventory (to enforce coverage)
  - Explicit rules: no omission, no meaning change, no softening, single source
- **Output**: Draft summary text
- **Guard**: Prompt instructs the model to include a `[CLAUSE COVERAGE]` section listing every clause it has summarised

### Skill 4 — Completeness Verification
- **Input**: Draft summary + clause inventory
- **Action**: Check each clause ID from the inventory appears in the summary; flag any missing ones
- **Output**: Verification report with pass/fail per clause
- **Guard**: If any clause is missing, re-prompt the model with the missing clause list and request a corrected summary

### Skill 5 — Output Writer
- **Input**: Verified summary text
- **Action**: Write the summary to `summary_[policy_name].txt` in the output directory
- **Output**: `.txt` file on disk
- **Guard**: Confirm file is non-empty after write

### Skill 6 — CRAFT Loop Logger
- **Input**: Each run's prompt, raw LLM output, verification report
- **Action**: Append a structured log entry to `craft_log.txt` with timestamp, failure type detected, and fix applied
- **Output**: Audit trail for tutor review
