# 🧬 Morris

### AI-Powered Mutation Testing for Rust

*Find the bugs hiding in your test suite*

[![Rust](https://img.shields.io/badge/rust-1.70%2B-orange.svg)](https://www.rust-lang.org/)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-skill-8A2BE2.svg)](https://claude.com/claude-code)

```
    ╔═══════════════════════════════════════╗
    ║  Your Code  →  Morris  →  Better Tests ║
    ╚═══════════════════════════════════════╝
```

---

## Changes from the original

This is a fork of [Marc Brooker's morris](https://github.com/mbrooker/morris). Key differences:

- **Claude Code skill instead of Bedrock** — the original used AWS Bedrock for AI reasoning; this fork uses a Claude Code `/morris` skill, so no API keys or AWS setup needed
- **Markdown + YAML output** — `cargo morris discover` outputs markdown with YAML frontmatter and fenced code blocks instead of a single-line JSON blob with escaped newlines
- **Diff-style mutation input** — mutation plans use a `## file:line` / `- original` / `+ mutated` format instead of JSON, which is less error-prone for LLMs to generate
- **File/directory targeting** — you can run morris on specific files or directories (`/morris src/lib.rs`)
- **Dropped serde/serde_json dependencies** — the simpler text formats eliminated the need for JSON serialization

---

## 🎯 What is Morris?

Morris is a [Claude Code](https://claude.com/claude-code) skill that performs intelligent mutation testing on Rust projects. Instead of exhaustively testing thousands of mutations, Morris uses AI to strategically select 5-8 high-value mutations that are most likely to reveal gaps in your test coverage.

Morris splits work between **deterministic code** (file discovery, test execution, mutation application) and **AI reasoning** (selecting mutations, analyzing results). The deterministic parts run as a cargo subcommand. The AI parts are handled by Claude Code — no API key required if you have a Claude subscription.

```
┌─────────────┐      ┌──────────────┐      ┌─────────────┐
│  Your Code  │ ───> │    Morris    │ ───> │  Test Gaps  │
│   + Tests   │      │ (Fixed Flow) │      │  + Fixes    │
└─────────────┘      └──────────────┘      └─────────────┘
                            │
                            ├─ Discovers files (deterministic)
                            ├─ Runs baseline tests (deterministic)
                            ├─ AI selects mutations (Claude Code)
                            ├─ Tests mutations (deterministic)
                            └─ AI analyzes results (Claude Code)
```

---

## 🚀 Quick Start

### Prerequisites

- [Claude Code](https://claude.com/claude-code) installed
- A Rust project with tests

### Installation

```bash
# Install the cargo subcommand
cargo install --git https://github.com/JulienEllie/morris

# Install the Claude Code skill (global — works in any project)
mkdir -p ~/.claude/skills
cp -r .claude/skills/morris ~/.claude/skills/morris
```

### Usage

In any Rust project, open Claude Code and type:

```
/morris
```

That's it! Claude will discover your source files, propose mutations, test them, and analyze the results.

You can also target specific files or directories:

```
/morris src/lib.rs
/morris src/parser/
```

---

## 📋 How It Works

Morris uses a fixed, deterministic workflow. Claude Code provides the AI reasoning, while the `cargo-morris` binary handles all file I/O, test execution, and mutation application.

```
┌─────────────────────────────────────────────────────────────┐
│                     Morris Workflow                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. 📁 Discovery (deterministic)                            │
│     └─ Recursively finds .rs files under src/              │
│                                                             │
│  2. 📖 Read Sources (deterministic)                         │
│     └─ Reads all source files into memory                  │
│                                                             │
│  3. ⏱️  Baseline (deterministic)                             │
│     └─ Runs `cargo test` to verify and measure timing      │
│                                                             │
│  4. 🧬 Mutation Plan (AI — Claude Code)                     │
│     └─ Claude proposes 5-8 strategic mutations             │
│        • Operators: > → <, + → -, == → !=                  │
│        • Boundaries: 0 → 1, len() → len()-1                │
│        • Logic: && → ||, true → false                      │
│                                                             │
│  5. 🧪 Testing Loop (deterministic)                         │
│     For each mutation:                                      │
│     ├─ Backup original file                                │
│     ├─ Apply mutation to single line                       │
│     ├─ Run tests (with 3x baseline timeout)                │
│     └─ Restore original file                               │
│                                                             │
│  6. 📊 Results Summary (deterministic)                      │
│     └─ Counts killed / survived / build errors             │
│                                                             │
│  7. 💡 Analysis (AI — Claude Code)                          │
│     └─ Claude explains surviving mutations and             │
│        suggests specific tests to catch them               │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎛️ CLI Subcommands

Morris exposes three subcommands used by the `/morris` skill. You don't normally call these directly, but they're available if you want to integrate Morris into other workflows.

| Command | Description |
|---------|-------------|
| `cargo morris discover [paths...]` | Find source files, read them, run baseline tests. Outputs markdown. |
| `cargo morris test --timeout <secs>` | Read mutation plan from stdin, test each mutation, output results. |
| `cargo morris apply --timeout <secs>` | Read analysis text from stdin, auto-apply suggested tests. |

All subcommands accept `-v` / `--verbose` for debug logging.

---

## 📊 Example Session

```
> /morris

🧬 Morris v0.3.0 - Mutation Testing

📁 Discovering source files...
   src/lib.rs

📖 Reading source files...
⏱️  Running baseline tests...
   ✅ Baseline passed in 1.2s (mutation timeout: 30.0s)

[Claude proposes 6 mutations...]

🧪 Testing 6 mutations...

   [1/1] src/lib.rs:42 - Change > to <... ❌ SURVIVED
   [2/2] src/lib.rs:67 - Change + to -... ❌ SURVIVED
   [3/3] src/lib.rs:89 - Change == to !=... ✅ KILLED
   [4/4] src/lib.rs:23 - Change >= to >... ✅ KILLED
   [5/5] src/lib.rs:51 - Change true to false... ❌ SURVIVED
   [6/6] src/lib.rs:15 - Remove bounds check... 🔧 BUILD ERROR

📊 Results: 2 killed, 3 survived out of 5 testable mutations

[Claude analyzes surviving mutations and suggests specific tests...]
```

---

## 🆚 Morris vs cargo-mutants

[cargo-mutants](https://mutants.rs/) is an excellent exhaustive mutation testing tool. Morris takes a different approach:

**cargo-mutants — Exhaustive Approach**
- Systematically generates all possible mutations
- Tests hundreds/thousands of mutations
- AST-based pattern matching
- Comprehensive coverage analysis
- Best for: CI/CD pipelines, audits

**Morris — AI-Guided Approach**
- Fixed workflow, AI used only for selection & analysis
- Selects 5-8 strategic mutations
- Contextual explanations of why mutations survive
- Best for: Interactive development, learning

The biggest difference is that mutants is a lot more mature, and probably more useful in production code bases for now.

---

## 🔧 Configuration

### Verbose Output

```bash
# Pass -v to any subcommand for debug logging
cargo morris discover -v
cargo morris test -v --timeout 30
```

---

## 🏗️ Architecture

Morris separates deterministic operations from AI reasoning. The `cargo-morris` binary handles all file I/O, test execution, and mutation application. Claude Code provides the intelligence — no API keys or separate billing required.

```
┌──────────────────────────────────────────────────────────┐
│                     Morris Architecture                  │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌────────────┐                                         │
│  │ Claude Code│  /morris                                │
│  │   Skill    │  (AI reasoning)                         │
│  └─────┬──────┘                                         │
│        │                                                 │
│        v                                                 │
│  ┌────────────────────────────────────────┐             │
│  │      cargo-morris (deterministic)      │             │
│  │                                        │             │
│  │  discover: find files, run baseline   │             │
│  │  test: apply mutations, run tests     │             │
│  │  apply: inject new tests, verify      │             │
│  └────────────────────────────────────────┘             │
│                                                          │
└──────────────────────────────────────────────────────────┘
```
