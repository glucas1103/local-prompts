# Task: Enrich Context from User Story

## Objective

Enrich technical context with decisions from completed user story using dynamic structure analysis.

## Language Requirements

All generated content must be written in English.

## Inputs

- `roadmap/user-stories/story-{id}.md` (completed with Dev Results)

## Steps

1. **Read Dev Results section**
   - Focus on "Technical Decisions" subsection
   - Read "Context Impact" for guidance
   - Identify all decisions to document

2. **Scan existing context structure**
   - List files in `context/technical/context/` and `context/technical/tools/`
   - For each relevant file: read structure and identify sections
   - Note: Do NOT use fixed categories - adapt to existing structure

3. **Map decisions to context files**
   - Match each decision to appropriate context file(s)
   - Identify best section within file (e.g., "Historical Decisions", "Patterns", etc.)
   - If no section exists, create "Historical Decisions"

4. **Enrich context files**
   - Add decision entry to appropriate section
   - Include: decision summary, rationale, impact, user story reference, date
   - Follow existing format and style of the file

5. **Validate enrichment**
   - Check bidirectional references (User Story ↔ Context)
   - Verify no duplication
   - Ensure markdown validity

## Output

- Updated context files in `context/technical/`
- Each with new entries referencing source user story

## Success Criteria

- All decisions from Dev Results reflected in context
- Sections identified dynamically (not from fixed list)
- Files maintain existing structure
- Bidirectional references work
- No duplication
