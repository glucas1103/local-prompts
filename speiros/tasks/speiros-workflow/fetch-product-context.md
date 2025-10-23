# Task: Fetch Product Context

## Objective

Extract relevant product context from context/product/ folder based on brief analysis.

## Language Requirements

All generated content must be written in English.

## Inputs

- `roadmap/briefs/brief-{id}.md`
- `context/product/**/*.md`

## Steps

1. **Analyze brief thoroughly**
   - Read complete brief
   - Identify feature domain (auth, admin, dashboard, etc.)
   - Note mentioned features or concepts

2. **Scan context/product/ structure**
   - List all available product context files
   - Identify potentially relevant files

3. **Read relevant context files**
   - Read files matching brief domain
   - Extract pertinent information
   - Note existing features, user flows, business rules

4. **Create context summary**
   - Write artifact to `.speiros/artifacts/speiros-workflow/briefs/brief-{id}-product-context.md`
   - Structure:
     - **Relevant Features**: Existing features related to brief
     - **User Flows**: Current user journeys impacted
     - **Business Rules**: Constraints or rules to respect
     - **Context Sources**: List of product-context files used

## Output

- `.speiros/artifacts/speiros-workflow/briefs/brief-{id}-product-context.md`

## Success Criteria

- All relevant product context extracted
- Summary is focused and concise
- No unnecessary information included
- Ready for functional story creation
