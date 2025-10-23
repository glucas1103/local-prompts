# Task: Create Technical Story

## Objective

Enrich user story with self-contained Implementation Plan section.

## Language Requirements

All generated content must be written in English.

## Inputs

- `roadmap/user-stories/story-{id}.md` (Product section)
- `.speiros/artifacts/speiros-workflow/user-stories/story-{id}-technical-context.md`
- `.speiros/templates/user-story-speiros.yaml`

## Steps

1. **Read complete functional story**
   - Understand all acceptance criteria
   - Note functional requirements

2. **Review technical context summary**
   - Understand applicable patterns
   - Note conventions and standards

3. **Write Technical Analysis**
   - Analyze implications of functional requirements
   - Identify technical challenges
   - Note complexity areas

4. **Create Technical Context section**
   - CRITICAL: Self-contained (no references to context/technical/)
   - Synthesize information from story-{id}-technical-context.md
   - Include all necessary technical background inline
   - Document patterns, conventions, examples to follow

5. **Elicit Implementation Plan**
   - Present draft plan to user
   - Ask: "Review this implementation plan - is it complete and correct?"
   - Wait for feedback
   - Adjust based on feedback
   - Structure as ordered task list:
     - Each task numbered and clear
     - Include what to create/modify
     - Reference acceptance criteria
     - Specify test requirements
     - Note error handling needs

6. **List Files to Modify/Create**
   - List all impacted files
   - Note: new files vs modifications
   - Include test files

7. **Identify Points of Attention**
   - List technical risks
   - Note edge cases
   - Warn about breaking changes
   - Mention performance considerations

8. **Update story status**
   - Set Status: "Spec-Ready"
   - Mark Spec section as complete

## Output

- `roadmap/user-stories/story-{id}.md` (Implementation Plan section filled)

## Success Criteria

- Story is self-contained (DevMan needs no external docs)
- Technical Context includes all necessary patterns/conventions
- Implementation Plan is detailed and actionable
- All files listed
- Ready for handoff to DevMan
- User has validated implementation approach
