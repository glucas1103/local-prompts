# Task: Enrich Product Context

## Objective

Interactive Q&A session with user to fill critical information gaps.

## Language Requirements

All generated content must be written in English.

## Inputs

- `.speiros/artifacts/audit-product/audit-results.md`

## Steps

1. **Prepare questions**
   - Review gaps from audit results
   - Prioritize: business context, decisions, vision
   - Group by domain for clarity

2. **Present gap analysis to user**
   - Show identified domains
   - Explain proposed structure
   - Get user confirmation/feedback

3. **Ask targeted questions** (elicit=true)
   - For each major gap:
     - "What is the business goal of [feature]?"
     - "Who are the primary users of [domain]?"
     - "Why was [technology/approach] chosen?"
     - "What's the future vision for [area]?"
   - Keep questions focused and specific
   - Allow "unknown" or "to research" answers

4. **Document responses**
   - Write to `.speiros/artifacts/audit-product/enrichment-qa.md`
   - Structure:
     - Questions Asked (numbered)
     - User Responses (verbatim)
     - Key Insights (extracted)
     - Remaining Gaps (what still needs research)

5. **Validate completeness**
   - Review with user: anything else to add?
   - Confirm: ready to proceed with structure creation

## Output

- `.speiros/artifacts/audit-product/enrichment-qa.md`

## Success Criteria

- Critical business context captured
- User validated domain structure
- Historical decisions documented
- Clear separation: answered vs. needs-research
