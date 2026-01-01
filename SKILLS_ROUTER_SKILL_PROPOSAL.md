# Skills Router SKILL.md Proposal v2.1

A consolidated synthesis of 7 brainstorm documents and 16 engineering reviews, with implementation-ready specifications.

---

## Executive Summary

This proposal defines how to build router/hub skills that delegate to specialized skills in Claude Code plugins. **Key insight**: Routing in Claude Code is **LLM-mediated instruction following**, not programmatic code execution. The SKILL.md provides guidance that Claude reads and acts upon.

**Status**: v2.1 - Approved for implementation (2025-12-31)
**Consensus Grade**: A (implementation-ready)

---

## How Routing Actually Works in Claude Code

> **Critical Understanding**: The router SKILL.md is *instructional*, not *executable*.

### The LLM IS the Router

```
1. User sends request ("create a bug in TES")
2. Claude Code matches skill descriptions → loads relevant SKILL.md files
3. Claude reads hub SKILL.md content and decides which skill applies
4. Claude invokes specialized skill via the Skill tool
5. Specialized skill's SKILL.md guides the actual operation
```

### What NOT to Do

Do NOT attempt to build:
- Python router scripts (Claude Code can't execute them)
- Regex-based trigger matching (use prose instead)
- Confidence scoring algorithms (Claude does this implicitly)
- State management databases (use conversation context)

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

### How to Invoke Skills

When you (Claude) determine a request needs a specialized skill, use the Skill tool:

```
Skill: jira-agile
```

The Skill tool takes only the skill name. Context from the conversation carries forward automatically.

**Example flow**:
```
User: "add a story to the sprint"
Claude: (recognizes this as jira-agile territory)
Claude: Skill: jira-agile
→ jira-agile SKILL.md loads
→ Claude continues with the original request using jira-agile guidance
```

**Important**: After a skill completes, the conversation continues naturally. Do NOT re-invoke the hub unless the user asks a new, unrelated question.

### Hub-as-Coordinator Pattern

The hub skill coordinates routing decisions—it's neither "always first" nor "only fallback":

**When the Hub Activates**:
- User mentions the product broadly ("help with jira", "jira question")
- User is unsure which skill to use ("which skill handles...")
- Request is ambiguous between multiple skills
- User wants to discover capabilities ("what can you do?")
- No specialized skill's triggers clearly match

**When Specialized Skills Activate Directly**:
- User explicitly invokes by name ("/jira-issue", "use jira-agile")
- Request clearly matches a single skill's triggers with strong signals
- Follow-up to a previous specialized skill operation

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
   - Maximum 3 options per disambiguation (collapse similar cases)

5. **Quick reference table**
   - One-line mapping: "I want to X" → skill-Y

6. **Permission awareness**
   - Check if operations will fail due to access
   - Explain access issues before attempting
   - Suggest alternatives when blocked

### P1: Should-Have

1. **Common workflow documentation**
   - Multi-skill patterns in natural language
   - Explicit data passing between steps

2. **Error handling guidance**
   - Templates for common error types
   - Recovery suggestions per error category

3. **Destructive operation warnings**
   - Which operations need confirmation
   - Dry-run requirements for bulk operations

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
| **Learning from corrections?** | No (defer to v2) | Risk of drift; log corrections for manual review instead. |
| **Multi-plugin federation?** | Out of scope for v1 | Solve single-product routing first. Cross-product is a platform concern. |
| **Fallback philosophy?** | Ask for clarification | Better UX than guessing wrong. Never silently fail. |
| **Partial workflow failure?** | Stop + report + offer options | Log what succeeded, explain what failed, offer recovery choices. |
| **Error attribution?** | Last executed skill owns it | Simple and traceable. Surface original request for context. |
| **Context lifetime?** | Session-scoped with expiry | Cleared on session end. Re-confirm after 5+ messages or 3+ minutes. |
| **Skill availability?** | Acknowledge and suggest alternatives | If skill missing, say so clearly and offer alternatives. |

---

## Certainty-Based Routing (Replaces Confidence Scoring)

Instead of computed confidence scores, use natural language certainty instructions:

### When to Route with High Certainty

Route directly to the skill without asking when:
- User explicitly mentions the skill or its primary trigger
- Request contains a strong entity signal (e.g., issue key → jira-issue)
- Only one skill could reasonably handle the request

**Examples of high-certainty routing**:
- "create a bug in TES" → jira-issue (create + issue type)
- "find all open bugs" → jira-search (find/search + criteria)
- "move TES-123 to Done" → jira-lifecycle (status transition)

### When to Clarify First

Ask the user before routing when:
- Request matches 2+ skills with similar likelihood
- Request is vague or could be interpreted multiple ways
- Destructive operations are implied

**Examples requiring clarification**:
- "show me the sprint" → Could be sprint details (jira-agile) or issues in sprint (jira-search)
- "update the issues" → Single issue (jira-issue) or bulk update (jira-bulk)?
- "remove the bugs" → Delete (destructive!) or just close?

### When Nothing Matches

If no skill clearly applies:
1. Show available skills with brief descriptions
2. Ask: "Could you tell me more about what you're trying to do?"
3. Suggest: "Try `/browse-skills` to see all capabilities"

---

## Context Awareness

Claude maintains conversation context naturally. The SKILL.md instructs how to use it:

### Pronoun Resolution

When the user says "it", "this issue", or "that":
- If exactly one issue mentioned in last 3 messages → use it
- If multiple issues mentioned → ask: "Which issue - TES-123 or TES-456?"
- If no issue in last 5 messages → ask: "Which issue are you referring to?"

**Special case - after CREATE**:
```
User: "create a bug in TES" → TES-789 created
User: "assign it to me"
→ "it" = TES-789 (the issue just created)
```

**Special case - after SEARCH**:
```
User: "find all open bugs" → Found TES-100, TES-101, TES-102
User: "close them"
→ "them" = the search results (use jira-bulk)
```

### Project Scope

When the user mentions a project:
- Remember it for subsequent requests in this conversation
- "Create a bug in TES" → TES is now the active project
- "Create another bug" → Use TES implicitly
- Explicit project mention updates the active project

### Context Expiration

After 5+ messages or 3+ minutes since last reference:
- Re-confirm rather than assume: "Do you mean TES-123 from earlier?"
- Don't guess when context is stale

### Conflicting Context

When recent mentions conflict:
```
User: "Search for bugs in TES" → Found TES-100, TES-101
User: "Show me TES-456"  [different issue]
User: "Close it"
→ "it" = TES-456 (most recent explicit mention wins)
```

---

## Error Handling Guidance

### Error Response Templates

**Skill Not Found**:
```
"I don't have a skill called '{name}'. Available skills for JIRA:
- jira-issue, jira-search, jira-agile, jira-lifecycle...

Did you mean one of these?"
```

**Permission Denied**:
```
"You don't have permission to {operation} in project {PROJECT}.
You may need to:
- Request access from your JIRA administrator
- Use a different project where you have access

Would you like me to check your permissions with jira-admin?"
```

**Rate Limited**:
```
"JIRA is rate limiting requests. Please wait {X} seconds before retrying.
For bulk operations, consider using jira-bulk with smaller batch sizes."
```

**Entity Not Found**:
```
"Issue TES-999 doesn't exist. Possible causes:
- Typo in the issue key
- Issue was deleted
- You don't have access to view it

Would you like me to search for similar issues?"
```

**Skill Execution Failed**:
```
"The {skill_name} operation failed: {error_message}

Suggestions:
- {recovery_option_1}
- {recovery_option_2}

Try a different approach with {alternative_skill}?"
```

### Error Escalation Paths

| From Skill | Escalate To | When |
|------------|-------------|------|
| jira-issue | jira-bulk | >10 issues to modify |
| jira-lifecycle | jira-admin | Workflow/permission issues |
| jira-search | jira-fields | Unknown field errors |
| Any skill | jira-ops | Cache/connectivity issues |

---

## Partial Workflow Failure

When a multi-step workflow fails partway:

1. **Stop immediately** - Don't attempt remaining steps
2. **Report what succeeded**: "Created epic TES-100 and story TES-101"
3. **Explain the failure**: "Story 2 failed: field 'customfield_123' is required"
4. **Offer options**:
   - "Fix the issue and continue with remaining stories?"
   - "Keep what was created and stop here?"
   - "Delete TES-100 and TES-101 to start fresh?"

**Default behavior**: Keep successful items. Deletion requires explicit confirmation.

### Data Passing Between Steps

When one skill's output feeds another:
- Capture entity IDs from responses (e.g., epic key from jira-agile)
- State this explicitly: "Created EPIC-123. Now creating stories..."
- Reference captured data in subsequent operations

**Example**:
```
1. Invoke jira-agile to create the epic
   → Note: Epic TES-100 created
2. Invoke jira-issue to create each story
   → "Link to the epic we just created" (Claude knows TES-100)
   → Stories TES-101, TES-102 created
3. Confirm: "Created epic TES-100 with 2 stories linked"
```

---

## Anti-Patterns

### 1. Greedy Matching

**What it looks like**:
```
jira-search triggers: [find, search, query, list, show, get, display, view]
```

**Why it's bad**: "show me TES-123" matches "show" but should route to jira-issue.

**How to detect**: Test "show me TES-123" - if routed to jira-search, triggers are too greedy.

**Fix**:
```
jira-search triggers: [search issues, find issues, JQL query, filter]
NOT for: "show me TES-XXX" (single issue lookup)
```

### 2. Context Amnesia

**What it looks like**: No mention of conversation history in routing instructions.

**Why it's bad**:
```
User: "Create a bug in TES" → TES-123 created
User: "Assign it to me"
Router: "Assign what? Please provide an issue key." ✗
```

**How to detect**: Test create → "assign it" sequence.

**Fix**: Add explicit context resolution rules (see Context Awareness section).

### 3. Implicit Destruction

**What it looks like**: No special handling for destructive operations.

**Why it's bad**: User says "remove the bugs" → *deletes 47 bugs without confirmation*

**How to detect**: Test "delete TES-123" - if executed immediately, safety is missing.

**Fix**:
```
Before routing to delete operations:
1. Confirm: "This will DELETE issue TES-123. Confirm? (yes/no)"
2. Wait for explicit "yes"
3. Then route to skill
```

### 4. Monolithic SKILL.md

**What it looks like**: Hub SKILL.md includes detailed script documentation, API references, examples for every edge case.

**Why it's bad**: Wastes tokens (loaded on every request), slows routing.

**How to detect**: `wc -l SKILL.md` - if >200 lines, likely monolithic.

**Fix**: Keep only routing essentials. Move details to docs/ folder.

### 5. Skill Leakage

**What it looks like**: Hub skill starts performing operations directly instead of delegating.

**Why it's bad**: Hub should only route, never execute.

**Fix**: Hub says "I'll use jira-issue to create that for you" then invokes the skill.

### 6. Circular References

**What it looks like**:
```
jira-issue: NOT for sprint operations → use jira-agile
jira-agile: NOT for issue creation → use jira-issue
```

User asks: "Create an issue in an epic" → Router ping-pongs.

**Fix**: Add precedence rules - context matters:
```
"Create a story in epic TES-1" → jira-agile (epic context)
"Create a bug in TES" → jira-issue (no epic/sprint context)
```

### 7. Disambiguation Overload

**What it looks like**: 6+ options for "show sprint".

**Fix**: Maximum 3 options. Collapse similar cases:
1. Sprint metadata (details/velocity/reports) → jira-agile
2. Issues in the sprint → jira-search

### 8. No Escape Hatch

**What it looks like**: Router always interprets and routes. User can't bypass.

**Fix**: If user explicitly invokes a skill by name, defer immediately:
- "/jira-issue create bug" → Route to jira-issue (don't second-guess)

---

## Reference Implementation: jira-assistant

> **Note**: Triggers are configured in plugin.json, not in SKILL.md frontmatter. This SKILL.md content is loaded when the skill activates.

```markdown
# JIRA Assistant

This hub routes requests to specialized JIRA skills. It does not execute JIRA operations directly—it helps find the right skill.

## Quick Reference

| I want to... | Use this skill | Risk |
|--------------|----------------|:----:|
| Create/edit/delete a single issue | jira-issue | ⚠️ |
| Search with JQL, export results | jira-search | - |
| Change status, assign, set versions | jira-lifecycle | ⚠️ |
| Manage sprints, epics, story points | jira-agile | - |
| Add comments, attachments, watchers | jira-collaborate | - |
| Link issues, view dependencies | jira-relationships | - |
| Log time, manage worklogs | jira-time | - |
| Handle service desk requests | jira-jsm | - |
| Update 10+ issues at once | jira-bulk | ⚠️⚠️ |
| Generate Git branch names | jira-dev | - |
| Find custom field IDs | jira-fields | - |
| Manage cache, diagnostics | jira-ops | - |
| Project settings, permissions | jira-admin | ⚠️⚠️ |

**Risk Legend**: `-` Read-only/safe | `⚠️` Has destructive ops (confirm) | `⚠️⚠️` High-risk (confirm + dry-run)

## Routing Rules

1. **Explicit skill mention wins** - If user says "use jira-agile", use it
2. **Entity signals** - Issue key present → likely jira-issue or jira-lifecycle
3. **Quantity determines bulk** - More than 10 issues → jira-bulk
4. **Keywords drive routing**:
   - "search", "find", "JQL" → jira-search
   - "sprint", "epic", "backlog" → jira-agile
   - "transition", "move to", "assign" → jira-lifecycle
   - "comment", "attach" → jira-collaborate

## Negative Triggers

### jira-issue should NOT handle:
- Bulk operations (>10 issues) → jira-bulk
- Status transitions → jira-lifecycle
- Comments/attachments → jira-collaborate
- Sprint/epic operations → jira-agile

### jira-search should NOT handle:
- Single issue lookup by key → jira-issue
- Modifying issues → jira-issue or jira-bulk

### jira-lifecycle should NOT handle:
- Field updates (summary, description) → jira-issue
- Bulk transitions → jira-bulk

### jira-agile should NOT handle:
- Issue CRUD (except epic creation) → jira-issue
- JQL searches → jira-search
- Time tracking → jira-time

### jira-bulk should NOT handle:
- Single issue operations → jira-issue
- Sprint management → jira-agile

## Disambiguation Examples

### "Show me the sprint"
Could mean:
1. Sprint metadata (dates, goals, capacity) → jira-agile
2. Issues in the current sprint → jira-search

Ask: "Do you want sprint details or the issues in the sprint?"

### "Update the issue"
Could mean:
1. Change fields on one issue → jira-issue
2. Transition status → jira-lifecycle
3. Update multiple issues → jira-bulk

Ask: "What would you like to update - fields, status, or multiple issues?"

### "Create an issue in the epic"
Context determines:
- Epic context explicit → jira-agile
- Just issue creation → jira-issue

## Context Awareness

When user says "it" or "that issue":
- Reference the last issue key from our conversation
- If TES-123 was just created, "assign it to me" means TES-123

When user establishes project context:
- "In TES, find bugs" → TES is now the active project
- Subsequent requests use TES implicitly

## Common Workflows

### Create Epic with Stories
1. Use jira-agile to create the epic → Note epic key (e.g., TES-100)
2. Use jira-issue to create each story → Link to TES-100
3. Confirm: "Created epic TES-100 with N stories"

### Bulk Close from Search
1. Use jira-search to find matching issues
2. Use jira-bulk with --dry-run to preview
3. Confirm count with user before executing

## Handling Errors

If a skill fails:
- Report the error clearly
- Suggest recovery options
- Offer alternative approaches

If a skill is not available:
- Acknowledge the limitation
- Suggest alternatives

## What This Hub Does NOT Do

- Execute JIRA operations directly
- Guess when uncertain (asks instead)
- Perform destructive operations without confirmation
```

---

## Token Budget Guidelines

| Component | Target | Hard Limit |
|-----------|--------|------------|
| Hub SKILL.md | 150-180 lines | 200 lines |
| Specialized SKILL.md | 80-120 lines | 150 lines |
| Combined (hub + specialized) | ~250-300 lines | 350 lines |

### Token Usage Example

```
Hub SKILL.md (jira-assistant):     ~3,500 tokens
Specialized SKILL.md (jira-issue): ~2,500 tokens
User query:                          ~150 tokens
Claude's working memory:           ~2,000 tokens
─────────────────────────────────────────────────
Total context:                     ~8,150 tokens
```

**Target**: Keep combined hub + specialized < 8,000 tokens.

### If You Exceed Budget

1. Move examples to docs/ folder
2. Condense routing rules (remove redundancy)
3. Use tables instead of prose where possible
4. Split large specialized skill into sub-skills

### Progressive Disclosure Pattern

```
Level 0: Hub SKILL.md (~150 lines)
         └── Routing rules, quick reference, disambiguation

Level 1: Specialized SKILL.md (~100 lines)
         └── Skill-specific guidance, scripts, examples

Level 2: docs/ folder (loaded on demand)
         └── Detailed API docs, edge cases, troubleshooting
```

---

## Testing Strategy

### Test Execution Method

**Manual Testing (MVP)**:
1. Start Claude Code session with plugin installed
2. For each test case, send the input
3. Record: which skill was invoked OR what clarification was asked
4. Compare to expected outcome
5. Log discrepancies in `routing_failures.md`

**Semi-Automated Testing (Phase 2)**:
```python
# test_routing.py
import anthropic

def test_high_certainty_routing():
    client = anthropic.Anthropic()
    hub_skill = open('.claude/skills/jira-assistant/SKILL.md').read()

    response = client.messages.create(
        model="claude-sonnet-4-20250514",
        system=hub_skill,
        messages=[{"role": "user", "content": "create a bug in TES"}]
    )

    assert "jira-issue" in response.content
    assert "?" not in response.content[:100]  # No clarification asked
```

**Regression Testing**:
1. Save baseline responses for golden set
2. On SKILL.md changes, compare new responses
3. Flag any routing decision changes for review

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

  # Disambiguation required
  - id: TC002
    input: "show me the sprint"
    expected_skill: null  # Ambiguous
    certainty: low
    disambiguation_options:
      - jira-agile
      - jira-search

  # Context-dependent
  - id: TC003
    input: "assign it to me"
    context:
      last_issue: TES-123
    expected_skill: jira-lifecycle

  # Negative trigger test
  - id: TC004
    input: "transition 50 issues to Done"
    expected_skill: jira-bulk
    not_skill: jira-lifecycle
```

### Coverage Requirements

| Category | Minimum | Purpose |
|----------|---------|---------|
| Direct routing | 20 | Clear single-skill matches |
| Disambiguation | 15 | Multiple skills could match |
| Context-dependent | 10 | Requires conversation history |
| Negative triggers | 10 | Should NOT route to skill |
| Workflows | 10 | Multi-skill sequences |
| Edge cases | 10 | Typos, partial, malformed |
| **Total** | **75** | |

### Acceptance Criteria

**Phase 1**:
- [ ] All skills in registry (count matches plugin.json)
- [ ] Negative triggers for all skills documented
- [ ] Golden set accuracy >90%
- [ ] Hub SKILL.md <200 lines

**Phase 2**:
- [ ] Disambiguation quality >80% (user picks option, doesn't rephrase)
- [ ] Pronoun resolution >95% correct
- [ ] Error templates for all categories

**Phase 3**:
- [ ] 100% destructive ops require confirmation
- [ ] Error messages include recovery suggestions

---

## Success Metrics

### How to Measure

```
accuracy = (correct_routes + correct_clarifications) / total_tests

Where:
- correct_route: Claude invoked expected_skill
- correct_clarification: Claude asked when certainty=low
- Incorrect: Wrong skill OR failed to clarify when needed
```

### Targets by Phase

| Phase | Metric | Target | Measurement |
|-------|--------|--------|-------------|
| 1 | Accuracy on golden set | >90% | Automated test run |
| 1 | Skills in registry | 100% | Count check |
| 1 | Hub SKILL.md size | <200 lines | wc -l |
| 2 | Disambiguation rate for clear queries | <10% | Log analysis |
| 2 | User accepts first clarification | >80% | User picks option |
| 2 | Context resolution | >95% | Pronoun test cases |
| 3 | Destructive ops confirmed | 100% | Safety test cases |
| 3 | Error messages with recovery | 100% | Error test cases |

---

## Discovery Commands

### /browse-skills

Displays all available skills with brief descriptions:

```
Available JIRA Skills:

CRUD & Core:
  jira-issue      Create, read, update, delete issues
  jira-search     Find issues with JQL, export results
  jira-bulk       Update 10+ issues at once

Workflow:
  jira-lifecycle  Status transitions, assignments
  jira-agile      Sprints, epics, story points

Collaboration:
  jira-collaborate Comments, attachments, watchers
  jira-relationships Issue linking, dependencies

Specialized:
  jira-time       Time tracking, worklogs
  jira-jsm        Service desk operations
  jira-dev        Git integration
  jira-fields     Custom field discovery
  jira-ops        Cache, diagnostics
  jira-admin      Project settings, permissions

Type "tell me about [skill]" for details.
```

### "What can you do?"

If user asks about capabilities, show the Quick Reference table and offer details about specific skills.

---

## Router Maintenance

### Adding a New Skill

1. **Add to Quick Reference Table**
2. **Update Routing Rules** (add trigger keywords)
3. **Add Negative Triggers to Other Skills** (what they should NOT handle)
4. **Add Disambiguation Cases** (if overlap exists)
5. **Update Golden Test Set** (5+ test cases for new skill)
6. **Run regression tests**

### Deprecating a Skill

1. Add `(deprecated)` status in registry
2. Include deprecation note: "Use {replacement} instead"
3. Hub continues routing but warns user
4. After 2 versions, change to `removed`

### Diagnosing Routing Failures

When users report "it picked the wrong skill":

1. **Log the failure**:
   ```yaml
   user_input: "show me sprint capacity"
   expected_skill: jira-agile
   actual_skill: jira-search
   ```

2. **Analyze**: Was instruction ambiguous? Common edge case?

3. **Fix router SKILL.md**: Add specific trigger or negative trigger

4. **Add test case** to prevent regression

---

## Non-Goals (Out of Scope for v1)

- **Cross-product routing** - Solve single-product first
  *v2 consideration: meta-router pattern*
- **Learning from corrections** - Risk of drift
  *v2 consideration: log corrections for batch review*
- **Embedding-based semantic similarity**
- **Automatic workflow optimization**
- **Multi-language support**
- **Skill versioning/deprecation workflows** (defer to v2)

---

## Implementation Phases

### Phase 1: Foundation (MVP)
- [ ] Complete skill registry (verify count matches plugin.json)
- [ ] Routing rules in natural language
- [ ] Quick reference table with risk indicators
- [ ] Basic context instructions
- [ ] Disambiguation templates (max 3 options each)

### Phase 2: Polish
- [ ] Negative triggers for ALL skills (not just 2)
- [ ] Common workflow documentation with data passing
- [ ] Error handling templates
- [ ] Destructive operation warnings

### Phase 3: Enhancement
- [ ] Discovery commands (/browse-skills)
- [ ] Detailed docs in docs/ folder
- [ ] Performance optimization guidance
- [ ] Maintenance guide

---

## Decision Log

| ID | Decision | Rationale | Alternatives Rejected |
|----|----------|-----------|----------------------|
| D1 | LLM-mediated routing | Claude Code uses SKILL.md as context, not code | Programmatic router with computed scores |
| D2 | Hub-as-coordinator | Routes broad/ambiguous; specialized for clear matches | Hub-as-only-entry-point (too much overhead) |
| D3 | Natural language certainty | LLM implicit confidence, not numeric | Computed confidence thresholds |
| D4 | Context via conversation | Claude maintains session state | Custom state management |
| D5 | Prose-based SKILL.md | LLM reads natural language better | YAML configuration files |
| D6 | Skill tool invocation | Standard Claude Code mechanism | Custom delegation protocol |
| D7 | Negative triggers as blocklist | Explicit "NOT for" is clearer | Implicit via routing rules only |
| D8 | Max 3 disambiguation options | More options overwhelm users | Unlimited options |
| D9 | Session-scoped context with expiry | Re-confirm after 5+ messages | Infinite context retention |

---

## Conclusion

The router SKILL.md is not code—it's **instructions for Claude**. Write it as if explaining routing logic to a smart colleague who will follow your guidance.

**Key principles**:
1. Claude is the router; SKILL.md guides its decisions
2. Use natural language, not configuration
3. Provide examples Claude can pattern-match
4. Be explicit about when to ask for clarification
5. Keep hub skill minimal; load details on demand
6. Never execute operations from hub—always delegate

**Next steps**:
1. Implement jira-assistant using this template
2. Build golden test set (75+ cases)
3. Validate with real user scenarios
4. Iterate based on routing failures

---

## Version History

| Version | Date | Changes | Grade |
|---------|------|---------|-------|
| 1.0 | 2024-12-31 | Initial synthesis of 7 brainstorms | B+ |
| 2.0 | 2024-12-31 | LLM-mediated routing paradigm, resolved open questions | A- |
| 2.1 | 2024-12-31 | Error handling, test execution, maintenance guide, all feedback incorporated | A |

### Changes in v2.1

1. Removed YAML frontmatter from reference implementation (not how Claude Code works)
2. Added concrete Skill tool invocation syntax
3. Renamed D2 to "Hub-as-coordinator" with clearer activation rules
4. Added complete error handling templates
5. Added workflow data passing instructions
6. Added test execution method (manual + semi-automated)
7. Defined /browse-skills output format
8. Added partial failure/rollback guidance
9. Added context expiration rules
10. Added permission/skill availability handling
11. Revised token budget (150-200 realistic)
12. Completed negative triggers for all skills
13. Added success metrics measurement formulas
14. Added 3 more anti-patterns (skill leakage, circular refs, disambiguation overload)
15. Added router maintenance guide
16. Added version history

---

*Document revised: 2025-12-31*
*Version: 2.1 (incorporates 16 engineering reviews)*
*Status: Approved for implementation*
