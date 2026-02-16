---
name: ideation-helper
description: Transform raw brain dumps (dictated freestyle) into structured project plans. Use when you have messy ideas, scattered thoughts, or dictated stream-of-consciousness about something you want to accomplish. Produces project briefs and action steps written to ./docs/ideation/{project-name}/.
---

# Ideation

Transform unstructured brain dumps into structured, actionable project artifacts through a confidence-gated workflow.

## Critical: Use AskUserQuestion Tool

**ALWAYS use the `AskUserQuestion` tool when asking clarifying questions.** Do not ask questions in plain text. The tool provides structured options and ensures the user can respond clearly.

Use `AskUserQuestion` for:

- Clarifying questions during confidence scoring (Phase 2)
- Project name confirmation before writing artifacts
- Brief approval
- Workflow choice (plans vs straight to action steps)
- Phase review feedback before action steps generation
- Any decision point requiring user input

## Workflow Pipeline

```
INTAKE → CONTEXT GATHERING → BRIEF FORMATION → PHASING → ACTION STEPS GENERATION → HANDOFF
              ↓                      ↓                ↓            ↓                  ↓
         Scan files &          confidence < 95%?    Plans or    Repeatable?       Analyze deps
         ask questions              ↓               straight      ↓                  ↓
                               ASK QUESTIONS        to action   Template +       Sequential?
                                    ↓               steps?      per-phase        Parallel?
                               (loop until ≥95%)                deltas           Agent Team?
```

## Phase 1: Intake

Accept whatever the user provides:

- Scattered thoughts and half-formed ideas
- Voice dictation transcripts (messy, stream-of-consciousness)
- Bullet points mixed with rambling
- Topic jumping and tangents
- Contradictions and unclear statements
- Technical jargon mixed with vague descriptions

**Don't require organization. The mess is the input.**

Acknowledge receipt and begin analysis. Do not ask for clarification yet.

## Phase 2: Context Gathering

**Before scoring confidence or generating any artifacts, understand the existing situation.** This is critical — action steps written without understanding the current state, constraints, and prior work will be generic and wrong.

### 2.1 Determine if Context Gathering is Needed

Context gathering is needed when:

- The brain dump references existing work, systems, or prior progress
- The project directory contains existing files, notes, or documents
- The user mentions extending or building on something that already exists

Skip context gathering for brand-new projects with no prior work.

### 2.2 Scan Existing Materials

Use `Glob`/`Grep`/`Read` or the `Task` tool with `subagent_type: "Explore"` to understand:

1. **Existing materials** — Are there notes, documents, spreadsheets, or prior plans?
2. **Current state** — What has already been done? What's in progress?
3. **Constraints** — Budget documents, timelines, requirements from others?
4. **Related work** — Similar projects or templates that could inform this one?

### 2.3 Ask About Context

In addition to scanning files, use `AskUserQuestion` to ask the user about their situation:

- What resources do they have (budget, time, help from others)?
- What constraints exist (deadlines, limitations, dependencies on others)?
- What has already been tried or started?
- What similar projects have they done before?

Structure these as `AskUserQuestion` calls with options where possible.

### 2.4 Record Findings

Retain context for use in later phases. These inform:

- **Confidence scoring** — better understanding of the problem space
- **Brief** — realistic scope boundaries based on actual constraints
- **Action steps** — "Similar to" references, accurate resource lists, practical approaches

**Do not write context findings to files.** They're context for the ideation process, not an artifact.

## Phase 3: Brief Formation

### 3.1 Analyze the Brain Dump

Extract from the raw input + context gathering:

1. **Problem signals**: What pain point or need is being described?
2. **Goal signals**: What does the user want to achieve?
3. **Success signals**: How will they know it worked?
4. **Scope signals**: What's included? What's explicitly excluded?
5. **Contradictions**: Note any conflicting statements
6. **Practical constraints**: What does the current situation enable or limit?

### 3.2 Calculate Confidence Score

Read `references/confidence-rubric.md` for detailed scoring criteria.

**Score conservatively.** When uncertain between two levels, choose the lower one. One extra round of questions costs minutes; a bad brief costs hours. Do not inflate scores to move forward faster.

Score each dimension (0-20 points):

| Dimension        | Question                                                       |
| ---------------- | -------------------------------------------------------------- |
| Problem Clarity  | Do I understand what problem we're solving and why it matters? |
| Goal Definition  | Are the goals specific and measurable?                         |
| Success Criteria | Can I write a checklist to verify "done"?                      |
| Scope Boundaries | Do I know what's in and out of scope?                          |
| Consistency      | Are there contradictions I need resolved?                      |

**Total: /100 points**

### 3.3 Confidence Thresholds

| Score | Action                                                    |
| ----- | --------------------------------------------------------- |
| < 70  | Major gaps. Ask 5+ questions targeting lowest dimensions. |
| 70-84 | Moderate gaps. Ask 3-5 targeted questions.                |
| 85-94 | Minor gaps. Ask 1-2 specific questions.                   |
| ≥ 95  | Ready to generate brief.                                  |

### 3.4 Ask Clarifying Questions

When confidence < 95%, **MUST use `AskUserQuestion` tool** to ask clarifying questions. Structure questions with clear options when possible.

**Using AskUserQuestion effectively**:

- Provide 2-4 options per question when choices are clear
- Use `multiSelect: true` when multiple answers apply
- Keep question headers short (max 12 chars)
- Include descriptions that explain implications of each choice

**Question strategy**:

- Target the lowest-scoring dimension first
- Be specific, not open-ended
- Offer options when possible ("Is it A, B, or C?")
- Reference what was stated ("You mentioned X, did you mean...?")
- Limit to 3-5 questions per round
- After each round, recalculate confidence

See `references/confidence-rubric.md` for question templates by dimension and best practices.

### 3.5 Generate Brief

When confidence ≥ 95%, generate the brief document.

1. Use `AskUserQuestion` to confirm project name if not obvious from context
2. Convert to kebab-case for directory name
3. Create output directory: `./docs/ideation/{project-name}/`
4. Write `brief.md` using `references/brief-template.md`
5. Use `AskUserQuestion` to get approval:

```
Question: "Does this brief accurately capture your intent?"
Options:
- "Approved" - Brief is accurate, proceed
- "Needs changes" - Some parts need revision
- "Missing scope" - Important items are not captured
- "Start over" - Fundamentally off track, re-analyze
```

**If not approved:** Revise the brief based on feedback. Do not re-score confidence unless the feedback reveals a fundamental misunderstanding — in that case, return to Phase 3.2 and re-score. Otherwise, edit `brief.md` directly and re-present for approval. Iterate until approved.

**Do not proceed until brief is explicitly approved.**

## Phase 4: Phasing & Action Steps

After brief is approved, determine phases and generate action steps. Plans are optional.

### 4.1 Choose Workflow

Use `AskUserQuestion` to ask:

```
Question: "How should we proceed from the brief?"
Options:
- "Straight to action steps" — Recommended for focused projects.
  Brief defines what, action steps define how. Faster.
- "Plans then action steps" — Recommended for large scope or when
  you need to share requirements with others. Adds a requirements
  layer for alignment.
```

### 4.2 Determine Phases

Regardless of plan choice, analyze the brief and break scope into logical phases.

**Small-project shortcut:** If the scope is small enough to accomplish in a single phase (1-3 activities, a handful of deliverables), skip phasing entirely. Generate a single `action-steps.md` (no phase number needed) and proceed directly to handoff. Not every project needs multiple phases — don't force structure where simplicity suffices.

**Phasing criteria** (for multi-phase projects):

- Dependencies (what must be done first?)
- Risk (tackle high-risk items early)
- Value delivery (can you benefit after each phase?)
- Complexity (balance phases for consistent effort)

Typical phasing:

- Phase 1: Foundation / core setup
- Phase 2+: Additional activities, enhancements, extensions
- Phase N: Future considerations

**Detect repeatable patterns:** If 3+ phases follow the same structure with different inputs (e.g., "set up {room}" or "onboard {team member}"), note this — it affects how action steps are generated (see 4.4).

### 4.3 Generate Plans (only if user chose "Plans then action steps")

For each phase, generate `plan-phase-{n}.md` using `references/plan-template.md`.

Include:

- Phase overview and rationale
- Who benefits and how
- What needs to happen (grouped by activity)
- Quality standards
- Dependencies (what must be done first and what this phase produces)
- Done checklist

Present all plans for review. Use `AskUserQuestion`:

```
Question: "Do these plan phases look correct?"
Options:
- "Approved" - Phases and requirements look good, proceed to action steps
- "Adjust phases" - Need to move activities between phases
- "Missing requirements" - Some requirements are missing or unclear
- "Start over" - Need to revisit the brief
```

Iterate until user explicitly approves.

### 4.4 Generate Action Steps

Generate action steps using `references/action-steps-template.md`. How action steps are generated depends on whether phases are repeatable:

#### Standard phases (each is unique)

For each phase, generate a full `action-steps-phase-{n}.md` with:

- Overall approach
- Deliverables
- Step-by-step instructions
- Verification steps
- Contingency plans
- Completion checklist
- Progress check strategy (where applicable)

**Reference existing work:** When context gathering (Phase 2) found relevant prior work or similar projects, include "Similar to: {description}" in the action steps. This gives concrete examples to follow.

**Designing progress checks:** For each major activity, define how to verify it's on track during execution, not just at the end. See `references/progress-check-guide.md` for task-type mapping and design criteria.

#### Repeatable phases (3+ phases follow the same pattern)

When multiple phases share the same structure (e.g., "set up {room}"), avoid generating N nearly-identical full action steps. Instead:

1. **Generate one full template** — `action-steps-template-{pattern-name}.md` — with detailed steps, using placeholders for the variable parts.

2. **Generate lightweight per-phase delta files** — `action-steps-phase-{n}.md` — containing only:
   - Phase-specific inputs (e.g., room name, dimensions, budget allocation)
   - Deviations from the template (what's different about this phase)
   - Any phase-specific concerns or notes
   - Reference to the template: "Follow `action-steps-template-{pattern-name}.md` with the inputs below"

**Example for room-by-room renovation:**

`action-steps-template-room-setup.md`:
```markdown
# Room Setup Template

## Pattern
For each room:
1. Measure the space and document dimensions
2. Research furniture/fixtures that fit the space and budget
3. Purchase selected items
4. Assemble and arrange
5. Test and adjust

## Step-by-Step Instructions
{Detailed steps with {placeholders} for variable parts}

## Verification
{Common verification steps}
```

`action-steps-phase-3.md`:
```markdown
# Action Steps: Guest Bedroom Setup

**Template**: ./action-steps-template-room-setup.md

## Inputs
- Room: Guest bedroom
- Dimensions: 10x12 feet
- Budget: $600
- Priority items: Bed frame, mattress, nightstand
- Style: Minimalist, neutral colors

## Deviations from template
- Room has sloped ceiling on one side — tall furniture won't fit along south wall
- No closet — need freestanding wardrobe or clothing rack

## Phase-specific concerns
- Guest bed should be queen size (confirmed by homeowner)
- Need blackout curtains for guest comfort
```

This approach:
- Saves significant effort
- Makes the per-phase differences obvious
- Avoids repetition across action steps
- Makes it easy to update the shared pattern in one place

### 4.5 Present Phases for Review

Whether using plans or straight-to-action-steps, present the phase breakdown and action steps for user approval before proceeding to handoff.

**Before presenting action steps, evaluate quality** using the Action Steps Quality Check from `references/confidence-rubric.md`. Self-review each set of action steps:

- **Strong**: All major deliverables have verification steps, completion checklist defined, contingency plans for high-risk items → present as-is
- **Adequate**: Most deliverables have checks but some gaps → present with a note about what's missing
- **Weak**: No verification steps, or major deliverables have no way to confirm completion → revise before presenting

If Weak, fix the gaps first. Don't present action steps without verification for major deliverables.

Use `AskUserQuestion`:

```
Question: "Do these action steps look correct?"
Options:
- "Approved" - Action steps look good, proceed to execution handoff
- "Adjust approach" - Strategy needs changes
- "Missing steps" - Some activities or deliverables are missing
- "Revisit phases" - Phase breakdown needs restructuring
```

If not approved, revise the relevant action steps based on feedback and re-present. Iterate until approved.

## Phase 5: Execution Handoff

After action steps are generated, analyze orchestration options and hand off for execution.

### 5.1 Analyze Orchestration Strategy

Do not create tasks during ideation handoff — they are ephemeral and will be lost when the user starts a fresh session. Each `/execute-spec` session creates its own granular tasks.

Analyze the phase dependency graph to determine the best execution strategy.

**Detect parallelizable phases:**

- Examine which phases are blocked by what
- If 2+ phases share the same single blocker (e.g., all blocked only by Phase 1), they are **parallelizable**
- If phases form a linear chain (Phase 2 → Phase 3 → Phase 4), they are **sequential**
- Mixed graphs have both parallel and sequential segments

**Determine recommended strategy:**

| Pattern | Recommendation |
|---------|----------------|
| All phases sequential (chain) | **Sequential execution** — one session at a time |
| 2+ independent phases | **Agent team** — lead orchestrates teammates in parallel |
| Mixed dependencies | **Hybrid** — sequential for dependent chain, agent team for independent group |

### 5.2 Present Handoff Summary

Present the execution strategy directly to the user. No manifest file — the brief and action steps are the artifacts. The handoff summary is conversational output.

**Always include:**

```
Ideation complete. Artifacts written to `./docs/ideation/{project-name}/`.
```

**For sequential execution or when a blocking phase must complete first:**

```
## To start execution

```bash
claude
/execute-spec docs/ideation/{project-name}/action-steps-phase-1.md
```

Each `/execute-spec` session creates its own tasks.
```

**When ALL phases are independent (no blocking phase), skip the sequential start and go straight to the agent team prompt below.**

**When 2+ phases are parallelizable (after a blocking phase or from the start), present an agent team prompt.**

Agent teams let a single lead session automatically spawn and coordinate multiple teammates — the user starts **one** `claude` session, and the lead handles spawning, task assignment, plan approval, and synthesis. No manual terminal juggling.

```
## Parallel Execution with Agent Teams

After {blocking phase} is complete, {N} phases can run in parallel.

Ensure agent teams are enabled in `.claude/settings.json` or `~/.claude/settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Start **one** new Claude Code session and enter **delegate mode** (Shift+Tab), then paste:

```
{Blocking phase} is complete. Create an agent team to execute
{N} remaining phases in parallel. Each phase is independent.

Spawn {N} teammates with plan approval required. Each teammate should:
1. Read their assigned action steps file (and the template if referenced)
2. Review any existing context before planning
3. Plan their approach and wait for approval
4. Execute following the action steps and project approach
5. Run verification steps from their action steps after completion

Only approve plans that include verification steps and match
the project's established approach.

Teammates:

1. "{Phase title}" — docs/ideation/{project-name}/action-steps-phase-{n}.md
   {One-line context about what this phase does}

2. "{Phase title}" — docs/ideation/{project-name}/action-steps-phase-{n}.md
   {One-line context about what this phase does}

...
```
```

**Why delegate mode?** Pressing Shift+Tab restricts the lead to coordination-only tools: spawning teammates, messaging, managing tasks, and approving plans. This prevents the lead from executing tasks itself and ensures work is distributed to teammates.

**Why one session?** The lead automatically spawns each teammate as a separate Claude Code instance. Each teammate gets its own context window, loads project context, and works independently. You interact with the lead and it coordinates everything — use Shift+Up/Down to message individual teammates if needed.

**Shared deliverable detection:** Scan the action steps files' "Deliverables" sections. If multiple action steps produce overlapping deliverables, add a coordination note to the team prompt:

```
Coordinate on shared deliverables ({list}) to avoid conflicts —
only one teammate should work on a shared deliverable at a time.
```

**Batching:** If more than 5 parallelizable phases, recommend starting with the highest-priority batch first, then spawning the rest after plans are approved.

### 5.3 Why Fresh Sessions?

- Ideation consumes significant context (brief, action steps, exploration)
- Execution benefits from clean context focused on the action steps
- Human review between phases catches issues early
- Each phase is independently completable
- Each session creates granular tasks scoped to that phase

## Output Artifacts

All artifacts written to `./docs/ideation/{project-name}/`:

```
brief.md                              # Project brief (problem, goals, success, scope)
plan-phase-1.md                       # Phase 1 requirements (only if plans chosen)
...
action-steps-phase-1.md               # Phase 1 action steps (always full)
action-steps-template-{pattern}.md    # Shared template for repeatable phases (if applicable)
action-steps-phase-{n}.md             # Per-phase delta referencing template (if repeatable)
...
```

## Bundled Resources

### References

- `references/brief-template.md` - Template for project brief
- `references/plan-template.md` - Template for phased plan documents
- `references/action-steps-template.md` - Template for action steps
- `references/confidence-rubric.md` - Detailed scoring criteria for confidence assessment and action steps quality
- `references/progress-check-guide.md` - Task-type mapping for progress verification
- `references/workflow-example.md` - End-to-end workflow walkthrough

### Examples

Completed artifact examples for reference when generating output:

- `examples/brief-example.md` - A filled-in brief for a home office renovation
- `examples/plan-example.md` - A filled-in plan for the same project (Phase 1)
- `examples/action-steps-example.md` - A filled-in action steps for the same project

When generating artifacts, reference these examples for tone, structure, and level of detail.

## Important Notes

- **ALWAYS use `AskUserQuestion` tool for clarifications and approvals.** Never ask questions in plain text.
- **Gather context** before scoring confidence (unless brand-new project with no prior work).
- **Score confidence conservatively.** When uncertain, score lower.
- Never skip the confidence check. Don't assume understanding.
- Always write artifacts to files. Don't just display them.
- Each phase should be independently valuable.
- Action steps should be detailed enough to follow without re-reading the brief or plans.
- Keep briefs lean. Heavy docs slow iteration.
- **Reference existing work and prior progress** in action steps — "Similar to" with descriptions of what was done before.
- **Use template + delta** for repeatable phases — don't generate N identical action steps.
- **Small projects don't need phases.** If scope is 1-3 activities, generate a single set of action steps.
