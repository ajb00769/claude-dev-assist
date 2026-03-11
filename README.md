# claude-dev-assist

Shared Claude Code agents and skills designed to be symlinked across any project, regardless of tech stack.

## Setup

Symlink into your project's `.claude/` directory:

```bash
# From your project root
ln -s /path/to/claude-dev-assist/agents .claude/agents
ln -s /path/to/claude-dev-assist/skills .claude/skills
```

Or symlink individual files as needed. See `CLAUDE.md` for full setup details.

## Agents

| Agent | Model | Purpose |
|-------|-------|---------|
| **100x-software-engineer** | opus | Elite generalist — system thinking, deep debugging, architecture decisions, proactive problem detection, cross-codebase mastery. The go-to for complex or ambiguous tasks. |
| **test-engineer** | opus | Test specialist — writes comprehensive tests in any language/framework. Auto-detects the test runner and follows existing project patterns. |
| **safe-refactorer** | sonnet | Refactoring with safety guarantees — writes tests before refactoring if coverage is missing, runs tests after each change. Handles structural cleanup, performance optimization, and code simplification. |
| **cybersec-engineer** | opus | Security specialist — application security, cloud security, threat modeling, OWASP Top 10 audits, vulnerability analysis. |
| **azure-devops-engineer** | opus | Azure-specific — Azure CLI, pipelines, infrastructure as code (Bicep/Terraform), DevOps workflows. Always verifies against latest docs. |

## Skills

| Skill | Trigger | What It Does |
|-------|---------|-------------|
| `/pr` | Creating pull requests | Checks branch state, analyzes commits, creates PR with consistent formatting and issue linking. |
| `/code-review` | Reviewing code or PRs | Systematic security/performance/correctness/maintainability checklist. Can post inline GitHub PR comments. |
| `/security` | Security audits | OWASP Top 10 code audit + cloud infrastructure security checks. Outputs inline findings or full report. |
| `/refactoring` | Restructuring code | Pre-refactoring checklist, safe patterns (extract, rename, move, signature change), backward compatibility rules. |
| `/debug` | Troubleshooting bugs | Structured debugging workflow: reproduce, gather context, hypothesize, isolate, fix, verify, prevent. |
| `/migrate` | Database schema changes | Auto-detects migration tool, safety checklist for destructive ops, backward-compatible migration patterns. |
| `/structure` | Project structure audit | Enforces separation of concerns, modularity, reusability. Detects opinionated frameworks and follows their conventions; applies layered architecture to unopinionated stacks. |
| `/backend` | Scaffolding backend code | Auto-detects tech stack, provides idiomatic endpoint/model/service/schema patterns following existing conventions. |
| `/github-actions` | CI/CD workflows | Workflow creation, triggers, secrets, environments, debugging failed runs. Includes example templates. |
| `/skill-creator` | Creating new skills | Meta-skill for building new skills — progressive disclosure patterns, bundled resource organization, packaging. |

## Design Principles

**Stack agnostic**: All agents and skills auto-detect the project's language, framework, and tooling rather than assuming a specific stack. Examples use neutral pseudocode or note multiple language alternatives.

**Progressive disclosure**: Skill metadata is always in context (~100 words). The full skill body loads only when triggered. Reference files load only when needed by the skill.

**Safety first**: The safe-refactorer writes characterization tests before changing code. The security skill checks OWASP Top 10. The migrate skill has destructive operation guardrails.

**Convention over configuration**: Agents read existing project patterns (test files, route handlers, migration files) before scaffolding new code, ensuring consistency.
