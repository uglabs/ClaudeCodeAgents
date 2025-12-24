# Agent Origin & Dependency Table

## Custom Agents (This Repository)

| Agent | File | Added | Commit | Origin | Purpose |
|-------|------|-------|--------|--------|---------|
| Jenny | `Jenny.md` | 2025-07-31 | 2c420d9 | Custom | Implementation verification against specs |
| karen | `karen.md` | 2025-07-31 | 2c420d9 | Custom | Reality checks, project completion assessment |
| claude-md-compliance-checker | `claude-md-compliance-checker.md` | 2025-07-31 | 2c420d9 | Custom | CLAUDE.md guideline compliance |
| code-quality-pragmatist | `code-quality-pragmatist.md` | 2025-07-31 | 2c420d9 | Custom | Over-engineering detection |
| task-completion-validator | `task-completion-validator.md` | 2025-07-31 | 2c420d9 | Custom | Verify claimed completions actually work |
| ui-comprehensive-tester | `ui-comprehensive-tester.md` | 2025-07-31 | 2c420d9 | Custom | Cross-platform UI testing |
| ultrathink-debugger | `ultrathink-debugger.md` | 2025-08-03 | dffb448 | Custom | Deep debugging with extended thinking |
| smith | `smith.md` | 2025-12-24 | e5d1d8a | Custom | Feature development orchestrator (4-phase) |

## Agent Dependency Graph

### smith (orchestrator)
```
smith
├── REQUIRED
│   ├── @Jenny ............... (custom) spec validation
│   ├── @karen ............... (custom) reality checks
│   └── @python-pro .......... (built-in) implementation
│
└── OPTIONAL
    ├── @architect-reviewer .. (built-in) design decisions
    ├── @code-quality-pragmatist (custom) quality review
    ├── @test-automator ...... (built-in) write tests
    ├── @ui-comprehensive-tester (custom) UI validation
    ├── @task-completion-validator (custom) stub detection
    ├── @technical-writer .... (built-in) documentation
    ├── @debugger ............ (built-in) troubleshooting
    └── @ultrathink-debugger . (custom) deep debugging
```

### karen
```
karen
└── RECOMMENDS
    ├── @task-completion-validator (custom)
    ├── @code-quality-pragmatist .. (custom)
    └── @claude-md-compliance-checker (custom)
```

### Jenny
```
Jenny
└── RECOMMENDS
    ├── @code-quality-pragmatist .. (custom)
    ├── @claude-md-compliance-checker (custom)
    ├── @task-completion-validator (custom)
    └── @karen .................... (custom)
```

### task-completion-validator
```
task-completion-validator
└── RECOMMENDS
    ├── @code-quality-pragmatist .. (custom)
    ├── @claude-md-compliance-checker (custom)
    └── @karen .................... (custom)
```

### code-quality-pragmatist
```
code-quality-pragmatist
└── RECOMMENDS
    ├── @claude-md-compliance-checker (custom)
    ├── @task-completion-validator (custom)
    └── @karen .................... (custom)
```

### claude-md-compliance-checker
```
claude-md-compliance-checker
└── RECOMMENDS
    ├── @task-completion-validator (custom)
    └── @code-quality-pragmatist .. (custom)
```

### ui-comprehensive-tester
```
ui-comprehensive-tester
└── STANDALONE (no agent dependencies)
```

### ultrathink-debugger
```
ultrathink-debugger
└── STANDALONE (no agent dependencies)
```

## Built-in Agents Referenced

These agents exist in `~/.claude/agents/` (120 total) and are referenced by custom agents:

| Agent | Referenced By | Role |
|-------|---------------|------|
| @python-pro | smith | Primary implementation delegate |
| @architect-reviewer | smith | Architectural design decisions |
| @test-automator | smith | Test creation |
| @technical-writer | smith | Documentation updates |
| @debugger | smith | Standard debugging |

## Agent Categories

### Validation Agents
| Agent | Focus | Called When |
|-------|-------|-------------|
| Jenny | Spec compliance | After implementation, verify matches requirements |
| karen | Reality check | Verify feature actually works end-to-end |
| task-completion-validator | Stub detection | Ensure implementation isn't mocked/incomplete |
| claude-md-compliance-checker | Project rules | Verify code follows CLAUDE.md guidelines |

### Quality Agents
| Agent | Focus | Called When |
|-------|-------|-------------|
| code-quality-pragmatist | Over-engineering | Review for unnecessary complexity |
| ui-comprehensive-tester | UI validation | After UI changes |

### Development Agents
| Agent | Focus | Called When |
|-------|-------|-------------|
| smith | Orchestration | Feature development (concept → tested code) |
| ultrathink-debugger | Deep debugging | Complex bugs requiring extended analysis |

## Missing Agents (Gaps Identified)

| Proposed Agent | Purpose | Gap Identified By |
|----------------|---------|-------------------|
| @test-validator | Validate test quality (not just pass/fail) | smith review (2025-12-24) |

## Installation

Custom agents should be copied to `~/.claude/agents/`:

```bash
# Install all custom agents
cp *.md ~/.claude/agents/

# Or install specific agents
cp Jenny.md karen.md smith.md ~/.claude/agents/
```

---

*Last updated: 2025-12-24*
