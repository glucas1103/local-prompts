# Task: Fetch Technical Context

## Objective

Extract relevant technical context from context/technical/ folder based on story analysis.

## Language Requirements

All generated content must be written in English.

## Inputs

- `roadmap/user-stories/story-{id}.md`
- `.speiros/artifacts/speiros-workflow/user-stories/story-{id}-analysis.md`
- `context/technical/**/*.md`

## Steps

1. **Review analysis**
   - Read story-{id}-analysis.md
   - Identify required technical domains
   - Note specific patterns or tools mentioned

2. **Scan context/technical/ structure**
   - List all available technical context files
   - Identify relevant files based on domains

3. **Read relevant context files**
   - Read files matching technical domains
   - Extract: patterns, conventions, architecture decisions
   - Note: code examples, testing patterns, best practices

4. **Create technical context summary**
   - Write to `.speiros/artifacts/speiros-workflow/user-stories/story-{id}-technical-context.md`
   - Structure:
     - **Architecture Patterns**: Relevant patterns to follow
     - **Code Conventions**: Naming, structure conventions
     - **Testing Requirements**: Test patterns and standards
     - **Dependencies**: Existing libraries/tools to use
     - **Related Code**: Existing code to reference or modify
     - **Context Sources**: List of technical-context files used

## Output

- `.speiros/artifacts/speiros-workflow/user-stories/story-{id}-technical-context.md`

## Success Criteria

- All relevant technical context extracted
- Patterns and conventions clearly documented
- Focused on implementation needs
- Ready for technical story creation
