---
name: morris
description: Run AI-powered mutation testing on the current Rust project. Discovers source files, proposes strategic mutations, tests them, and analyzes surviving mutations to suggest test improvements.
---

# Morris - AI-Powered Mutation Testing

Run mutation testing on this Rust project using the `cargo-morris` binary for deterministic operations and your own reasoning for mutation planning and analysis.

## Workflow

### Step 1: Discover and baseline

Run the discover command to find source files and establish baseline timing:

```
cargo morris discover $ARGUMENTS
```

This outputs markdown to stdout with:
- YAML frontmatter containing `source_files`, `baseline_duration_secs`, and `test_timeout_secs`
- Fenced code blocks with line-numbered source code for each file

Save the `test_timeout_secs` value — you will need it later.

### Step 2: Propose mutations

Analyze the source code from the discover output. Propose 5-8 strategic single-line mutations that are likely to **survive** the existing test suite (i.e., reveal test coverage gaps).

Focus on:
- Boundary conditions (>, <, >=, <=)
- Arithmetic operators (+, -, *, /)
- Logic operators (&&, ||, !, ==, !=)
- Off-by-one errors
- Return value changes

Output your plan in diff-style format — one block per mutation:

```
## file:line description
- original line
+ mutated line
```

Example:
```
## src/lib.rs:42 Change > to >= to test boundary
-     if x > 0 {
+     if x >= 0 {

## src/lib.rs:67 Change + to -
-     x + 1
+     x - 1
```

Rules:
- `## ` header: file path (relative to project root), colon, line number, space, then description
- `- ` line: the original code copied EXACTLY as it appears in the source (including indentation)
- `+ ` line: the mutated replacement with the same indentation
- Line numbers are shown as `  N| code` in the discover output — use the number before the `|`
- Each mutation must be a single line change that still compiles
- Do NOT mutate test code — only mutate production code
- Blank lines between blocks are ignored

### Step 3: Test mutations

Pipe the mutation plan to the test command:

```
echo '<your mutation plan>' | cargo morris test --timeout <test_timeout_secs>
```

This will apply each mutation, run `cargo test`, restore the file, and output a results summary.

### Step 4: Analyze results

Read the results summary. For each **SURVIVED** mutation, explain:
1. Why the current tests don't catch it
2. A specific test that would catch it (show the code)

Be concise and actionable.

If there are no surviving mutations, congratulate the user — their tests are solid.
