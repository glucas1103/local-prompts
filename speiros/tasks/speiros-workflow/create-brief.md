# Task: Create Brief

## Objective

Transform user prompt into structured brief document.

## Language Requirements

All generated content must be written in English.

## Inputs

- User prompt/request
- `.speiros/templates/speiros-workflow/brief-tmpl.yaml`

## Steps

1. **Parse user request**
   - Read and understand user's request
   - Identify main objective
   - Extract context clues

2. **Fill Metadata section**
   - Generate brief ID (format: brief-XXX)
   - Create descriptive title
   - Set creation date
   - Set status to "Draft"

3. **Transcribe Original Request**
   - Copy user prompt verbatim
   - Preserve exact wording

4. **Reformulate request**
   - Rewrite professionally and clearly
   - Structure into coherent paragraphs
   - Remove ambiguity

5. **Elicit Objective**
   - Ask: "What is the main objective of this request?"
   - Wait for user response
   - Record answer

6. **Extract Context**
   - Identify business or user context
   - Note relevant background information

7. **Elicit Scope**
   - Ask: "What is IN and OUT scope?"
   - Wait for user response
   - Record IN/OUT scope clearly

8. **Add Next Step**
   - Note: "This brief will be analyzed by ProductMan to create a functional user story."

## Output

- `roadmap/briefs/brief-{id}.md`

## Success Criteria

- Brief follows template structure
- All elicitations completed
- Clear objective and scope defined
- Ready for product context analysis
