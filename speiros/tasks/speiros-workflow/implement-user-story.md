# Task: Implement User Story

## Objective

Implement complete user story following the detailed implementation plan.

## Language Requirements

All generated content must be written in English.

## Inputs

- `roadmap/user-stories/story-{id}.md` (complete: Product + Spec sections)

## Steps

1. **Read complete story**
   - Read Product section: understand functional goals
   - Read Spec section: understand implementation plan
   - Review acceptance criteria
   - Study implementation plan tasks

2. **Execute implementation plan sequentially**
   - Follow each task in order
   - For each task:
     - Implement code changes
     - Write tests immediately
     - Verify tests pass
     - Mark task complete before moving to next

3. **Follow Technical Context guidelines**
   - Respect patterns documented in story
   - Follow conventions noted in story
   - Use examples provided in Technical Context section

4. **Handle all edge cases**
   - Implement error handling
   - Validate inputs
   - Handle loading states (frontend)
   - Add appropriate logging

5. **Write comprehensive tests**
   - Unit tests for new functions/components
   - Integration tests for API endpoints
   - Follow testing patterns from Technical Context

6. **Run all tests**
   - Execute test suite
   - Fix any failing tests
   - Ensure no regressions

7. **Document Implementation Results**
   - Update story file with Dev Results section:
     - Summary of what was implemented
     - List of files created
     - List of files modified
     - Test coverage added
     - Any deviations from plan (with rationale)

8. **Update story status**
   - Set Status: "Done"

## Output

- Source code files (created/modified)
- Test files (created/modified)
- `roadmap/user-stories/story-{id}.md` (Dev Results section filled)

## Success Criteria

- All acceptance criteria met
- All implementation plan tasks completed
- All tests passing
- Code follows conventions from Technical Context
- Dev Results section complete
- Ready for memory update
