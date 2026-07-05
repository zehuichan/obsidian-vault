---
url: "https://ponytail.dev/"
clipped: 2026-07-05
title: "ponytail — the lazy senior dev for your AI agent"
---

# ponytail

He says nothing. He writes one line. It works.

A ruleset that makes your AI coding agent write the least code that works — like a senior dev who's been paged at 3am one too many times.

- GitHub: DietrichGebert/ponytail
- License: MIT

## Your agent reaches for fifty lines. The job needs one.

```diff
- class CacheManager:
-     def __init__(self, ttl, maxsize):
-         self._store, self._lock = {}, Lock()
-     # ...44 more lines you maintain forever
+ @lru_cache(maxsize=1000)
+ def fetch(...): ...
```

Same behavior. 48 fewer lines. Zero bugs in code that no longer exists.

## Stop at the first rung that holds

1. **Does this need to exist?** Speculative need = skip it. (YAGNI)
2. **Already in this codebase?** Reuse the helper, util, or pattern that already lives here.
3. **Does the standard library do it?** Use it.
4. **Native platform feature covers it?** `<input type="date">` over a picker lib.
5. **Already-installed dependency solves it?** Use it. Don't add a new one.
6. **Can it be one line?** One line.
7. **Only then:** the minimum code that works.

## Less code. Fewer tokens. Same safety.

| Metric | Result |
|--------|--------|
| Less code | 54% |
| Fewer tokens | 22% |
| Lower cost | 20% |
| Faster | 27% |
| Safety kept | 100% |

Medians across 12 feature tasks on a FastAPI + React repo. Validation, error handling, security and accessibility are never simplified away.

## Install

### Claude Code

```
/plugin marketplace add DietrichGebert/ponytail
/plugin install ponytail@ponytail
```

### Codex

```
codex plugin marketplace add DietrichGebert/ponytail
```

### Copilot CLI

```
copilot plugin install ponytail@ponytail
```

### Gemini CLI

```
gemini extensions install github.com/DietrichGebert/ponytail
```

### Pi harness

```
pi install git:github.com/DietrichGebert/ponytail
```

Also supports OpenCode, Cursor, Windsurf, Cline, Kiro, Zed and more — 14+ agents. Full list in the README.

## Drive it from chat

| Command | Description |
|---------|-------------|
| `/ponytail lite\|full\|ultra\|off` | Set intensity, or turn it off |
| `/ponytail-review` | Find over-engineering in the current diff |
| `/ponytail-audit` | Scan the whole repo for bloat |
| `/ponytail-debt` | Collect deferred shortcuts into a ledger |
| `/ponytail-gain` | Show the benchmark scoreboard |
| `/ponytail-help` | Quick command reference |

## Pick how lazy

### lite (default)

Builds what you asked, names the lazier alternative in one line. You pick.

### full

The ladder, enforced. Stdlib and native first. Shortest diff, shortest explanation.

### ultra

YAGNI extremist. Ships the one-liner and challenges the rest of the requirement in the same breath.

> The best code is the code never written.
