# Skills Router SKILL.md Proposal v2.0

A consolidated synthesis of 7 brainstorm documents and 8 engineering reviews, with implementation-ready specifications.

---

## Executive Summary

This proposal defines how to build router/hub skills that delegate to specialized skills in Claude Code plugins. **Key insight from engineering review**: Routing in Claude Code is **LLM-mediated instruction following**, not programmatic code execution. The SKILL.md provides guidance that Claude reads and acts upon.

**Status**: Revised based on 8 engineering reviews (2025-12-31)

---

## How Routing Actually Works in Claude Code

> **Critical Understanding**: The router SKILL.md is *instructional*, not *executable*.

### The LLM IS the Router

```
1. User sends request ("create a bug in TES")
2. Claude Code matches triggers → loads relevant SKILL.md files
3. Claude reads hub SKILL.md content and decides which skill applies
4. Claude invokes specialized skill via the Skill tool
5. Specialized skill's SKILL.md guides the actual operation
```

### Implications for Design

| Traditional Router | Claude Code Router |
|-------------------|-------------------|
| Computed confidence scores | LLM implicit certainty |
| Regex pattern matching | Natural language understanding |
| Programmatic decision trees | Prose-based routing instructions |
| YAML configuration | Markdown with examples |
| Runtime execution | Context for LLM reasoning |

**Claude already excels at**:
- Entity extraction (issue keys, project keys, user references)
- Intent classification
- Disambiguation through conversation
- Context retention within a session

**The SKILL.md should provide**:
- Clear routing rules in natural language
- Examples Claude can pattern-match against
- Explicit instructions for edge cases
- Guidance on when to ask for clarification

---

## Routing Mechanism

### How the Router Invokes Skills

When the hub skill determines a request should go to a specialized skill:

1. **Invoke the Skill tool** with the target skill name
2. The target skill's SKILL.md is loaded into context
3. The target skill handles execution

```markdown
## Routing Instructions

When you determine the user needs jira-agile functionality:
1. Invoke the Skill tool with skill name "jira-agile"
2. The jira-agile SKILL.md will guide you from there
3. Do NOT attempt agile operations using jira-issue scripts
```

### Hub-as-Entry-Point Pattern

The hub skill is the **primary entry point** for broad queries, not a fallback:

```markdown
## When This Hub Skill Activates

- User mentions the product broadly ("help with jira", "jira question")
- User is unsure which skill to use
- Request is ambiguous between multiple skills
- User wants to discover capabilities

## When Specialized Skills Activate Directly

- User invokes skill by name ("/jira-issue create...")
- Request clearly matches a single skill's triggers
- Follow-up to a previous specialized skill operation
```

---

## Feature Priorities (Revised Based on Review)

### P0: Must-Have for MVP

1. **Complete skill registry**
   - All skills documented with capabilities
   - Status indicators (active/deprecated/experimental)
   - No missing skills (this causes routing failures)

2. **Routing rules in natural language**
   - Clear instructions Claude can follow
   - Explicit precedence when multiple skills match
   - Negative triggers (when NOT to route)

3. **Context awareness (basic)**
   - Pronoun resolution: "it" = last mentioned entity
   - Project scope: persist current project context
   - Last operation: for follow-up commands

4. **Disambiguation instructions**
   - When to ask clarifying questions
   - Template questions for common ambiguities
   - How to present options to users

5. **Quick reference table**
   - One-line mapping: "I want to X" → skill-Y

### P1: Should-Have

1. **Common workflow documentation**
   - Multi-skill patterns in natural language
   - Step-by-step instructions Claude follows

2. **Error handling guidance**
   - What to do when skills fail
   - How to suggest alternatives

3. **Destructive operation warnings**
   - Which operations need confirmation
   - Dry-run requirements

### P2-P3: Nice-to-Have

1. Discovery commands (`/browse-skills`)
2. Explain mode (why this skill was chosen)
3. Performance hints

---

## Decisions on Open Questions

Based on engineering review consensus:

| Question | Decision | Rationale |
|----------|----------|-----------|
| **Stateless vs Stateful?** | Stateful within session | Claude naturally maintains conversation context. Instruct it to reference recent entities. |
| **Learning from corrections?** | No (defer to v2) | Risk of drift; log corrections for manual threshold review instead. |
| **Multi-plugin federation?** | Out of scope for v1 | Solve single-product routing first. Cross-product is a platform concern. |
| **Fallback philosophy?** | Ask for clarification | Better UX than guessing wrong. Never silently fail. |
| **Partial workflow failure?** | Stop + report + offer guidance | Log what succeeded, explain what failed, suggest resolution. |
| **Error attribution?** | Last executed skill owns it | Simple and traceable. Surface original request for context. |
| **Context lifetime?** | Session-scoped | Cleared on session end. Use explicit references when uncertain. |

---

## Certainty-Based Routing (Replaces Confidence Scoring)

Instead of computed confidence scores, use natural language certainty instructions:

```markdown
## When to Route with High Certainty

Route directly to the skill without asking when:
- User explicitly mentions the skill or its primary trigger
- Request contains a strong entity signal (e.g., issue key → jira-issue)
- Only one skill could reasonably handle the request

Examples of high-certainty routing:
- "create a bug in TES" → jira-issue (create + issue type)
- "find all open bugs" → jira-search (find/search + criteria)
- "move TES-123 to Done" → jira-lifecycle (status transition)

## When to Clarify First

Ask the user before routing when:
- Request matches 2+ skills with similar likelihood
- Request is vague or could be interpreted multiple ways
- Destructive operations are implied

Examples requiring clarification:
- "show me the sprint" → Could be sprint details (jira-agile) or issues in sprint (jira-search)
- "update the issues" → Single issue (jira-issue) or bulk update (jira-bulk)?
- "remove the bugs" → Delete (destructive!) or just close?

## When Nothing Matches

If no skill clearly applies:
1. Show available skills with brief descriptions
2. Ask: "Could you tell me more about what you're trying to do?"
3. Suggest: "Try `/browse-skills` to see all capabilities"
```

---

## Context Awareness

Claude maintains conversation context naturally. The SKILL.md instructs how to use it:

```markdown
## Using Conversation Context

### Pronoun Resolution
When the user says "it", "this issue", or "that":
- Check the conversation for the last issue key mentioned
- If multiple candidates, ask: "Which issue - TES-123 or TES-456?"
- If no prior issue, ask: "Which issue are you referring to?"

### Project Scope
When the user mentions a project:
- Remember it for subsequent requests in this conversation
- "Create a bug in TES" → TES is now the active project
- "Create another bug" → Use TES implicitly

### Follow-up Commands
After an operation completes:
- "Assign it to me" → "it" = the issue just created/viewed
- "Add that to the sprint" → "that" = last mentioned issue
- "Link them together" → "them" = last search results

### When Context is Ambiguous
If unsure what the user means:
- Ask explicitly rather than guessing
- "You mentioned TES-123 and TES-456 earlier. Which one?"
```

---

## Anti-Patterns (Preserved from Original)

### 1. Greedy Matching
```
BAD: Triggers like [find, search, query, list, show, get, ...]
     Captures requests that belong to other skills

GOOD: Specific triggers [search issues, find issues, JQL query]
      With negative triggers: NOT "show me TES-123" (that's jira-issue)
```

### 2. Context Amnesia
```
BAD:  User: "Create a bug in TES" → TES-123 created
      User: "Assign it to me"
      Router: "Assign what? Please specify an issue key."

GOOD: Router remembers TES-123, assigns successfully
```

### 3. Implicit Destruction
```
BAD:  User: "remove the bugs"
      Router: *deletes 47 bugs without confirmation*

GOOD: "This will delete 47 bugs. Would you like to:
       1. Run with --dry-run first to preview
       2. Proceed with deletion
       3. Cancel"
```

### 4. Monolithic SKILL.md
```
BAD:  500+ line SKILL.md with everything
      - Wastes tokens
      - Hard to maintain

GOOD: ~120-150 lines for hub skill
      - Essential routing only
      - Detailed docs in references/ folder
```

---

## Reference Implementation: jira-assistant

```markdown
---
name: jira-assistant
description: Central hub for JIRA operations. Routes to 14 specialized skills. Use when working with JIRA or unsure which skill applies.
triggers:
  - jira
  - help with jira
  - what can you do with jira
  - which jira skill
---

# JIRA Assistant

I route requests to specialized JIRA skills. I don't handle operations directly - I help you find the right skill.

## Quick Reference

| I want to... | Use this skill |
|--------------|----------------|
| Create/edit/delete a single issue | jira-issue |
| Search with JQL, export results | jira-search |
| Change status, assign, set versions | jira-lifecycle |
| Manage sprints, epics, story points | jira-agile |
| Add comments, attachments, watchers | jira-collaborate |
| Link issues, view dependencies | jira-relationships |
| Log time, manage worklogs | jira-time |
| Handle service desk requests | jira-jsm |
| Update 10+ issues at once | jira-bulk |
| Generate Git branch names | jira-dev |
| Find custom field IDs | jira-fields |
| Manage cache, diagnostics | jira-ops |
| Project settings, permissions | jira-admin |

## Routing Rules

1. **Explicit skill mention wins** - If user says "use jira-agile", use it
2. **Entity signals** - Issue key present → likely jira-issue or jira-lifecycle
3. **Quantity determines bulk** - More than 10 issues → jira-bulk
4. **Keywords drive routing**:
   - "search", "find", "JQL" → jira-search
   - "sprint", "epic", "backlog" → jira-agile
   - "transition", "move to", "assign" → jira-lifecycle
   - "comment", "attach" → jira-collaborate

## Negative Triggers (When NOT to Route)

### jira-issue should NOT handle:
- Bulk operations (>10 issues) → use jira-bulk
- Status transitions → use jira-lifecycle
- Comments → use jira-collaborate
- Sprint/epic operations → use jira-agile

### jira-search should NOT handle:
- Single issue lookup by key → use jira-issue
- Modifying issues → use jira-issue or jira-bulk

## Disambiguation

### "Show me the sprint"
Could mean:
1. Sprint details (dates, goals, capacity) → jira-agile
2. Issues in the current sprint → jira-search

Ask: "Do you want sprint metadata or the issues in the sprint?"

### "Update the issue"
Could mean:
1. Change fields on one issue → jira-issue
2. Transition status → jira-lifecycle
3. Update multiple issues → jira-bulk

Ask: "What would you like to update - fields, status, or multiple issues?"

## Context Awareness

When user says "it" or "that issue":
- Reference the last issue key from our conversation
- If TES-123 was just created, "assign it to me" means TES-123

When user establishes project context:
- "In TES, find bugs" → TES is now the active project
- Subsequent requests use TES implicitly

## Common Workflows

### Create Epic with Stories
1. Use jira-agile to create the epic
2. Use jira-issue to create each story
3. Use jira-agile to link stories to epic
4. Optionally add to sprint with jira-agile

### Bulk Close from Search
1. Use jira-search to find matching issues
2. Use jira-bulk with --dry-run to preview
3. Confirm with user before executing

## What I Don't Do

- I don't execute JIRA operations directly
- I don't guess when uncertain - I ask
- I don't perform destructive operations without confirmation

## Discovery

- Ask "what can you do?" for this overview
- Ask "tell me about jira-agile" for skill details
- Use `/browse-skills` for interactive browser
```

---

## Token Budget Guidelines

| Component | Target | Rationale |
|-----------|--------|-----------|
| Hub SKILL.md | 120-150 lines | Always loaded; keep minimal |
| Specialized SKILL.md | 80-150 lines | Loaded on demand |
| References docs | Variable | Only loaded when needed |

### Progressive Disclosure Pattern

```
Level 0: Hub SKILL.md (~120 lines)
         └── Routing rules, quick reference, disambiguation

Level 1: Specialized SKILL.md (~100 lines)
         └── Skill-specific guidance, scripts, examples

Level 2: references/ folder (loaded on demand)
         └── Detailed API docs, edge cases, troubleshooting
```

Instruction in hub: "For detailed API documentation, read `references/api-v2.md`"

---

## Testing Strategy

### Golden Test Set Format

```yaml
# routing_golden.yaml
version: "1.0"
tests:
  # High certainty - route directly
  - id: TC001
    input: "create a bug in TES"
    expected_skill: jira-issue
    certainty: high
    entities:
      project: TES
      issue_type: Bug

  # Disambiguation required
  - id: TC002
    input: "show me the sprint"
    expected_skill: null  # Ambiguous
    certainty: low
    disambiguation_options:
      - skill: jira-agile
        description: "Sprint details (dates, goals)"
      - skill: jira-search
        description: "Issues in the sprint"

  # Context-dependent
  - id: TC003
    input: "assign it to me"
    context:
      last_issue: TES-123
    expected_skill: jira-lifecycle
    resolved_entity: TES-123

  # Negative trigger test
  - id: TC004
    input: "transition 50 issues to Done"
    expected_skill: jira-bulk
    not_skill: jira-lifecycle  # Quantity triggers bulk
```

### Coverage Requirements

| Category | Minimum Cases | Purpose |
|----------|---------------|---------|
| Direct routing | 20 | Clear single-skill matches |
| Disambiguation | 15 | Multiple skills could match |
| Context-dependent | 10 | Requires conversation history |
| Negative triggers | 10 | Should NOT route to skill |
| Workflows | 10 | Multi-skill sequences |
| Edge cases | 10 | Typos, partial, malformed |

---

## Success Metrics

### Phase 1 (Foundation)
- 90%+ accuracy on golden test set
- All 14 skills in registry (0 missing)
- Hub SKILL.md under 150 lines

### Phase 2 (Intelligence)
- <10% disambiguation requests for clear queries
- 95%+ user satisfaction with clarification quality
- Context resolution working for pronouns

### Phase 3 (Resilience)
- 0 destructive operations without confirmation
- Clear error messages with recovery suggestions

---

## Non-Goals (Out of Scope for v1)

- Cross-product routing (JIRA + Confluence in one query)
- Learning from user corrections
- Embedding-based semantic similarity
- Automatic workflow optimization
- Multi-language support

---

## Implementation Phases (Revised)

### Phase 1: Foundation (MVP)
- [ ] Complete skill registry (all 14 skills)
- [ ] Routing rules in natural language
- [ ] Quick reference table
- [ ] Basic context instructions
- [ ] Disambiguation templates

### Phase 2: Polish
- [ ] Negative triggers for all skills
- [ ] Common workflow documentation
- [ ] Error handling guidance
- [ ] Destructive operation warnings

### Phase 3: Enhancement
- [ ] Discovery commands
- [ ] Detailed references
- [ ] Performance optimization guidance

---

## Decision Log

| ID | Decision | Rationale | Alternatives Rejected |
|----|----------|-----------|----------------------|
| D1 | LLM-mediated routing | Claude Code uses SKILL.md as context, not code | Programmatic router with computed scores |
| D2 | Hub-as-entry-point | Catch broad/ambiguous requests first | Hub-as-fallback (rejected: inefficient) |
| D3 | Natural language certainty | LLM implicit confidence, not numeric | Computed confidence thresholds |
| D4 | Context via conversation | Claude maintains session state | Custom state management |
| D5 | Prose-based SKILL.md | LLM reads natural language better | YAML configuration files |
| D6 | Skill tool invocation | Standard Claude Code mechanism | Custom delegation protocol |

---

## Conclusion

The router SKILL.md is not code—it's **instructions for Claude**. Write it as if explaining routing logic to a smart colleague who will follow your guidance.

**Key principles**:
1. Claude is the router; SKILL.md guides its decisions
2. Use natural language, not configuration
3. Provide examples Claude can pattern-match
4. Be explicit about when to ask for clarification
5. Keep hub skill minimal; load details on demand

**Next steps**:
1. Implement jira-assistant using this template
2. Build golden test set (75+ cases)
3. Validate with real user scenarios
4. Iterate based on routing failures

---

*Document revised: 2025-12-31*
*Version: 2.0 (incorporates 8 engineering reviews)*
*Sources: 7 SKILLS_ROUTER_BRAINSTORM.md files + 8 SRS_RESPONSE_01.md feedback documents*
