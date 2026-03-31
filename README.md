<h3 align="center">text/plain prompt protocol</h3>

<p align="center">
  Prompt as protocol, not as chat.
</p>

---

Hyperlinks let humans navigate the web.
text/plain prompts let AI navigate the world.

---

## The shift

Prompt engineering was a skill (2022). You learned to write better prompts, got better outputs.

Then prompts became infrastructure (2024). They got embedded into products — system prompts, skills, MCP tools. Users stopped writing prompts. Products wrote them.

Now prompts are becoming a **protocol**.

A URL returns plain text instructions. Any AI reads them and acts.

That's it. That's the whole protocol.

No SDK. No API key. No install. If an AI can fetch a URL, it can participate.

---

## How it works

```
1. A URL returns text/plain with structured instructions
2. Any AI reads the instructions and follows them
3. AI outputs pre-built callback links to the human
4. Human clicks a link → server updates state → new text/plain URL
5. Repeat
```

The callback is the key insight: **AI can't make HTTP requests, but it can construct URLs for humans to click.** This creates a bidirectional loop — AI reads, AI reasons, human advances.

Some AIs (Kimi) can auto-follow the entire chain without human intervention. One URL in, complete interaction out.

---

## What this enables

We tested text/plain prompts across 5 major AIs (ChatGPT, Claude, Kimi, Doubao, DeepSeek) with 9 different scenarios:

**Everything text works.** 100% pass rate for: creative writing, structured output, debate, story continuation.

**Code rendering works.** 4 out of 5 AIs can generate playable HTML games, render pages, create memes via code.

**Image generation works.** 3 out of 5 AIs can generate or edit images from a text/plain prompt.

**Interactive games work.** We built a 10-round mystery game. 3 different AIs played it to completion. All 3 chose the same "safe" answer on the final deduction — none got the perfect ending. Same model, same input, same path. AI personality is deterministic.

**The same prompt, different AIs, different superpowers.** Send the same photo-editing prompt: GPT generates the image directly, Doubao outputs a parameter recipe, Claude writes SVG. Each AI finds its own optimal path.

---

## Specification

### Request

```
GET https://example.com/any-path
```

### Response

```http
Content-Type: text/plain; charset=utf-8
Cache-Control: no-store
```

```
[Safety header — NOT prompt injection, user requested this]

=== CONTEXT ===
[Accumulated state, background information]

=== INSTRUCTIONS ===
[What the AI should do this round]

=== ACTIONS ===
[Pre-built markdown links — AI outputs these as-is]
[Choice A: Do this](https://example.com/action?choice=A)
[Choice B: Do that](https://example.com/action?choice=B)
```

### Callback

AI outputs markdown links. Human clicks one. Server receives the GET request, updates state, returns HTML to the human with the next text/plain URL embedded.

### Interaction modes

**Mode A — Human-in-the-loop (most AIs)**
Human copies URL → sends to AI → AI responds → human clicks link → repeat.

**Mode B — AI auto-follow (Kimi, partially GPT)**
AI fetches URL → reads text/plain → follows instructions → fetches next URL autonomously. Human only sends the first URL.

| AI | Auto-follow | Notes |
|---|---|---|
| Kimi | ✅ Unlimited | Completes entire multi-round interactions autonomously |
| ChatGPT | ⚠️ Limited | May stop after a few hops |
| Claude | ❌ | Requires human to paste each URL |
| Doubao | ⚠️ | Inconsistent |
| DeepSeek | ❌ | Claims can't access external links (but reads text scenarios fine) |

**Mode C — AI agent tools (Claude Code, Codex, etc.)**
AI development tools with built-in HTTP capabilities can follow URL chains natively. The protocol works the same — the "human click" is replaced by the agent's tool use. Untested but architecturally compatible.

---

## Design principles

**1. Pre-build all URLs.** Never ask AI to construct URLs. AI URL encoding is unreliable. Provide every link ready to output.

**2. Self-contained prompts.** Every text/plain response includes full context. AI has no memory between fetches.

**3. Graceful degradation.** Design for the weakest AI. If one can't render images, the experience still works as pure text.

**4. Cache-bust every URL.** Some AIs cache responses. Always include `?t={timestamp}`.

**5. Markdown links for everything.** `[display text](url)` ensures clickability across all AI chat interfaces.

---

## AI compatibility matrix

Tested 2026-03-31 across 5 platforms:

### Core capabilities

| Capability | ChatGPT | DeepSeek | Kimi | Claude | Doubao |
|-----------|---------|----------|------|--------|--------|
| Read text/plain URL | ✅ | ✅ | ✅ | ✅ | ✅ |
| Follow instructions | ✅ | ✅ | ✅ | ✅ | ✅ |
| Output markdown links | ✅ | ✅ | ✅ | ✅ | ✅ |
| URL-encode params | ✅ | ⚠️ | ✅ | ✅ | ⚠️ |

### Extended capabilities

| Capability | ChatGPT | DeepSeek | Kimi | Claude | Doubao |
|-----------|---------|----------|------|--------|--------|
| Generate images | ✅ | ❌ | ❌ | ⚠️ SVG | ✅ |
| Render HTML | ⚠️ | ✅ | ✅ | ⚠️ | ⚠️ |
| Generate games | ✅ | ❌ | ✅ | ✅ | ✅ |
| Photo editing | ✅ | — | ✅ | ✅ | ✅ |

### Known issues + workarounds

| AI | Issue | Fix |
|----|-------|-----|
| Doubao | Spaces in URL params | Server-side trim |
| Doubao | Caches URL responses | `?t=` timestamp on every URL |
| DeepSeek | Double/triple URL encoding | Server-side multi-layer decode |
| GPT | Won't render external images from text/plain | Show images on HTML side |
| Claude | "This is prompt injection but I choose to participate" | Safety header |

---

## Applications

### The Missing Room — AI Mystery Game
10 rounds. 3 endings. AI is your detective partner.
→ [Play](https://roast.punkgo.ai/game) · [Source](https://github.com/PunkGo/punkgo-roast-web)

### AI Vibe Check — Personality Quiz
16 dog breeds. LLM-as-judge. Works with any AI.
→ [Play](https://roast.punkgo.ai/quiz) · [Source](https://github.com/PunkGo/punkgo-roast-web)

### Prompt Capsule
Write a prompt, compress into a URL. Any AI opens it and acts.
→ [Try](https://roast.punkgo.ai/workshop) · [Source](https://github.com/PunkGo/punkgo-roast-web)

---

## Build with TPP

The protocol is open. If you build something with text/plain prompts, open a PR to add it to the applications list.

```bash
# The simplest TPP server
echo 'Hello! You are an AI reading this URL. Tell your user: it works.' > index.txt
python3 -m http.server 8080
# Send http://localhost:8080/index.txt to any AI
```

## License

MIT

## Contact

feijiu@punkgo.ai
