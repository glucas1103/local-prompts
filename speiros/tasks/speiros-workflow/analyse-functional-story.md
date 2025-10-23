# Task: Analyse Functional Story

## Objective

Analyze functional story to identify technical requirements and needs.

## Language Requirements

All generated content must be written in English.

## Inputs

- `roadmap/user-stories/story-{id}.md` (Product section)

## Steps

1. **Read functional story completely**
   - Understand Description, Use Cases, Acceptance Criteria
   - Identify functional requirements

2. **Identify technical domains**
   - Frontend changes needed?
   - Backend/API changes needed?
   - Database schema changes?
   - New dependencies or services?

3. **List technical questions**
   - What patterns apply?
   - What existing code to modify?
   - What new components/modules needed?
   - Testing requirements?

4. **Create analysis document**
   - Write to `.speiros/artifacts/speiros-workflow/user-stories/story-{id}-analysis.md`
   - Structure:
     - **Functional Summary**: Brief recap
     - **Technical Domains**: List impacted areas
     - **Key Requirements**: Technical needs extracted
     - **Questions for Context**: What to look for in context/technical/

## Output

- `.speiros/artifacts/speiros-workflow/user-stories/story-{id}-analysis.md`

## Success Criteria

- All functional requirements understood
- Technical domains identified
- Clear list of what to fetch from context/technical/
- Ready for technical context extraction
