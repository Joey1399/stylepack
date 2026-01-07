# stylepack

Local-first **style profiling + style packaging** for LLM workflows.  
Turn your existing **code samples, writing samples, articles, and snippets** into a compact **Style Profile** that helps LLM outputs match your “voice” and conventions—without pasting giant examples into every prompt.

---

## Why this exists

LLM output often feels “generic”:
- code uses odd conventions that don’t match your repo
- writing lacks voice/cadence and reads obviously AI-generated
- dumping lots of examples into prompts is expensive, noisy, and inconsistent

**stylepack** aims to solve that by **distilling style into a small, reusable bundle** that can be injected into prompts or exported into editor/agent instruction files.

---

## What stylepack generates

Given a folder of samples, stylepack produces:

### 1) `style.md` — Human-readable Style Profile
A concise style guide:
- do/don’t rules
- tone, structure, naming, comment patterns
- preferred idioms and “house style”
- a small number of short “golden exemplars” (high signal, not giant dumps)

### 2) `style.json` — Machine-readable StyleSpec
A structured representation of style:
- detected conventions + confidence
- rule weights/priorities
- exemplar pointers
- optional language-specific sections

### 3) Optional exports (editor/agent integrations)
- **Cursor rules export**: generates rules files under `.cursor/rules/`
- **GitHub Copilot instructions export**: writes `.github/copilot-instructions.md`

### 4) Optional formatter configuration
- For C/C++: best-effort `.clang-format` emitted/tuned to match observed formatting
- For Python: Black guidance/config stub (Black is intentionally opinionated)

---

## Supported inputs (planned)

- Code repos (C++, Python, JS/TS, etc.)
- Markdown/docs/articles
- Plain text notes
- “Golden files” list (hand-picked best examples)
- “Anti-examples” list (things you explicitly *don’t* want)

---

## How it works (high-level)

stylepack is designed to separate **style** from **substance**:

1. **Ingest**
   - collect samples + metadata (paths, languages, timestamps)
2. **Normalize**
   - extract text/code in a consistent internal format
3. **Signal extraction**
   - formatting cues (indent/braces/wrapping)
   - naming conventions (snake_case vs camelCase, acronyms, prefixes)
   - structural habits (function size, helper patterns, file organization)
   - comment/documentation patterns (docstrings, headers, inline comments)
   - for prose: tone cues, cadence, punctuation habits, section structure
4. **Distillation**
   - generate a compact Style Profile (rules + exemplars)
   - avoid “overprompting” by keeping exemplars short and representative
5. **(Optional) Apply + Validate**
   - generate a style-aware preamble for any LLM prompt
   - score a generated output for “style adherence” and flag violations

---

## Example project layout (planned)

stylepack/

stylepack/

main.py

- cli.py

ingest/

signals/

distill/

export/

score/

examples/

tests/

README.md


-------------------------------------------------------------------------------------------------------------------------------------------

---

## CLI sketch (planned)

> These are placeholder commands for the future implementation.

- `stylepack init ./samples --lang cpp`
  - generates `style.md` + `style.json`
- `stylepack export cursor`
  - writes Cursor rules into `.cursor/rules/`
- `stylepack export copilot`
  - writes `.github/copilot-instructions.md`
- `stylepack build-preamble --budget 600`
  - prints a compact “style preamble” for prompt injection
- `stylepack score --generated out.txt --reference ./samples`
  - reports style adherence + top violations

---

## What “style” means here (so it’s not just formatting)

Formatters handle surface formatting well, but stylepack targets higher-level conventions:

### Code
- naming conventions (identifiers, files, modules)
- preferred structure (early returns, helper layout, error handling patterns)
- comment / doc style (tone, density, doc templates)
- architectural idioms (how you organize “small units”)

### Writing
- voice/tone (formal vs casual, direct vs narrative)
- cadence (sentence length patterns, punctuation habits)
- structural templates (how intros/conclusions are written)
- vocabulary bias (preferred words/phrases, “avoid list”)

---

## Guardrails (recommended)

This kind of tool can be abused for impersonation. Intended use:
- profiling your **own** writing/code
- profiling your **team’s** repo with permission
- creating a brand voice you own

Suggested design choices:
- provenance logs (what sources contributed to which rules)
- explicit confirmation that the user has rights to the samples

---

## Roadmap ideas (future you)

- Language plugins: `cpp`, `python`, `js`, `markdown`
- “Style conflicts” detector (mixed conventions inside a repo)
- Auto-tuning loop for `.clang-format` based on diff minimization
- “Voice dial” (more/less formal/terse) while preserving signature
- CI integration: score diffs against StyleSpec; fail if under threshold
- IDE extension: show “why” a rule fired + quick fixes

---

## Non-goals (for sanity)

- Not trying to replace linters/formatters
- Not trying to fully “train a model” (this is prompt + workflow tooling)
- Not trying to guarantee perfect voice cloning—goal is **consistent style adherence**




--------------------------------------------------------------------------------------------------------------------------------------------







#PRODUCT BRIEF/NOTES

A) PROBLEM
- LLM-generated code often arrives in a “default model style” that differs from a developer/team’s established conventions.
- This creates measurable friction:
  • Higher cognitive load: code looks unfamiliar, harder to review and maintain.
  • Review churn: PR comments become “nit/style bikeshedding” instead of substantive feedback.
  • Style drift: LLM-assisted changes introduce inconsistent patterns across the codebase.
  • Lower trust/adoption: devs reject otherwise-correct output because it doesn’t match house style.
- Existing formatters/linters help after the fact, but do not reliably:
  • steer generation toward repo conventions, or
  • encode higher-level style (naming, error handling, docstrings, module structure), or
  • prevent semantic/logic “template reuse” when examples are provided.




B) SOLUTION + SCOPE

Solution concept: A versioned, reusable “Style Profile” (aka style policy) per language that is applied during LLM-assisted
generation and refactoring. The Style Profile is created by compiling a user/team’s code corpus into constraints and
configuration (style transfer without semantic transfer).

Core principles:
- Corpus-to-policy compilation: extract style rules/features from code into a compact Style Spec, not raw “reference code”.
- Constrained generation + deterministic enforcement:
  • generation-time guidance (soft constraints)
  • post-gen enforcement via formatters/linters/codemods (hard constraints)
- Scope + precedence model: Org / Repo / User profiles with override rules.
- Versioning + governance: profiles are versioned, auditable, and pin-able to repos/branches; enterprise profiles support RBAC/approval.

1B) SMALL (1–2 USER) CODEBASES — PERSONAL CONSISTENCY

Goal: Make output look like “my code” so it’s immediately readable and maintainable.
Scope:
- Personal Style Profile per language (e.g., Python, TS).
- Toggle on/off per session/workspace; optional strictness (Guided vs Strict).
- Apply profile in chat/IDE generation, refactors, and code reviews.
- Default to style-only mode (prevents copying the “meat” of the corpus).

2B) ENTERPRISE (MULTI-USER) CODEBASES — ORG STANDARDIZATION

Goal: Enforce a single house style across many developers and LLM contributions.
Scope:
- Org Style Profile per language, with repo-level overrides and user-level preferences where allowed.
- CI gate: “must comply with org/repo style profile” (formatter/linter required).
- Migration tooling: safe, staged normalization of heterogeneous codebases via batch PRs (format -> lint autofix -> codemods).
- Governance: RBAC, approvals, version pinning, and change logs for style policy updates.




C) USER FLOWS (CREATE + USE FROM USER POV)
1) Create a Style Profile (Onboarding)
- User selects:
  • language(s)
  • repo(s) or sample files/snippets
  • optional: existing formatter/linter configs (Black/Prettier/gofmt/ESLint/Ruff/etc.)
- System outputs:
  • Style Card (human-readable summary of detected conventions)
  • Style Spec (machine-readable policy + references to tooling configs)
  • optional “strictness” defaults and overrides

2) Use a Style Profile (Generation)
- In chat/IDE:
  • choose active profile scope: Org / Repo / User
  • toggle “Apply style profile”
  • choose strictness:
    - Guided: best-effort style, minimal enforcement
    - Strict: run formatter/linter/codemod; fail or auto-fix until compliant
- Output is returned as:
  • code diff/patch + final formatted result
  • optional style compliance report (“what was adjusted and why”)

3) Reference Mode vs Style-Only Mode (Safety against “meat theft”)
- Default: Style-only mode (policy extracted; raw example code not injected into generation context).
- Optional: Reference mode (user explicitly allows retrieving specific files/snippets as reference), with:
  • clear UI indicator
  • traceability (“which references were used”)

4) Enterprise Migration Flow (Normalize an Existing Codebase)
- Admin selects target Org profile + repo(s).
- System proposes a staged plan:
  • Stage 1: formatting/imports only (non-semantic)
  • Stage 2: lint autofixes
  • Stage 3: AST codemods (opt-in; guarded by tests)
- System generates batch PRs with CI checks + rollback support.




D) SUCCESS METRICS
Quality/Compliance
- First-pass compliance rate: % of LLM outputs passing repo lint/format checks without manual edits.
- Style compliance score: adherence to Style Spec (naming, docstrings, patterns) beyond pure formatting.

Developer Productivity / Adoption
- PR review churn: reduction in comments labeled “nit/style” and time spent on style feedback.
- Time-to-merge for LLM-assisted PRs.
- Suggestion acceptance rate: % of proposed patches applied with minimal edits.
- “Style profile enabled” retention: how often users keep it on for a repo/workspace.

Enterprise Outcomes
- Codebase consistency index over time (variance in style signals decreases).
- Migration success: % of batch PRs merged; regression rate (test failures/rollbacks).
- Policy governance health: frequency of style profile changes, approval latency, and breakage incidents.


EXAMPLES

Output A (Style Profile: “minimal, concise, fewer comments”)
![IMG_1893](https://github.com/user-attachments/assets/67aa19aa-e103-4843-98a0-90fe12a14d25)


Output B (Style Profile: “typed, explicit, guard clauses, Google docstrings”)
![IMG_1892](https://github.com/user-attachments/assets/30240f55-1cf7-4b7e-b4ff-4381dacacba1)


-------------------------------------------------------------------------------------------------

### CORE ARCHITECTURE

## Do I have a good idea of how to do this? (Yes) — Technical plan for a Style Profiling + “Style Pack” Tool

The goal is to let a developer or writer provide samples (code, articles, snippets) and produce a **compact, reusable Style Profile** that:
- reliably nudges an LLM to match the user’s voice/conventions
- avoids dumping huge examples into prompts (token bloat + noise)
- can be exported into tools people already use (Cursor / Copilot)
- preserves **correctness** (especially for code)

---

# 1) What the tool outputs

### A) `style.md` (human-readable)
A concise style guide:
- Rules (Do / Don’t)
- Conventions (naming, structure, comments/docs, tone)
- 2–6 short “golden exemplars” (high-signal, representative)

### B) `style.json` (machine-readable StyleSpec)
A structured spec the tool can apply consistently:
- detected conventions + confidence
- weights / priorities (e.g., naming > whitespace)
- exemplar pointers and metadata
- language-specific sections

### C) Exports (so it becomes “flip a switch”)
- Cursor rule files under `.cursor/rules/`
- GitHub Copilot repo instructions: `.github/copilot-instructions.md`

### D) Optional formatter configs (when applicable)
- C/C++: `.clang-format` (or guidance if one already exists)
- Python: `pyproject.toml` Black config guidance/stub

---

# 2) Core architecture (high-level pipeline)

## Step 1 — Ingest
Inputs:
- Code repos (e.g., C++, Python)
- Writing samples (md/txt/docx)
- Optional: “golden” file list (user-curated best examples)
- Optional: “anti-examples” list (things to avoid)

## Step 2 — Normalize & segment
Goal: convert heterogeneous inputs into a consistent internal representation.

**For code**
- Store file text + metadata (path, language, timestamps)
- Segment into units: functions, classes, comments, includes/imports

**For writing**
- Normalize whitespace
- Preserve headings/sections
- Segment into paragraphs/sections

## Step 3 — Extract style signals (the actual “profiling”)
This is where you distinguish *style* from *content*.

### Code style signals (beyond formatting)
Use a parser (AST) so you can extract structural patterns reliably:
- naming patterns (camelCase vs snake_case; member prefixes)
- function “shape”: typical length, nesting depth, early-return frequency
- error-handling patterns (exceptions vs status returns)
- comment/doc patterns (Docstring templates, Doxygen usage, tone)
- include/import ordering tendencies
- typical module/file organization (headers, helpers, test style)

### Prose style signals
Lightweight stylometry (enough to be useful; not “forensics-grade”):
- sentence length distribution, punctuation habits
- transition phrase patterns
- paragraph length distribution
- headings & section patterns
- vocabulary preferences + “avoid list”

> Note: stylometry features are often “structural” (sentence length, punctuation, function words), because content words can correlate with topic rather than style.

## Step 4 — Distill into a compact StyleSpec (“condensing”)
This is the key: **don’t** dump 20 files into the prompt.

Your distillation should:
- rank candidate rules by **confidence × impact**
- keep a small stable set (ex: 15–40 rules)
- choose a few representative “golden exemplars”
- generate “anti-patterns” (what to avoid)

### Exemplar selection strategy (practical)
- Build a candidate set of short snippets (10–30 lines code; 1–3 paragraphs prose)
- Embed or vectorize snippets
- Cluster (or use MMR-style diversity selection)
- Pick 1 representative snippet per cluster

Why: you want **coverage** without token bloat.

### Important warning: overprompting can backfire
Adding too many few-shot examples can degrade model performance in some cases.
So exemplars must be small, curated, and measured.

## Step 5 — Runtime “apply” mode (preserve correctness)
**Two-pass generation** is the safest approach for code:

1) **Pass A (Correctness first):**
   - “Solve the task; prioritize correctness, tests, requirements.”

2) **Pass B (Style transform):**
   - “Rewrite to match StyleSpec; do not change behavior.”

Then validate:
- run formatter
- run linter (optional)
- run tests / compile (if available)

This makes style adherence repeatable and reduces “style rewrite broke behavior.”

## Step 6 — Export to where people already work
Make your tool instantly valuable by exporting to:
- `.cursor/rules/` for Cursor
- `.github/copilot-instructions.md` for Copilot

The export content should be:
- short and direct (bulleted rules)
- include 2–6 short exemplars
- include anti-patterns (“avoid these phrases/patterns”)
- reference formatter configs when present (e.g., “follow `.clang-format`”)

## Step 7 — Scoring (trust-building feature)
Users will trust the tool more if you can measure it.

For **code**
- formatting compliance: formatter diff ~ 0
- naming compliance: regex-based checks against inferred conventions
- comment/doc compliance: presence + template matching
- complexity deltas: function length/nesting drift warnings

For **writing**
- stylometric deltas: sentence length & punctuation distribution shifts
- structure deltas: headings/paragraph patterns
- similarity score: embedding similarity (optional)

---

# 3) Implementation notes (what I’d actually build)

## Parsing: Tree-sitter (recommended)
Use Tree-sitter via Python bindings to parse multiple languages and extract structure.
- Pros: multi-language, fast, battle-tested
- Use it to find functions/classes/comments/import blocks

## Formatting anchors (don’t reinvent them)
- If `.clang-format` exists, treat it as ground truth for formatting.
- If not, infer a minimal config, then refine later.
- For Python, detect Black usage and generate guidance accordingly.

## Store StyleSpec with explicit priorities
Example fields:
- `formatting` (lowest priority if formatter is present)
- `naming` (high priority)
- `structure` (high priority)
- `comments_docs` (medium-high)
- `idioms` (medium-high)
- `prose_voice` (tone/cadence rules)

Then your “apply” step can choose what to enforce depending on the task type.

---

# 4) MVP plan (strong first version)

### MVP v0: C++ “stylepack”
1) `init`: scan a C++ repo → generate `style.md` + `style.json`
2) `export`: write Cursor rules + Copilot instructions
3) `preamble`: print a compact “style prefix” for prompts
4) `score`: compute style adherence on a generated file

Why C++ first:
- clang-format provides a powerful formatting anchor
- the remaining “style” work is conventions/idioms—where LLMs often drift

### MVP v1: Add writing samples
- ingest markdown/articles
- extract voice features
- generate a prose Style Profile + exemplars
- score “voice drift”

---

# 5) Safety / ethics guardrails (recommended)
A style mimicking tool can be used for impersonation. The intended use:
- your own samples
- your team’s repo with permission
- a brand voice you own

Suggested design:
- provenance logs (which source files contributed to which rules)
- explicit “I own/have rights to these samples” confirmation
- avoid “public figure voice packs” by default

---

# References (for future you)
Cursor rules folder behavior:
- https://cursor.com/docs/context/rules

GitHub Copilot repository instructions file:
- https://docs.github.com/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot
VS Code Copilot instructions doc:
- https://code.visualstudio.com/docs/copilot/customization/custom-instructions

clang-format docs:
- https://clang.llvm.org/docs/ClangFormat.html
- https://clang.llvm.org/docs/ClangFormatStyleOptions.html

Black configuration:
- https://black.readthedocs.io/en/stable/usage_and_configuration/the_basics.html

EditorConfig:
- https://spec.editorconfig.org/

Tree-sitter + Python bindings:
- https://tree-sitter.github.io/tree-sitter/
- https://github.com/tree-sitter/py-tree-sitter

Overprompting / too many examples can degrade performance (few-shot dilemma):
- https://arxiv.org/abs/2509.13196

Stylometry background:
- https://pmc.ncbi.nlm.nih.gov/articles/PMC11707938/


