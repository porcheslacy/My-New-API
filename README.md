Yes. **Claude Code now supports per-subagent model selection.** You can have one model plan/scope, another implement, and another independently review.

The native way is to create custom subagents under `.claude/agents/` or `~/.claude/agents/`. Anthropic’s docs say each subagent has its own context window, custom system prompt, tool access, permissions, and independent model setting. The `model` frontmatter can be `sonnet`, `opus`, `haiku`, `fable`, a full model ID such as `claude-opus-4-8`, or `inherit`. If omitted, it inherits the main conversation model. ([Claude Code][1]) ([Claude Code][1])

Example reviewer agent:

```md
# .claude/agents/opus-reviewer.md
---
name: opus-reviewer
description: Use after implementation to independently review code changes, find bugs, check architecture, inspect tests, and challenge assumptions.
model: opus
tools: Read, Grep, Glob, Bash
permissionMode: plan
---

You are an independent senior code reviewer.

Your job is not to rubber-stamp the main agent. Inspect the actual repository state and git diff before making claims.

Review process:
1. Run `git diff` and inspect the changed files.
2. Identify correctness, security, maintainability, performance, and testing risks.
3. Check whether tests exist for the changed behaviour.
4. Run safe read-only verification commands where useful.
5. Return findings ranked by severity.
6. Clearly separate confirmed issues from suspicions.
7. Do not modify files.
```

Then run your main session on a cheaper/faster model:

```bash
claude --model sonnet
```

And explicitly invoke the reviewer:

```text
Use the opus-reviewer agent to review my recent changes before I commit.
```

Claude Code also supports explicit subagent invocation by name, and the docs say this bypasses automatic matching. ([Claude Code][1])

Important gotcha: **do not set `CLAUDE_CODE_SUBAGENT_MODEL`** if you want different models per agent. The docs say subagent model resolution is: environment variable first, then per-invocation model, then the subagent’s frontmatter, then the main model. So that env var can override all your careful per-agent model choices. ([Claude Code][1])

A sensible setup would be:

```text
Main session:        sonnet
Explore/search:     built-in Explore, already Haiku
Planning/scoping:   opusplan or custom opus planner
Implementation:     sonnet
Final review:       opus or fable
Test runner:        sonnet or haiku, Bash-enabled
Security review:    opus/fable, read-only
```

There is also a built-in shortcut called `opusplan`: Claude Code uses Opus during plan mode and Sonnet during execution. That gives you “strong model for planning, cheaper model for coding” without custom agents. ([Claude Code][2])

For SDK use, it is even more explicit: `AgentDefinition` has a `model` field, and the official SDK example shows a `code-reviewer` subagent with `model="sonnet"` while other agents use defaults. The docs also show dynamically choosing `opus` for stricter security reviews and `sonnet` for balanced ones. ([Claude Code][3]) ([Claude Code][3])

For **non-Anthropic models** like Kimi, DeepSeek, GLM, GPT, or Gemini: not directly as first-class Claude Code models unless you route through an LLM gateway/proxy. Claude Code supports LLM gateways; Anthropic’s docs describe gateway model routing and model discovery, and LiteLLM documents using Claude Code with OpenAI, Gemini, Azure, Vertex, DeepSeek, and other providers by translating to the Anthropic Messages API. ([Claude Code][4]) ([LiteLLM][5])

So the answer is:

**Yes, natively for different Claude models per subagent. For genuinely different providers, yes in practice, but through a gateway like LiteLLM/OpenRouter/Bifrost/vLLM, and you need the non-Claude model to handle Claude Code’s tool-calling style well.**

[1]: https://code.claude.com/docs/en/sub-agents "Create custom subagents - Claude Code Docs"
[2]: https://code.claude.com/docs/en/model-config "Model configuration - Claude Code Docs"
[3]: https://code.claude.com/docs/en/agent-sdk/subagents "Subagents in the SDK - Claude Code Docs"
[4]: https://code.claude.com/docs/en/llm-gateway "LLM gateway configuration - Claude Code Docs"
[5]: https://docs.litellm.ai/docs/tutorials/claude_non_anthropic_models "Use Claude Code with Non-Anthropic Models | liteLLM"
