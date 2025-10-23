# Task: Delegate Technical Context

## Objective

For each technical domain, delegate deep codebase research to sub-context-agent.

## Language Requirements

All generated content must be written in English.

## Inputs

- `.speiros/artifacts/audit-technical/structure-plan.md`

## Steps

1. **Load structure plan**
   - Read all technical domains and research assignments
   - Prioritize by: high → medium → low

2. **For each technical domain** (iterative):

   a. **Prepare delegation**
   - Extract domain name and technical scope
   - Identify: key search terms, target directories
   - Define: what to find (patterns, configs, APIs, tests)

   b. **Invoke sub-context-agent**
   - Pass: domain name, search scope
   - Command: `@sub-context-agent *search {domain}`
   - Provide: template to fill, files to examine

   c. **Monitor research**
   - Sub-agent searches codebase
   - Extracts: code patterns, configurations, examples, tests
   - Generates: intermediate research report

   d. **Collect results**
   - Save to `.speiros/artifacts/audit-technical/research/{category}/{topic}.md`
   - Examples:
     - `.../research/context/API.md`
     - `.../research/tools/CLERK.md`
     - `.../research/memory/TECHNICAL-DECISIONS.md`
   - Verify: sufficient technical detail captured
   - Note: any remaining gaps

3. **Track progress**
   - Maintain delegation log
   - Mark domains: completed, in-progress, blocked
   - Escalate: if sub-agent stuck

4. **Consolidate findings**
   - Review all research files
   - Ensure: no domain left behind
   - Prepare: for template population phase

## Output

- `.speiros/artifacts/audit-technical/research/**/*.md` (multiple files)
- Delegation log (progress tracking)

## Success Criteria

- All technical domains researched
- Research files contain concrete technical findings
- Code examples, configs, and references included
- Ready for template population
