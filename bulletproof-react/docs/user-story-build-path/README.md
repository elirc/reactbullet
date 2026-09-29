# User Story Build Path

This path is a practical ticket ladder for `apps/react-vite`. You will build real features against real patterns already used in this codebase.

## How To Use These Stories

1. Do stories in order.
2. Before coding, open every file listed in “Files You’ll Likely Touch.”
3. Implement with smallest safe diff first.
4. Validate acceptance criteria one by one.
5. Write down one lesson and one unresolved question after each story.

## Difficulty Progression

1. Stories 1-3: UI edits on existing flows.
2. Stories 4-5: New UI behavior over existing data contracts.
3. Stories 6-7: Add endpoint or state mechanism.
4. Stories 8-9: Full-stack feature across UI + handlers + persistence.
5. Story 10: Architecture-level decision with performance/security tradeoffs.

## How To Know A Story Is Done

1. Every acceptance criterion can be verified manually or via test.
2. You can explain the end-to-end flow with file-level evidence.
3. Existing behavior outside scope still works.
4. Your change includes appropriate cache invalidation and policy checks.

## Getting Unstuck Without Getting Spoiled

Use this prompt with Claude/ChatGPT:

`I'm working on Story X. I'm stuck on Y. Here is what I've tried: Z. Don't give me the solution — ask me questions that help me figure it out.`

Tips:

1. Include exact error text and file/line references.
2. Ask for debugging questions, not code.
3. After answering the questions, try one new hypothesis before asking again.

