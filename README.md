# bookbabel

> **Translate technical books and long-form articles across languages — preserving structure, glossary, and style. Built for LLM workflows.**

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
![Status](https://img.shields.io/badge/status-Phase%200%20%E2%80%94%20Design-orange)

bookbabel is a translation framework for **books and long-form articles**, optimized for LLM-based pipelines. Unlike single-shot LLM calls, bookbabel preserves Markdown / EPUB structure, enforces a shared glossary across chapters, and validates the output against the source — so a 30,000-word book can be translated into 5 languages without losing headings, code blocks, or terminology consistency.

> **Status**: Phase 0 — design and proof of concept. The author is actively dogfooding this as part of running a multilingual book publishing pipeline (4 languages, 50+ titles). Early feedback welcome.

---

## Why does this exist?

Technical book authors and indie publishers hit the same walls when translating long-form content with LLMs:

- **Structure breaks.** Headings get reordered, code fences lose backticks, tables flatten into prose, frontmatter disappears.
- **Glossary drifts.** "Context window" gets rendered three different ways across chapters. "Tensor Core" turns into "テンソル コア", "テンソルコア", and "Tensorコア" in the same book.
- **Hallucinations sneak in.** LLMs invent facts in long contexts — fictional citations, misattributed quotes — and there is no automated catch.
- **Token costs explode.** Naive prompting re-sends the glossary and style guide every chapter. A 30k-word book × 5 languages × 3-5 retries balloons into hundreds of dollars.
- **Voice erodes.** First-person tone, formality level, and AI-tell phrases all drift between chapters.

bookbabel addresses these as **first-class concerns**, not afterthoughts.

---

## Architecture

```
   Source (Markdown / EPUB / Zenn frontmatter)
              │
              ▼
     ┌─────────────────┐
     │   Glossarizer   │  extract / maintain shared glossary
     └─────────────────┘
              │
              ▼
     ┌─────────────────┐
     │   Translator    │  LLM call with cached glossary + style guide
     └─────────────────┘
              │
              ▼
     ┌─────────────────┐
     │    Verifier     │  structure diff, NER preservation, AI Slop scan
     └─────────────────┘
              │
              ▼
   Target language outputs (per format: Kindle EPUB, Zenn, Astro, etc.)
```

Each stage is a self-contained module so you can swap the LLM backend, plug in a custom verifier, or run only part of the pipeline.

---

## Planned features

- [ ] **Glossary-first translation** — terms locked across chapters via a single `glossary.json`
- [ ] **Structure preservation** — Markdown headings, code fences, tables, frontmatter, image refs all round-trip
- [ ] **Structural diff verification** — heading count, code-block parity, table-row parity, image-ref parity
- [ ] **Hallucination detection** — named-entity preservation check, fictional citation scan
- [ ] **Multi-language fan-out** — one source → N targets translated in parallel
- [ ] **Prompt cache integration** — Anthropic prompt caching for 10-50x token reduction on repeated context
- [ ] **AI Slop detection per language** — pattern-based stylistic-residue removal in JA / EN / ES / PT-BR / KO
- [ ] **Format adapters** — Kindle EPUB, Zenn Book (config.yaml + chapter Markdown), Astro Markdown, generic Markdown
- [ ] **Pluggable LLM backend** — Claude (default), GPT, Gemini
- [ ] **CLI + library API** — automate via shell or import as a Python module

---

## Planned CLI

```bash
# translate one book to three languages with a shared glossary
bookbabel translate src/book.md --to es,pt-br,en --glossary glossary.json

# verify a translated chapter against its source
bookbabel verify es/chapter-03.md --against src/chapter-03.md

# build a Kindle-ready EPUB from a translated tree
bookbabel publish --target kindle --lang es --out dist/book-es.epub
```

---

## Why not just use existing tools?

|  | bookbabel | DeepL | gpt-translator scripts | Lokalise / Crowdin / Transifex |
|---|---|---|---|---|
| Long-form (book-scale) | ✅ | ⚠️ chunk-size limits | ⚠️ no glossary lock | ❌ string-table only |
| Glossary lock | ✅ | partial | ❌ | ✅ (manual) |
| Markdown / EPUB structure | ✅ | ❌ | ⚠️ | ❌ |
| Hallucination detection | ✅ | ❌ | ❌ | ❌ |
| Multi-target parallel | ✅ | ⚠️ sequential | ❌ | ✅ |
| LLM-native pricing | ✅ pay-per-token | ❌ subscription | ✅ | ❌ subscription |
| OSS | ✅ Apache 2.0 | ❌ | varies | ❌ |

If you only translate UI strings or short-form copy, Lokalise/Crowdin are great. If you translate one short article occasionally, DeepL is great. bookbabel is for the **30k-word technical book that must ship in 4 languages without melting your token budget or your prose voice**.

---

## Roadmap

| Phase | Scope | Target |
|---|---|---|
| **Phase 0** *(current)* | Architecture spec, glossary format design, Markdown PoC | mid-2026 |
| Phase 1 | CLI release, EPUB support, structure verifier | summer 2026 |
| Phase 2 | Multi-language parallel fan-out, Anthropic prompt-cache integration | autumn 2026 |
| Phase 3 | Kindle / Zenn / Astro publishing adapters, web UI | late 2026 / early 2027 |

---

## Author & origin

bookbabel is built and dogfooded by **[@kenimo49](https://github.com/kenimo49)** (ken imoto, [kenimoto.dev](https://kenimoto.dev)), who runs a Kindle publishing pipeline across 4 languages (JA / EN / ES / PT-BR), an 8-language web framework site ([llmoframework.com](https://llmoframework.com)), and 50+ technical books on Zenn. The translation pain encountered while running that pipeline directly drives the design choices here.

In other words: the author is the worst-case user, and the tool exists because no existing solution survived contact with a real multilingual book pipeline.

---

## Contributing

Phase 0 is **design discussion**. The fastest way to contribute right now:

1. Open an issue with your translation horror story (so the verifier knows what to catch)
2. Comment on the Phase 0 design issue once it lands
3. Star the repo if you want to be notified when Phase 1 ships

---

## License

[Apache 2.0](LICENSE) — commercial use, modification, distribution, and patent-grant friendly.
