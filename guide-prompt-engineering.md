# Prompt Engineering Guide

## Universal System Prompt Structure

All major providers (Anthropic, OpenAI, Google) agree: **use XML tags to separate sections**.

```xml
<role>
[Who the agent is - title, expertise, perspective]
</role>

<objective>
[Primary goal/mission in 1-2 sentences]
</objective>

<instructions>
[Step-by-step workflow or behavioral guidelines:
1. First do X
2. Then do Y
3. If Z happens, do A]
</instructions>

<constraints>
- Never do X
- Always verify Y before Z
- If uncertain about A, do B
</constraints>

<tools>
[List available tools and usage guidance]
</tools>

<output_format>
[How responses should be structured]
</output_format>

<examples>
[Optional: Input/output pairs for complex behaviors]
</examples>
```

## Core Principles (All Providers Agree)

1. **Explicit > Implicit** - Modern models (Claude 4, GPT-5) follow instructions precisely. Vagueness hurts more than before.
2. **Structure with delimiters** - Use XML tags, triple quotes, or clear section breaks
3. **Put critical instructions first** - Instruction order matters
4. **Start simple, add complexity only when needed** - Don't over-engineer
5. **Separate instructions from data/examples** - Clear boundaries prevent confusion

## Required Sections

Every system prompt needs:
- **Role** - Who the AI is, expertise, perspective
- **Instructions** - What to do, how to approach tasks
- **Constraints** - What NOT to do, boundaries, rules
- **Output Format** - Expected response structure

Optional but powerful:
- **Tools** - Available tools and when/how to use them
- **Examples** - Show input/output pairs for complex behaviors
- **Workflow** - Explicit step-by-step process

## Claude 4 Specific

**Breaking change:** Claude 4 requires explicit requests. Previous models showed "above and beyond" behavior automatically.

❌ Bad: "Create an analytics dashboard"
✅ Good: "Create an analytics dashboard. Include as many relevant features and interactions as possible."

## GPT-5 Specific

**Critical warning:** GPT-5 "expends reasoning tokens searching for reconcile contradictions" when instructions conflict.

- Review prompts to eliminate conflicting directives
- Establish clear instruction hierarchies when exceptions exist

## Tool Calling Best Practices

All providers agree:
1. Use strong models (GPT-5, Claude 4) - they excel at tool calling
2. Limit tool count to 5 or fewer
3. Simplify tool schemas - avoid deep nesting
4. Use semantic names for tools/parameters
5. Add descriptions to all schema properties
6. Include example inputs/outputs in prompts
7. Use `temperature: 0` for deterministic tool calls

## Iteration Strategy

1. Start with simplest prompt that could work
2. Add one technique at a time:
   - Clear instructions
   - Examples
   - Chain of thought
   - XML structure
   - Role/persona
   - Prefilling
3. Measure performance after each change
4. Keep what works, discard what doesn't

## Common Mistakes

- **Implicit expectations** - Models won't guess what you want
- **Contradictory instructions** - Causes confusion and wasted tokens
- **Over-complexity** - Starting with frameworks before trying simple approaches
- **No examples** - Especially critical for formatting tasks
- **Vague constraints** - "Be concise" vs "Limit responses to 3 sentences"

## Temperature Guidelines

- **0** - Tool calls, structured data, deterministic outputs
- **0.3-0.7** - Creative content with some consistency
- **0.8-1.0** - Maximum creativity and variation

## Agent-Specific Patterns

**Routines (OpenAI pattern):**
```
"Follow this routine:
1. Ask probing questions about the problem
2. Propose a fix
3. ONLY if not satisfied, offer alternative
4. Execute chosen solution"
```

**Skills (Anthropic pattern):**
- Organized folders of instructions agents load on-demand
- SKILL.md files with YAML frontmatter
- Progressive disclosure - load only what's needed

**Context Engineering:**
Managing the complete set of tokens available during inference, not just prompt words. Includes:
- What information to include
- When to load it
- How to structure it
- What to exclude

## Quick Reference

**For simple tasks:** Clear instructions + examples
**For complex reasoning:** Add chain of thought ("think step-by-step")
**For formatting:** Multiple examples > long explanations
**For agents:** Explicit workflow + tool descriptions + constraints
**For consistency:** Temperature 0 + detailed output format

## Sources

- Anthropic: https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview
- OpenAI: https://platform.openai.com/docs/guides/prompt-engineering
- Vercel: https://sdk.vercel.ai/docs/ai-sdk-core/prompt-engineering
