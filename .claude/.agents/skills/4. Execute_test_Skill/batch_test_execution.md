---
description: Mandatory rules for writing batch test scripts. Prevents false PASS from partial coverage.
---

# Batch Test Script Execution Rules

## When Writing or Modifying Batch Test Scripts

These rules are MANDATORY and override any shortcuts or optimizations:

### 1. FULL DATA COVERAGE
- If TC defines N data values → script MUST test ALL N values individually
- Each value produces its own PASS/FAIL log entry
- NEVER test only 1 value when TC has multiple

### 2. FULL ASSERTION COVERAGE  
- If TC defines M expected results → script MUST assert ALL M conditions
- Log each assertion individually: `PASS:check_name` or `FAIL:check_name`
- NEVER collapse multiple checks into one boolean

### 3. NO SILENT SKIPS
- Every TC from manual document MUST have a test function
- If TC cannot be automated → log as `SKIP:reason`
- Final summary MUST show registered vs executed count

### 4. MODULE-SPECIFIC TESTS ARE NOT OPTIONAL
- Admin/Reception/Doctor module tests (or equivalent per ticket) MUST be included
- Use conditional: `if (mod.type !== 'XX') return`

### 5. ERROR ISOLATION
- Each TC sub-case in its own try/catch
- One failure must NOT prevent other TCs from running

### 6. COVERAGE VALIDATION
- Before declaring script "done", count:
  - TC IDs in manual doc vs TC functions in script
  - Data values per TC in manual vs in script
  - Expected results per TC in manual vs assertions in script
- If any count mismatches → script is INCOMPLETE
