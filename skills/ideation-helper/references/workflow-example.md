# Workflow Example

**User provides brain dump** (via dictation):

```
okay so I want to organize a workshop for my community, like a weekend thing
where people can learn practical skills, maybe like basic home repairs or
gardening or cooking, I'm not sure which topic yet, and we'd need a venue
obviously, I was thinking the community center but I haven't checked if
it's available, and we'd need someone to teach it, maybe I could ask around
or find someone on Facebook, and I want to keep it free or really cheap
because the whole point is accessibility, oh and we should probably have
some kind of sign-up system so we know how many people are coming, and
maybe provide some basic supplies or materials, I'm thinking like 20-30
people max, and it should be hands-on not just a lecture
```

**Process**:

1. **Intake**: Accept without judgment

2. **Context Gathering**: Check if the user has any existing documents, prior event plans, venue contacts, or community group info. Ask about budget, timeline, and available helpers.

3. **Analysis**:
   - Problem: Community lacks accessible skill-building opportunities
   - Goals: Organize a hands-on workshop, keep it free/cheap, 20-30 people
   - Unclear: Topic? Venue confirmed? Budget? Timeline? Who teaches?
   - Confidence: ~50/100 (low goal definition, unclear scope)

4. **Questions** (round 1):
   - "Which topic interests your community most: home repairs, gardening, or cooking?"
   - "What's your budget for this? Zero (fully volunteer/donated) or do you have some funds?"
   - "When are you hoping to hold this? Rough timeline?"
   - "Do you have any connections who could teach, or do we need to find someone?"

5. **User responds** → Recalculate confidence → Repeat if needed

6. **Confidence reaches 96%** → Generate brief

7. **Brief approved** → Ask: "Straight to action steps or plans first?"

8. **Determine phases**:
   - Phase 1: Planning & logistics (venue, instructor, date, budget)
   - Phase 2: Promotion & sign-ups (flyers, social media, registration)
   - Phase 3: Preparation & execution (materials, setup, run the event)

9. **Generate action steps** (referencing any existing contacts or resources found in context gathering)
   - Each set of action steps includes verification steps and a completion checklist
   - Self-review quality before presenting — Strong/Adequate/Weak (see `confidence-rubric.md`)

10. **Execution handoff**: Analyze dependencies, present strategy
    - Phases are sequential (each depends on the previous)
    - Recommend: complete Phase 1, then Phase 2, then Phase 3

11. **Execution** (fresh sessions): For each phase:
    - Start fresh Claude session
    - Run `/execute-spec action-steps-phase-{n}.md`
    - Review, adjust, proceed
    - Repeat for next phase
