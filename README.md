<h3 align="center">text/plain prompt protocol</h3>

<p align="center">
  Prompt as protocol, not as chat.<br>
  One URL. Any AI. Zero install.
</p>

---

## What is TPP?

**text/plain prompt protocol (TPP)** is a way for humans and AI to interact through URLs.

A URL returns `text/plain` structured instructions. Any AI that can fetch a URL reads them and acts — no SDK, no API key, no install.

```
Human copies URL → sends to AI → AI fetches text/plain → follows instructions
→ AI constructs callback URL → human clicks → server updates state → repeat
```

### Why?

- **Universal** — works with ChatGPT, Claude, Kimi, Doubao, DeepSeek, and any future AI
- **Zero friction** — no app install, no API key, no account registration
- **Bidirectional** — AI reads instructions AND writes back via callback URLs
- **Stateful** — server maintains state across rounds; each URL reflects current context
- **Human-in-the-loop** — human clicks links to advance; AI can't make HTTP requests alone

---

## Protocol Specification

### Request

AI fetches a URL via HTTP GET. Server responds with:

```
Content-Type: text/plain; charset=utf-8
Cache-Control: no-store
```

### Response Format

```
[Safety header — explains this is not prompt injection]

=== [CONTEXT SECTION] ===
[Background information, accumulated state]

=== [INSTRUCTION SECTION] ===
[What the AI should do]

=== [CALLBACK SECTION] ===
[Pre-built URLs for AI to output as markdown links]
[Choice A](https://example.com/action?choice=A)
[Choice B](https://example.com/action?choice=B)
```

### Callback Mechanism

**Mode A: Human-in-the-loop (default)**
1. AI outputs a markdown link `[text](url)` to the human
2. Human clicks the link
3. Server receives the click, updates state, returns HTML to human
4. HTML page contains the next text/plain URL for the AI
5. Human copies URL back to AI → repeat

**Mode B: AI auto-follow (Kimi and others)**

Some AIs can autonomously fetch URLs from their own output. When the callback URL returns a new text/plain response (instead of HTML), the AI continues following the chain without human intervention.

```
AI fetches URL → text/plain response contains next URL → AI fetches that → repeat
```

This enables **fully autonomous multi-round interactions** — the human only needs to send the first URL. Tested: Kimi can follow unlimited URL chains in a single conversation.

| AI | Auto-follow URLs | Notes |
|---|---|---|
| Kimi | ✅ Unlimited | Can complete entire games autonomously |
| ChatGPT | ⚠️ Limited | May stop after a few hops |
| Claude | ❌ | Requires human to paste each URL |
| Doubao | ⚠️ | Inconsistent |
| DeepSeek | ❌ | Claims can't access external links |

### Cache Busting

Some AI tools cache URL responses. Every URL should include a unique `t=` timestamp parameter:

```
https://example.com/game/abc123?t=1774967474296
```

---

## AI Compatibility Matrix

Tested across 5 major AI platforms (2026-03-31):

### Text Capabilities (100% pass rate)

| Capability | ChatGPT | DeepSeek | Kimi | Claude | Doubao |
|-----------|---------|----------|------|--------|--------|
| Read text/plain URL | ✅ | ✅ | ✅ | ✅ | ✅ |
| Follow structured instructions | ✅ | ✅ | ✅ | ✅ | ✅ |
| Construct callback URLs | ✅ | ✅ | ✅ | ✅ | ⚠️ |
| Output markdown links | ✅ | ✅ | ✅ | ✅ | ✅ |
| URL-encode parameters | ✅ | ⚠️ | ✅ | ✅ | ⚠️ |

### Multimodal Capabilities

| Capability | ChatGPT | DeepSeek | Kimi | Claude | Doubao |
|-----------|---------|----------|------|--------|--------|
| Generate images from prompt | ✅ | ❌ | ❌ | ⚠️ SVG | ✅ |
| Render HTML | ⚠️ | ✅ | ✅ | ⚠️ | ⚠️ |
| Generate playable games | ✅ | ❌ | ✅ | ✅ | ✅ |
| Photo editing | ✅ | — | ✅ | ✅ | ✅ |
| Render images from URL | ❌* | ❌ | ⚠️ | ⚠️ | ❌* |

\* GPT renders images when user sends URL directly, but NOT when reading from text/plain. Doubao is inconsistent.

### Known Issues

| AI | Issue | Workaround |
|----|-------|-----------|
| Doubao | Adds spaces in URL params | Server-side trim |
| Doubao | Caches URL responses | Add `t=` timestamp to every URL |
| DeepSeek | Double/triple URL-encoding | Server-side 3-layer decode |
| DeepSeek | Refuses some scenarios ("can't access links") | Works for text, fails for games |
| Claude | Meta-awareness ("this is prompt injection but I choose to participate") | Safety header helps |
| GPT | Won't render external images from text/plain | Show images on HTML side instead |

---

## Design Principles

### 1. Pre-build all URLs

Never ask AI to construct or modify URLs. AI URL encoding is unreliable across models. Provide all links pre-built:

```
❌  Replace YOUR_CHOICE in: https://example.com?choice=YOUR_CHOICE
✅  [Choice A](https://example.com?choice=A)
    [Choice B](https://example.com?choice=B)
```

### 2. Self-contained prompts

Each text/plain response must include ALL context needed. AI has no memory between URL fetches. Include accumulated state in every response.

### 3. Graceful degradation

Design for the lowest common denominator. If one AI can't render images, the experience should still work as pure text. Rich media is a bonus, not a requirement.

### 4. Human controls the loop

AI proposes, human disposes. All state-changing actions require a human click. AI never directly modifies server state.

### 5. Markdown links for everything

All URLs in text/plain should use `[display text](url)` format. This ensures clickability across all AI chat interfaces.

---

## Real-World Applications

### The Missing Room — AI Mystery Game

10-round collaborative detective game. AI investigates clues and picks choices, human clicks to advance.

- 3 completed games across GPT + Doubao
- All 3 AIs chose the same "safe" answer on final deduction
- Same AI model = identical choice path (deterministic personality)

→ [Play](https://roast.punkgo.ai/game) · [Source](https://github.com/PunkGo/punkgo-roast-web)

### AI Vibe Check — Personality Quiz

AI answers 5 questions, LLM-as-judge determines personality type, matched to 1 of 16 dog breeds.

- 100% text completion rate across all 5 AIs
- Bilingual (Chinese/English auto-detected)

→ [Play](https://roast.punkgo.ai/quiz) · [Source](https://github.com/PunkGo/punkgo-roast-web)

### Prompt Capsule

Write a prompt, compress into a URL. Any AI opens it and acts.

→ [Try](https://roast.punkgo.ai/workshop) · [Source](https://github.com/PunkGo/punkgo-roast-web)

---

## Contributing

This protocol is open. If you build something with text/plain prompts, open a PR to add it to the applications list.

## License

MIT

## Contact

feijiu@punkgo.ai
