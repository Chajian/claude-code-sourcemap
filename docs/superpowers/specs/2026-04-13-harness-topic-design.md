# Harness Topic Design

## Summary

This design defines a standalone interactive topic page for `Harness`, aimed at ordinary AI practitioners who consume content rather than read repository internals. The page should explain what `Harness` is, why Agent stability depends more on it than on the model itself, and where `Prompt Engineering`, `Context Engineering`, and `Harness Engineering` differ.

The primary output is a single-file interactive HTML topic page under `docs/topics/harness/`. Its experience should feel like a guided explainer rather than a static article or a module catalog.

## Goals

- Help readers clearly understand what `Harness` means in the Agent context.
- Convince readers that many Agent stability problems are system problems, not primarily model problems.
- Use a simple progressive narrative:
  - `Prompt` solves "say the task clearly"
  - `Context` solves "give the right information"
  - `Harness` solves "keep doing the right thing in real execution"
- Keep the page readable for ordinary AI practitioners without assuming they want source-level details.
- Use strong, intuitive animation to show where `Prompt` and `Context` stop being enough.

## Non-Goals

- This page is not a full repository maintainer document.
- This page is not a complete six-module engineering handbook.
- This page is not a framework comparison or tool directory.
- This page should not become a generic AI glossary page.

## Audience

Primary audience: ordinary AI practitioners who want to understand Agent ideas quickly and visually.

Reader profile:

- Follows AI and Agent trends
- Understands models, prompts, tools, and workflows at a high level
- Does not necessarily want to read repository architecture docs
- Wants a strong conceptual takeaway with a few concrete examples

## Core Message

The page should drive one central judgment:

> The hard part of Agents is shifting from "make the model look smart" to "make the system finish work reliably."

Supporting framing:

- `Prompt Engineering` improves how tasks are expressed
- `Context Engineering` improves what the model sees
- `Harness Engineering` improves whether the system can keep acting correctly in real execution

## Format

The page should be a standalone interactive topic page, not a Markdown article.

Recommended location:

- `docs/topics/harness/index.html`

Recommended supporting integration:

- add a topic entry from `docs/README.md`
- do not require a new top-level `README.md` link in the first implementation pass

## Experience Direction

### Narrative Style

Use the approved `C` direction as the primary structure:

- a concept evolution chain:
  - `Prompt`
  - `Context`
  - `Harness`

But do not present it as a neutral taxonomy. The tone should remain plainspoken with clear judgment. The user-approved tone is:

- accessible
- sharp
- opinionated without becoming abstract

This means the page should not merely define terms. It should show why the first two layers are insufficient once execution enters the real world.

### Reading Mode

The page should behave like a guided interactive explainer:

- users advance step by step
- each step answers one question
- animations exist to clarify the boundary between concepts
- the page should still be understandable in a linear pass without requiring complex interaction logic

## Information Architecture

The page should follow this seven-beat sequence.

### 1. Opening Judgment

Purpose:

- pull readers into the problem before introducing terminology

Message:

- many Agents look smart but become unstable when asked to actually do things

Visual goal:

- show a task appearing to run, then visibly drifting or destabilizing

### 2. Prompt Stage

Purpose:

- acknowledge the value of prompt work instead of dismissing it

Message:

- `Prompt` solves how to state the task clearly

Visual goal:

- show a short, single-turn task becoming clearer and answering correctly

### 3. Context Stage

Purpose:

- show what `Context` adds beyond prompt

Message:

- `Context` solves how to provide the right information, constraints, and background

Visual goal:

- show task context cards appearing around the model: goals, constraints, memory, current state

### 4. Boundary Break

Purpose:

- create the first strong mental shift

Message:

- once real execution starts, the problem changes layers

Visual goal:

- page state changes
- tool output comes back incomplete or unstable
- steps drift out of order
- the atmosphere should visibly shift from smooth to unstable

This is the most important transition in the page.

### 5. Example One: Browser / Tool Use

Purpose:

- demonstrate that real-world tool use creates reliability problems that are not solved by better wording alone

Message:

- knowing what to click is not the same as reliably clicking the right thing in a changing environment

Visual goal:

- a browser-like scene
- target element changes or disappears
- tool output returns with ambiguity or error
- validation, state tracking, and retry logic bring the flow back under control

### 6. Example Two: Long-Horizon Task Orchestration

Purpose:

- show why long chains expose system weaknesses

Message:

- the challenge is no longer "can the model answer step one" but "can the system survive step seven"

Visual goal:

- a task chain elongates across multiple stages
- checkpoints, state continuation, and recovery mechanisms appear progressively
- the sequence becomes stable only after those supports are added

### 7. Harness Arrival and Closing Definition

Purpose:

- land the final takeaway only after the reader has seen the need for it

Message:

- `Harness` is not a fancier prompt term
- it is the execution support system that helps Agents keep doing the right thing under real conditions

Visual goal:

- support lines converge around the execution chain:
  - state
  - validation
  - orchestration
  - recovery

Closing definition:

- `Prompt` is about saying the task clearly
- `Context` is about giving the right information
- `Harness` is about keeping the model doing the right thing in real execution

## Example Strategy

The page should use only two concrete examples, both already approved:

1. browser / tool calling
2. long-horizon task orchestration

These examples must be used as evidence of the core claim, not as product demos.

### Example Handling Rules

Do not frame the examples like:

- "look what tools can do"
- "Agent can operate a browser automatically"
- "the planning system is very intelligent"

Instead frame them like:

- "real execution introduces unstable environments"
- "tool calls create verification and recovery requirements"
- "multi-step work requires state, checkpoints, and continuation"

## Visual and Motion Direction

The page should fit the repository's existing interactive topic style:

- single-file HTML
- bold hero section
- strong visual hierarchy
- custom CSS variables
- lightweight but deliberate motion
- desktop and mobile friendly

### Motion Principles

Animation must teach, not decorate.

Use motion for:

- showing concept escalation
- showing when the problem changes layers
- contrasting "smooth answer generation" with "unstable execution"
- showing support systems progressively stabilizing the chain

Avoid:

- decorative motion that adds no explanatory value
- overly playful effects that weaken the technical argument

### Look and Feel

The approved visual tone should feel:

- clear
- modern
- readable
- confident

Suggested visual direction:

- light background with depth
- warm-to-cool transitions around the boundary break
- strong contrast for unstable vs stabilized states
- cards, flow lines, and staged panels rather than dense paragraphs

## Copy Direction

The copy should stay simple, concrete, and slightly sharp.

Approved message pattern:

- do not start with definitions
- start with the user's likely misconception
- show where that misconception breaks
- only then introduce the term `Harness`

Candidate judgment lines to preserve:

- "Many teams think they need a stronger model. Later they realize what they lacked was a system that could actually hold the model up."
- "The challenge in Agents is moving from making the model look smart to making the system finish work reliably."
- "Once execution enters the real world, the problem stops being mainly about how the model answers."

## Harness Module Exposure

The page should not open with a six-part Harness map.

Instead:

- keep the front half focused on concept evolution and boundary break
- expose the module idea lightly near the end

Allowed end-state phrasing:

- a mature Harness usually includes pieces such as state, tools, orchestration, observation, constraints, and recovery

This keeps the page from collapsing into a module catalog while still giving readers a useful mental map.

## Content Layout Recommendation

Recommended page sections:

1. Hero judgment
2. Step rail or step selector
3. Main interactive explainer stage
4. Supporting panels for what changed / why it matters / common misconception
5. Closing takeaway with lightweight Harness component reveal

## Error Handling and Interaction Design

The page interaction should remain robust even if users do not click every step.

Requirements:

- support sequential reading without interaction dead ends
- make the active step visually obvious
- ensure copy remains understandable if animations are skipped or reduced
- keep performance stable on mobile and lower-powered devices

## Testing and Verification

Verification should cover:

- desktop rendering
- mobile rendering
- step progression logic
- animation readability
- text legibility under different viewport widths
- no broken links from docs entry points

Manual success criteria:

- a reader can summarize the difference between `Prompt`, `Context`, and `Harness` after one pass
- a reader can explain why browser/tool use and long task chains expose Harness problems
- a reader leaves with the conclusion that stability is primarily a system concern

## Risks

### Risk: Too abstract

If the page stays too conceptual, readers will treat it like another terminology explainer.

Mitigation:

- keep both concrete examples
- make the boundary-break step visually obvious

### Risk: Too engineering-heavy

If the page fully expands into module-by-module teaching, it will lose the intended audience.

Mitigation:

- delay module exposure until the end
- keep examples explanatory rather than implementation-heavy

### Risk: Motion without clarity

If animations look good but do not clarify the mental shift, the page will feel flashy but shallow.

Mitigation:

- every animation must answer "what changed?" or "why did the old approach stop being enough?"

## Definition of Done

This design is complete when implementation produces:

- a standalone `Harness` topic page under `docs/topics/harness/`
- a guided interactive sequence built around the approved concept-evolution structure
- two integrated examples:
  - browser / tool use
  - long-horizon orchestration
- a strong boundary-break moment between `Context` and `Harness`
- a closing takeaway that explains why Agent stability depends on the surrounding execution system
