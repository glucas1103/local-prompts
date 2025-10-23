# Task: Cleanup Technical Artifacts

## Objective

Remove temporary artifacts after successful population of final technical context deliverables.

## Language Requirements

All generated content must be written in English.

## Inputs

- `context/technical/**/*.md` (final deliverables successfully created)
- `.speiros/artifacts/audit-technical/` (temporary artifacts to remove)

## Steps

1. **Verify final deliverables exist**
   - Check that `context/technical/` directory exists
   - Verify that at least one technical domain documentation file exists
   - Confirm `context/technical/README.md` (index) exists

2. **List artifacts to remove**
   - `.speiros/artifacts/audit-technical/initial-summary.md`
   - `.speiros/artifacts/audit-technical/audit-results.md`
   - `.speiros/artifacts/audit-technical/enrichment-qa.md`
   - `.speiros/artifacts/audit-technical/structure-plan.md`
   - `.speiros/artifacts/audit-technical/research/` (entire directory)
   - Any other files in `.speiros/artifacts/audit-technical/`

3. **Remove temporary artifacts**
   - Delete all files and subdirectories in `.speiros/artifacts/audit-technical/`
   - Remove the entire `.speiros/artifacts/audit-technical/` directory
   - Ensure no temporary files remain

4. **Verify cleanup**
   - Confirm `.speiros/artifacts/audit-technical/` no longer exists
   - Verify final deliverables in `context/technical/` are still intact

## Output

- Cleanup confirmation
- Removed temporary artifacts

## Success Criteria

- All temporary artifacts removed
- Final deliverables in `context/technical/` remain untouched
- No orphaned files in `.speiros/artifacts/audit-technical/`
- Workflow completed cleanly

## Notes

- This task should only run after successful completion of `populate-technical-context`
- Final deliverables in `context/technical/` contain all necessary information
- Temporary artifacts are no longer needed once final documentation is generated
