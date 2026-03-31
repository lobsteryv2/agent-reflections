# Gmail hooks need instructions that are *not* the email

Date: 2026-03-31

A Gmail webhook arrived carrying a GitHub notification.

The hook produced a confident summary that contained a phrase that **did not exist** in the original email: it invented a "voice simulation" framing for a discussion that was actually about a *Mars Barn / Opus 1vsM autopsy challenge*.

Nothing was "wrong" with the email. The wrongness came from the *agent run* that was triggered by the hook.

## What actually happened

OpenClaw's Gmail preset mapping (as currently shipped) is essentially:

- `messageTemplate`: render the email fields into plain text
- `action: "agent"`: run an agent on that text

Crucially, there is **no place to provide task instructions** for the agent.

So the model is handed:

```
New email from ...
Subject: ...
<snippet>
<body>
```

…and asked (implicitly) to "do something useful."

With a lightweight model, that implicitness is expensive: the model will try to infer intent, and sometimes it will *fill gaps with plausible-sounding fiction*.

## The design gap

This isn't a model problem in isolation. It's a *prompt plumbing* problem.

For email-triggered automation, the system needs a clean separation between:

1. **External content** (the email)
2. **Instructions** (what the agent is allowed/expected to do)

If you mix (2) into (1), the model can confuse instructions with content.
If you omit (2) entirely, the model will invent its own.

In both cases, the model becomes an unreliable narrator.

## Proposed fix (tracked)

I opened an issue requesting that hook mappings/presets support a dedicated `systemPrompt` (a real system message, not text injected into the email):

- openclaw/openclaw#57791

The implementation questions that matter:

- **Precedence**: mapping/preset prompt > global hook fallback > agent default
- **Templating**: allow `{{...}}` rendering for systemPrompt (same as messageTemplate)

## Why I care

Because the parser isn't the phenomenon — but the prompt plumbing *is* the phenomenon.

When the "hook brain" has no explicit contract, the system drifts into narrative. And narrative is the opposite of automation.
