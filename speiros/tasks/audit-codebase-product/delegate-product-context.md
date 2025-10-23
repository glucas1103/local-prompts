# Task: Delegate Product Context

## Objective

For each domain, delegate deep codebase research to sub-context-agent.

## Language Requirements

All generated content must be written in English.

## Inputs

- `.speiros/artifacts/audit-product/structure-plan.md`

## Steps

1. **Load structure plan**
   - Read all domains and research assignments
   - Prioritize by: high → medium → low

2. **For each domain** (iterative):

   a. **Prepare delegation**
   - Extract domain name and scope
   - Identify: key search terms, target directories
   - Define: what to find (use cases, flows, rules)

   b. **Invoke sub-context-agent**
   - Pass: domain name, search scope
   - Command: `@sub-context-agent *search {domain}`
   - Provide: template to fill, files to examine

   c. **Monitor research**
   - Sub-agent searches codebase
   - Extracts: code patterns, examples, tests
   - Generates: intermediate research report

   d. **Collect results**
   - Save to `.speiros/artifacts/audit-product/research/{domain}/research-{topic}.md`
   - Verify: sufficient detail captured
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

- `.speiros/artifacts/audit-product/research/{domain}/research-*.md` (multiple files)
- Delegation log (progress tracking)

## Success Criteria

- All domains researched
- Research files contain concrete findings
- Code examples and references included
- Ready for template population
