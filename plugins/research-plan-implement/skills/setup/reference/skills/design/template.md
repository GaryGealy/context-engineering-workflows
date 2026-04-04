# Design: [Feature/Task Name]

## Current State
[What exists today, 3-5 sentences. Enough to ground the conversation, not a full audit.]

## Desired End State
[What does "done" look like? How will we know it works? This is the spec, not the implementation.]

## Patterns to Follow
[Which existing codebase patterns should we model after, with file:line references.]
[This is where you catch the agent following the wrong example before it writes code.]

## Testing Approach
[How should we verify this works? Adapt to the project:]

[If the codebase has good test infrastructure:]
- Suggest specific test types with examples from the codebase
- Reference existing test patterns found in research

[If it's API work:]
- Suggest conformance-style testing: define expected behavior, test against it
- Reference endpoints that could be exercised via curl or similar

[If it's UI work:]
- Suggest manual testing steps the agent can help execute
- Reference any existing UI test patterns

[If the codebase has minimal tests:]
- Frame as: "What would give you confidence this works?"
- Help the user think through verification strategies

[Always:]
- Surface what test patterns already exist in the project
- Suggest the testing approach that fits the shape of this work

## Key Decisions
[Resolved design choices with brief rationale. Things like:]
- "Using the existing queue system rather than adding a new dependency because X"
- "Following the v2 auth pattern, not the legacy one, because Y"

## Open Questions
[Anything you're unsure about — these will be walked through interactively.]

## What We're NOT Doing
[Explicit scope boundaries to prevent creep.]
