# Task: Create Functional Story

## Objective

Create self-contained functional user story (Product section only).

## Language Requirements

All generated content must be written in English.

## Inputs

- `roadmap/briefs/brief-{id}.md`
- `.speiros/artifacts/speiros-workflow/briefs/brief-{id}-product-context.md`
- `.speiros/templates/user-story-speiros.yaml`

## Steps

1. **Create story file**
   - Generate story ID (format: story-XXX)
   - Create file: `roadmap/user-stories/story-{id}.md`

2. **Fill Metadata**
   - ID, Title, Epic (if applicable)
   - Status: "Product-Ready"
   - Priority: based on brief
   - Estimation: initial estimate

3. **Write Description**
   - Format: "As a [who], I want [what], so that [why]"
   - Make it clear and user-centric

4. **Create Functional Context**
   - CRITICAL: Self-contained (no references to context/product/)
   - Synthesize information from brief-{id}-product-context.md
   - Include all necessary functional background
   - Describe related existing features inline

5. **Elicit Use Cases**
   - Ask: "Describe the main use cases for this feature"
   - Wait for user response
   - Document use cases clearly

6. **Elicit Acceptance Criteria**
   - Ask: "What are the functional acceptance criteria?"
   - Wait for user response
   - List as testable criteria
   - Number each criterion

7. **Mark sections**
   - Clearly mark Product section as complete
   - Leave Spec section empty for SpecMan

## Output

- `roadmap/user-stories/story-{id}.md` (Product section filled)

## Success Criteria

- Story is self-contained (no external references needed)
- Functional Context includes all necessary background
- Acceptance criteria are clear and testable
- Ready for handoff to SpecMan
- User has validated use cases and criteria
