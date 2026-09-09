---
name: 21st-ui-explore
description: Explore and compare multiple meaningfully different UI directions grounded in the current project's design system and 21st inspiration. Use when the user wants options, variants, concepts, a redesign direction, visual experimentation, or is unsure how a new interface should look. Trigger for requests such as "show me three directions", "explore alternatives", "what could this page look like", or "generate variants". Do not use for a straightforward implementation with an already-selected direction.
---

# Explore grounded UI directions

Create real choices without abandoning the project's visual identity.

## Workflow

1. Read `.21st/design.json` and the relevant product UI. If context is missing,
   run `21st init --design-context`.
2. Search 21st for multiple relevant references before generating:

   ```bash
   21st search "<interface and product context>" --context auto
   ```

3. Define three named directions. Each direction must differ on at least two
   meaningful axes such as hierarchy, information density, navigation model,
   content emphasis, interaction pattern, or composition. Color-only variants
   do not count.
4. Keep shared constraints fixed: real stack, tokens, brand assets, required
   content, accessibility, and responsive behavior.
5. Create comparable previews. Check MCP `get_usage.aiGenerationEnabled` or
   run `21st usage` and read the `21st AI generation` line. CLI 1.17.1+ also
   supports `21st usage --json`. Only when AI is explicitly enabled, use hosted
   generation with available AI credits:

   ```bash
   21st generate "<goal plus fixed constraints>" --context auto --variants 3
   ```

   Otherwise, implement the previews with your own coding agent, grounded in
   `21st search` and `21st get` (MCP: `search` and `get_component`). Do not call
   or suggest hosted `generate` or `iterate_generation` without AI access.

6. Present the options together when the host supports a picker. Otherwise give
   each option a preview/deep link and a compact comparison.
7. Recommend one direction with concrete tradeoffs, but let the user choose.
8. After selection, remove abandoned local variants and record the decision in
   `.21st/design.json`.

## Direction contract

For every direction provide:

- a short, descriptive name;
- the core idea and intended user effect;
- the 21st/project references used;
- what stays consistent with the project;
- two or more meaningful differences;
- accessibility or responsive risks;
- the best-fit scenario.

Do not generate random style mutations. Every difference must support a product
or usability hypothesis that a user can evaluate.
