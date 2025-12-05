# Repository Setup Best Practices

> One command to rule them all: `just setup`

## The Problem

Many setup steps need to happen **once per clone** but are easy to forget:

- Git hooks configuration (`git config core.hooksPath`)
- Dependency installation (`npm install`, `pip install`)
- Tool setup (Playwright browsers, pre-commit hooks)
- Environment configuration

When these are documented but not automated, developers (and agents) forget them, leading to:

- Missing git hooks (no linting, no beads sync)
- Broken dev environment
- "Works on my machine" issues

## The Solution

Create a `just setup` command that handles all per-clone initialization:

```just
# One-time setup after clone (run this first!)
setup:
    #!/usr/bin/env bash
    echo "Setting up development environment..."

    # Configure git hooks
    git config core.hooksPath .githooks
    echo "✓ Git hooks configured"

    # Install dependencies (pick your stack)
    npm install          # JavaScript/TypeScript
    # uv sync            # Python with uv
    # pip install -e .   # Python traditional

    # Tool-specific setup
    npx playwright install  # If using Playwright
    # pre-commit install    # If using pre-commit

    echo ""
    echo "✅ Setup complete! Run 'just dev' to start."
```

## What to Include in Setup

### Always

- **Git hooks** - `git config core.hooksPath .githooks`
- **Dependencies** - Package manager install

### Project-Specific

| If Using   | Add to Setup                    |
| ---------- | ------------------------------- |
| Playwright | `npx playwright install`        |
| pre-commit | `pre-commit install`            |
| Beads      | Hooks handle sync automatically |
| Docker     | `docker compose pull`           |
| Database   | Migration/seed scripts          |

## CLAUDE.md Integration

Reference the setup command prominently:

```markdown
## First-Time Setup

After cloning, run once:

\`\`\`bash
just setup
\`\`\`

This configures git hooks, installs dependencies, and sets up tools.
```

## Multi-Agent Considerations

When multiple agents clone the same repo:

1. Each clone needs `just setup` run independently
2. Git hooks are per-clone (not shared via git)
3. Setup is idempotent - safe to run multiple times

## Example: Full-Stack TypeScript Project

```just
setup:
    #!/usr/bin/env bash
    echo "Setting up development environment..."

    # Git hooks (beads sync, linting, push protection)
    git config core.hooksPath .githooks
    echo "✓ Git hooks configured"

    # Dependencies
    npm install
    echo "✓ npm dependencies installed"

    # Playwright for E2E tests
    npx playwright install
    echo "✓ Playwright browsers installed"

    # Environment file
    if [ ! -f .env ]; then
        cp .env.example .env
        echo "✓ Created .env from template"
    fi

    echo ""
    echo "✅ Setup complete!"
    echo "   Run 'just dev' to start the dev server"
    echo "   Run 'just test' to run tests"
```

## Example: Python Project

```just
setup:
    #!/usr/bin/env bash
    echo "Setting up development environment..."

    # Git hooks
    git config core.hooksPath .githooks
    echo "✓ Git hooks configured"

    # Dependencies with uv
    uv sync
    echo "✓ Python dependencies installed"

    # Pre-commit hooks
    uv run pre-commit install
    echo "✓ Pre-commit hooks installed"

    echo ""
    echo "✅ Setup complete!"
    echo "   Run 'just dev' to start"
    echo "   Run 'just test' to run tests"
```

## Verification

Add a `check-setup` command to verify everything is configured:

```just
# Verify setup is complete
check-setup:
    #!/usr/bin/env bash
    ERRORS=0

    # Check git hooks
    if [ "$(git config core.hooksPath)" != ".githooks" ]; then
        echo "❌ Git hooks not configured"
        ERRORS=$((ERRORS + 1))
    else
        echo "✓ Git hooks configured"
    fi

    # Check dependencies
    if [ ! -d "node_modules" ]; then
        echo "❌ npm dependencies not installed"
        ERRORS=$((ERRORS + 1))
    else
        echo "✓ npm dependencies installed"
    fi

    if [ $ERRORS -gt 0 ]; then
        echo ""
        echo "Run 'just setup' to fix"
        exit 1
    else
        echo ""
        echo "✅ All checks passed"
    fi
```
