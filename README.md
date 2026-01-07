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
cli.py
ingest/
signals/
distill/
export/
score/
examples/
tests/
README.md





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
