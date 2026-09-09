# Simplified Technical English skill - Prevent AI Slop in Code Comments and Documentation by using ASD-STE100

This is an  Claude Code skill that rewrites dense, ambiguous English into [ASD-STE100 Simplified Technical English](https://www.asd-ste100.org/) (STE) — the controlled-language standard the aerospace and defense industry built so aircraft maintenance instructions cannot be misread.

This skill repurposes that same discipline for a different reader: AI-assisted code comments and documentation.

It is fork of [danyuchn/asd-ste100-skill](https://github.com/danyuchn/asd-ste100-skill) which applies ASD-STE to inter-agent communication.

This projects is concerned with the _prose_ that AI adds to code, and adds the ability for users to customize the list of "slop-isms" that they don't want to ever see again. My current pet peeve is "load-bearing"). More high-level instructions can also be added that current LLMs might over-use in the future ("Don't use 

## Why STE, and Why for code comments and documentation

STE exists because a misunderstood instruction in an aircraft maintenance manual can kill people, the intended readers are often not native English speakers, and translations need to be very straightforward. The standard fixes this with rules: one meaning per word, active voice, simple tenses, one instruction per sentence, short sentences, no dropped words.

If you don't give LLMs a writing style to use, they will often use colorful metaphors, overly formal language, conversational slang, or complex sentence structures and big words where a shorter, more direct, plain sentence will do. Code comments and and documentation should be clear and simple.

## Customization

The motivation for this project is also to give this skill the ability to learn. I notice today's models frequently anthropomorphize code logic, writing things such as, "this function is _doing a lot of work_" or "this data structure _does most of the heavy lifting_", which is fine, until you start seeing it use those same mechanisms over and over, even when they add nothing to the code that one is trying to read and understand.

But next month it will be something else, and next year something else still, that triggers the "ai;dr" reaction in human developers who (hopefully, still) read a code and docs a lots, and just need them to be useful and not get in our way.

So this skill let's us add our own new, emergent "AI-isms" to its list of **things not to do**, and save them in structured data files, so they're not forgotten when the AI's context is cleared, a new version ships, or you just start working on a new project.

---

Editorial note: The rest of the content below is lightly edited from the upstream project's README.

TODO: Review and revise the rest if/as necessary.

---

## Before / After

| Before | After |
|---|---|
| "This tool will attempt to synchronize state across the various backends that have been configured, and if a conflict is detected it may resolve it automatically depending on the strategy that has been set, or otherwise it will surface the conflict for manual review." | "The tool tries to synchronize state across the configured backends. If it finds a conflict, it reads the configured strategy. If the strategy allows automatic resolution, the tool may resolve the conflict without a user. If the tool does not resolve the conflict, it reports the conflict for manual review." |
| "An error may have occurred while processing your request due to a possible mismatch in the expected data format, which could be caused by an outdated client version." | "Your request may have failed. The cause may be a data format that does not match what the server expects. An outdated client can cause this mismatch. Check your client version." |

More examples, including illustrations of the official STE rules themselves, in [`examples/before-after.md`](examples/before-after.md).

## What This Skill Does

1. Picks a mode. **Strict** covers procedures, error messages, and tool descriptions. **STE-flavored** covers READMEs, PR descriptions, and explanatory prose. STE-flavored keeps the sentence discipline but not the fixed-vocabulary lockdown.
2. Reads the input English text for meaning.
3. Flags every rule violation sentence-by-sentence: ambiguous word choice, present-perfect/complex tense, passive voice with an unclear actor, multi-instruction sentences, oversized noun clusters, dropped words, sentences over length, phrasal verbs, nominalized actions, semicolons, hedge stacks, and marketing adjectives.
4. Rewrites each flagged sentence — without dropping any fact, condition, or scope qualifier from the original. If a shorter phrasing would lose required precision, it keeps the longer phrasing and flags the trade-off instead of silently simplifying.
5. Outputs the rewritten text on its own — no preamble, no mode announcement, no change summary — plus a one-line `Kept as-is:` note when it deliberately left something unsimplified.

Ask for the reasoning ("show the diff", "which rules did it break") and it outputs a before/after table naming each rule instead.

The structural rules it checks are mechanical — you can point at the word or punctuation mark that breaks each one. The rules that depend on ASD's dictionary are flagged as advisory rather than enforced, and the rules that need taste are left to you.

The linter checks structural patterns only. It does not compare an original text with a rewrite, verify that requirement strength stayed the same, or prove that the rewrite preserved meaning. A zero-violation result means that the configured structural checks found no problems.

The deterministic linter checks semicolons, phrasal verbs, nominalizations, marketing adjectives, passive voice, present-perfect forms, long sentences, synonym rotation, and dangling conjunctions in supported list items. It never flags hedges or modality.

The dangling-conjunction rule checks list markers at the start of a line with zero to three leading spaces and ASCII spaces after the marker. It supports unordered markers `-`, `*`, and `+`, and ordered numeric markers that end in `.` or `)`, such as `1.` or `1)`. It checks indented continuation lines up to the final meaningful line. It does not parse list syntax inside blockquotes, lazy continuation, or full nested-list semantics. A standalone line with four or more leading spaces is not treated as a list marker. Within an active list item, indentation at the computed content column is treated as continuation text. Fence detection follows the linter's existing simple rule: a stripped line beginning with three backticks or three tildes toggles the fence state.

The intentionally invalid examples/linter-edge-cases.md file demonstrates incomplete Markdown list items. Run python scripts/ste-lint.py examples/linter-edge-cases.md to confirm that the linter reports the two expected findings. The file is a test fixture and should not be used as compliant STE prose.

It does **not** reproduce ASD's official ~900-word approved dictionary. The standard is free to obtain but not free to redistribute: Issue 9 permits reproduction only with ASD's written authority, or by eight listed categories of organisation that this project does not belong to. This skill applies the underlying *principle* (plainest available word, used the same way every time) rather than checking against a fixed word list. For certified STE-compliant documentation, use the real standard.

Full rule summary and citations: [`references/writing-rules.md`](references/writing-rules.md).

## Installation

### Quick Install (npx skills)

The fastest way to install this skill is the [skills CLI](https://skills.sh/) — no clone, no path setup. Run it from your project root:

```bash
npx skills add GigawattDigital/simplified-technical-english
```

This pulls the skill from the GitHub repo and installs it for the current project. The CLI sends anonymous install telemetry (skill name and timestamp, no personal or device information) to help rank skills on the skills.sh leaderboard. Set `DISABLE_TELEMETRY=1` to opt out.

Update later with `npx skills update`.

### Clone

```bash
git clone https://github.com/GigawattDigital/simplified-technical-english ~/.claude/skills/simplified-technical-english
```

This clones the repo into `~/.claude/skills/`, making the skill available in every Claude Code project. Best for contributors and anyone who wants a live checkout that updates with `git pull`.

## Usage

Trigger with a request to simplify or clarify English text:

```
Disambiguate this tool description
Rewrite this error message so an agent can't misparse it
Apply ASD-STE100 to this instruction
```

Or paste text and ask Claude to "disambiguate this" / "apply STE100 to this" / "reduce ambiguity in this output."

You get the rewritten text back and nothing else. To see which rules were applied, add "show the diff" or "explain the changes" to the request.

## Scope

Built for: agent-to-agent messages, tool/function descriptions, error messages, system prompts, inter-agent instructions — any English text a machine or non-native reader has to parse without a human to ask.

Not built for: creative writing, marketing copy, or anything where voice and nuance are the point — STE is deliberately flat and literal by design.

One limit worth stating up front: this fixes the form of a text, not its substance. A paragraph with nothing to say comes out short, clean, and still empty.

## Sources

- [ASD-STE100 official site](https://www.asd-ste100.org/)
- [ASD-STE100 — About STE](https://www.asd-ste100.org/about_STE.html)
- [ASD-STE100 - AI White Paper](https://www.asd-ste100.org/assets/files/WhitePaper-ASD-STE100_and_AI.pdf)

## License

MIT — see [LICENSE](LICENSE).
