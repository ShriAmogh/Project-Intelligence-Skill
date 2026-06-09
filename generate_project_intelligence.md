## Role

You are generating a machine-readable knowledge file for coding agents.
NOT human documentation. NOT a README.
Target reader: an AI coding agent that needs to navigate and modify this codebase.
Optimise for: token efficiency + navigational completeness.
Rule: if a coding agent can infer something from a one-liner, don't write a paragraph.

---

## Output contract

The file must let a coding agent answer ALL of these without opening any source file:

1. Where is the code that does X?
2. What is the exact signature of function Y, including its constructor?
3. What does every field in model W look like, with types and constraints?
4. If I call function A, what else gets called, in what order?
5. What will break if I change B?
6. What are the non-obvious things that will cause bugs?
7. What files does the system write to disk, and what is their schema?
8. What state is initialised in every constructor?

If the output cannot answer all eight, it is incomplete. Do not save it.

---

## Phase 1 — Detect project type

Read: `package.json`, `pyproject.toml`, `requirements.txt`, `Cargo.toml`,
`go.mod`, `Dockerfile`, `.env.example`, `README.md`

Detect ALL matching types:
`web-app` | `api-server` | `cli-tool` | `mobile-app` | `agentic-ai` |
`ml-pipeline` | `data-pipeline` | `library-sdk` | `desktop-app`

---

## Phase 2 — Codebase scan (complete ALL steps before writing a single line)

### Step 1 — File tree
List every file. One-line description of its job for each.
Include output/artifact files (JSON dumps, PNGs, logs) — not just source files.

### Step 2 — Entry points
Trace call chain 3 levels deep from every entry point.
For each call note: what it receives, what it returns, what it writes to disk.

### Step 3 — Every constructor
For every class, document `__init__`:
- Every parameter with type and default
- Every instance variable set (self.x = ...)
- Any side effects (files opened, clients created, network calls)
- What raises on bad input

### Step 4 — Every function and method
For every function/method (excluding __init__, covered above):
- Full signature with param types and return type
- One-line: what it does
- Side effects: mutates state / writes to disk / makes LLM calls / makes network calls
- Return behaviour: what it returns on success AND on failure (None vs raise vs error string)
- Gotcha: anything non-obvious — index conventions, silent failures, cumulative state,
  bypass strings, exact string matches required, flags that change behaviour

### Step 5 — Every data model
For every Pydantic model, dataclass, TypedDict, interface, or schema:
- Every field: name, type, constraints, and the rule if non-obvious
- Any validators or computed fields
- How it is used (input to X, output of Y)

### Step 6 — All output files
Find every file the system writes to disk at runtime (not source files):
- Path
- Format (JSON, CSV, PNG, etc.)
- Schema or structure
- When it is written (which function, under what condition)
- Whether it is cumulative/append or overwritten each run

### Step 7 — All prompts
For every function that builds or sends a prompt to an LLM:
- Which variables are injected and exactly where they come from
- The structural sections of the prompt in order
- The exact output format the model is expected to return

### Step 8 — Call graph
Trace every entry point through the full execution tree.
Show conditionals (only if FLAG=True), loops (for each X), and writes to disk.

### Step 9 — Config scan
Every env var AND every hardcoded constant:
- Name, file, default value
- What exactly fails or behaves wrongly if it is missing, wrong, or extreme

### Step 10 — Gotcha scan
Grep for: TODO / FIXME / HACK / NOTE / XXX / WORKAROUND
Also scan for all of these patterns — document every instance found:
- Constructor or class state that persists across runs (cumulative files, counters)
- Functions that return None on failure instead of raising
- Functions that fail silently (pass in except, empty return)
- Exact string literals that must match elsewhere in the code
- Index conventions that differ between layers (0-indexed vs 1-indexed)
- Disabled features (flags hardcoded to False)
- LLM-estimated or LLM-scored metrics presented as algorithmically computed
- Shared mutable state between calls
- Output files that are cumulative vs overwritten
- Any bypass logic (hardcoded strings that skip validation)

### Step 11 — Dependency scan
Every package in requirements.txt / package.json / pyproject.toml:
- Which specific files use it
- Exactly what it is used for in each file

---

## Phase 3 — Write project_intelligence.md

Rules:
- Fill every section from real code found in Phase 2
- No placeholder text. No prose paragraphs. No motivation or "why it was built"
- Every section is tables, code blocks, or terse bullet points — no narrative
- If a section is N/A: write `N/A — [one-line reason so future agents know it was checked]`
- Remove domain modules that don't match the detected project type
- §11 Feature Log initial entry must be filled with real function names, real model
  names, and real gotchas from the scan — not "all functions from scan"

---

```markdown
# Project Intelligence
> FOR CODING AGENTS ONLY. Not human documentation — that lives in README.
> Read this before touching any file. Update after every feature change.
> New context window? Read §1→§10, then start coding. Read §11 only if debugging history.

---

## META: How to maintain this file

| Event | Action |
|-------|--------|
| Feature added | Append to §11. Update §4 if functions changed. Update §3 if models changed. Update §6 if new output files added. |
| Function modified | Update its entry in §4 in-place. |
| Constructor modified | Update its __init__ block in §4 in-place. |
| File added | Add to §2 tree + §4. |
| File deleted | Remove from §2 + §4. |
| Config changed | Update §7 row in-place. |
| New output file | Add to §6. |
| Bug with non-obvious cause | Add to §8. |
| New context window | Read §1→§10, skip §11 unless debugging. |

---

## 1. Project Identity

```
name:    
type:    
lang:    
llm:     (if applicable — SDK + model names)
entry:   
deps:    
env:     
```

**System in one line**: [what it does, to what input, producing what output]

---

## 2. File Tree

```
/
├── file.py          # one-line: this file's single job
├── output.json      # RUNTIME OUTPUT: written by X, schema: {key: type}
```
Note: include runtime-generated files (JSON dumps, plots, logs, memory files).

---

## 3. Data Models

### `ModelName` (`path/to/file`)
```python
field_name: type              # constraint or rule — empty only if X
field_name: Literal["A","B"]  # what each value means
field_name: List[OtherModel]  # relationship to other model
```
used as: input to X() / output of Y() / persisted to path/file

[one block per model — every field, every constraint]

---

## 4. Function Map

### `path/to/file.py` — [one line: file's single responsibility]

```python
ClassName.__init__(param: type = default, ...) -> None
  # sets: self.var (type) — what it holds
  # sets: self.var (type) — what it holds
  # side effects: opens file / creates client / loads from disk
  # raises: ErrorType if condition

ClassName.method(param: type, ...) -> return_type
  # what it does
  # side effects: mutates self.x / writes path/file / makes LLM call
  # returns: X on success | None on failure (does not raise)
  # gotcha: anything non-obvious

module_function(param: type, ...) -> return_type
  # what it does
  # side effects: none | writes path/file | mutates global X
  # returns: X on success | raises ErrorType on failure
  # gotcha: anything non-obvious
```

[one block per file — every class, every __init__, every method, every function]

---

## 5. Output Files

| File | Format | Written by | When | Cumulative? |
|------|--------|-----------|------|-------------|
| path/file.json | JSON | function() | on every run / on condition | yes — appends / no — overwrites |

Schema for each non-trivial output file:
```
path/file.json:
{
  "key": type   # what this holds
}
```

---

## 6. Prompt Map

### `function_name()` in `path/to/file`
```
injected:  var_name   → source: where exactly this value comes from
           var_name   → source: where exactly this value comes from
structure: [SECTION LABEL] → [SECTION LABEL] → [SECTION LABEL]
model:     MODEL_CONSTANT (config.py)
output:    exact JSON shape or plain text format the model must return
```

[one block per prompt-building function — agentic-ai and ml-pipeline projects must fill this]

---

## 7. Call Graph

```
entry_point.py
└── ClassA.__init__()              ← initialises: client, state vars
└── ClassA.method()
    ├── function_x()               ← path/file.py
    ├── function_y()               ← path/file.py (only if FLAG=True)
    ├── [loop] ClassB.method()     ← path/file.py — for each item in X
    │   └── writes: path/output.json (overwrites)
    └── returns: Type | None
```

[cover every entry point — show conditionals, loops, disk writes, and return types]

---

## 8. Config & Environment

| Key | Required | Default | Breaks if wrong |
|-----|----------|---------|-----------------|
| ENV_VAR | yes | — | exact error or behaviour |
| CONSTANT (`file.py`) | — | value | exact error or behaviour |

[every env var AND every hardcoded constant — individual rows, no grouping]

---

## 9. Gotchas

- **[file.py:line]** — what the gotcha is. what it causes. what a developer must know.

[minimum 6 — all specific to this codebase — no generic advice]
[must include: any cumulative state, any silent failures, any exact-string dependencies,
 any disabled flags, any index convention mismatches, any LLM-scored metrics]

---

## 10. Dependencies

| Package | Used in | For |
|---------|---------|-----|
| name | specific file(s) | exactly what — not just "LLM calls" |

---

## 11. Architecture Decisions

| Decision | Chosen | Rejected | Why |
|----------|--------|----------|-----|

---

## [DOMAIN MODULES — include matching ones, delete the rest]

### Module: Web App
```
routing:     file-based | config-based | key file
rendering:   SSR | SSG | CSR | ISR
auth:        method | key file
state:       tool | key file
components:  path/
bundler:     tool
```

### Module: API Server
```
protocol:    REST | GraphQL | gRPC | tRPC
middleware:  [auth → rate_limit → validation → handler] with file per layer
validation:  tool | key file
errors:      response format | key file
pagination:  cursor | offset | none
```

### Module: Agentic AI
```
topology:    single | multi-agent | hierarchical | event-driven
agent files: path/file → ClassName (class-based | function-based)
tools:       path/file → registration: SDK native | manual router
             tool names: name → function → what it calls
llm:         SDK | model constant names | streaming: yes/no
prompts:     path/file | format: f-string | jinja2 | yaml
memory:      type: in-memory | file | DB | vector
             key file | what persists vs ephemeral | cumulative: yes/no
loop:        1. step → 2. step → 3. step (numbered, exact)
```

### Module: ML Pipeline
```
stages:    stage → file → input → output (table)
model:     architecture | framework | checkpoint path
train:     entry file
infer:     entry file
data:      path | format
tracking:  tool | key file
hardware:  CPU-only | GPU (CUDA version)
```

### Module: Data Pipeline
```
orchestrator:  tool | none
dags:          path
sources:       list with types
destinations:  list with types
transform:     tool | key file
schedule:      cron expression | trigger type
on_failure:    retry | alert | dead-letter | silent
```

### Module: CLI Tool
```
commands:     command → file → purpose (table)
arg_parser:   tool
config_file:  yes/no | format | path
output:       plain | JSON | table | rich
distribution: npm | pypi | homebrew | binary
```

### Module: Mobile App
```
platform:       iOS | Android | cross-platform
navigation:     library | key file
state:          library | key file
native_modules: list
api:            REST | GraphQL | key file
push:           provider | key file
build:          process steps
```

### Module: Library / SDK
```
public_api:  key file listing all exports
versioning:  semver | calver | strategy
targets:     ESM | CJS | UMD | wheel
peer_deps:   list with version constraints
docs:        tool | path
publish:     pipeline steps
```

---

## 12. Feature Log

> Append after every change. Never edit past entries.
> Format: `### [YYYY-MM-DD] title`

<!--
TEMPLATE — copy for each update:

### [YYYY-MM-DD] title

files_changed:        path/file, path/file
functions_added:      ClassName.method(param: type) -> return_type — one line what it does
functions_modified:   ClassName.method — what specifically changed
constructors_changed: ClassName.__init__ — what state var was added/removed
models_changed:       ModelName — field added/removed/type changed
output_files_changed: path/file — schema change or new file
config_changed:       CONSTANT — old_value → new_value
gotcha:               specific non-obvious thing (add to §9 too if permanent)

-->

### [YYYY-MM-DD] Initial scaffold

files_changed:     [list every file created]
functions_added:   [list every function with real signature — not "all functions"]
models_added:      [list every model with real field count — not "all models"]
output_files:      [list runtime output files and their schemas]
gotcha:            [real gotchas from Phase 2 Step 10 — minimum 3]
```

---

## Phase 4 — Self-check

Do not save until ALL of the following are true:

**Coverage**
- [ ] Every file in the repo (including output/runtime files) has an entry in §2
- [ ] Every class has its `__init__` documented in §4 with all self.vars listed
- [ ] Every function/method has signature + return type + failure behaviour in §4
- [ ] Every model has every field with type and constraint in §3
- [ ] Every output file (JSON, PNG, log) has an entry in §5 with schema

**Correctness**
- [ ] §6 call graph covers every entry point with conditionals and disk writes shown
- [ ] §7 has every env var AND every hardcoded constant as individual rows
- [ ] §6 prompt map filled for every prompt-building function (agentic-ai / ml projects)
- [ ] Only domain modules matching detected project type are included

**Gotchas**
- [ ] §9 has minimum 6 gotchas, all codebase-specific
- [ ] §9 explicitly covers: cumulative state, silent failures, exact-string dependencies,
      disabled flags, index mismatches, LLM-scored metrics (wherever they exist)

**Format**
- [ ] Zero prose paragraphs — every section is tables, code blocks, or bullet points
- [ ] Zero placeholder text — no "list from scan", no "TBD", no template examples

**Feature Log**
- [ ] §12 initial entry has real function signatures (not "all functions from scan")
- [ ] §12 initial entry has real model names with field counts
- [ ] §12 initial entry has real gotchas from the scan