# Task: Cleanup Product Artifacts

## Objective

Remove temporary artifacts after successful population of final product context deliverables.

## Language Requirements

All generated content must be written in English.

## Inputs

- `context/product/**/*.md` (final deliverables successfully created)
- `.speiros/artifacts/audit-product/` (temporary artifacts to remove)

## Steps

1. **Verify final deliverables exist**
   - Check that `context/product/` directory exists
   - Verify that at least one domain documentation file exists
   - Confirm `context/product/README.md` (index) exists

2. **List artifacts to remove**
   - `.speiros/artifacts/audit-product/initial-summary.md`
   - `.speiros/artifacts/audit-product/audit-results.md`
   - `.speiros/artifacts/audit-product/enrichment-qa.md`
   - `.speiros/artifacts/audit-product/structure-plan.md`
   - `.speiros/artifacts/audit-product/research/` (entire directory)
   - Any other files in `.speiros/artifacts/audit-product/`

3. **Remove temporary artifacts**
   - Delete all files and subdirectories in `.speiros/artifacts/audit-product/`
   - Remove the entire `.speiros/artifacts/audit-product/` directory
   - Ensure no temporary files remain

4. **Verify cleanup**
   - Confirm `.speiros/artifacts/audit-product/` no longer exists
   - Verify final deliverables in `context/product/` are still intact

## Output

- Cleanup confirmation
- Removed temporary artifacts

## Success Criteria

- All temporary artifacts removed
- Final deliverables in `context/product/` remain untouched
- No orphaned files in `.speiros/artifacts/audit-product/`
- Workflow completed cleanly

## Notes

- This task should only run after successful completion of `populate-product-context`
- Final deliverables in `context/product/` contain all necessary information
- Temporary artifacts are no longer needed once final documentation is generated
