# DepScope Hallucinations Dataset

> **Public corpus of verified LLM-generated package-name hallucinations** observed in production AI coding agent traffic across 19 package ecosystems.

[![License: CC-BY-NC-SA 4.0](https://img.shields.io/badge/License-CC--BY--NC--SA_4.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)
[![Live API](https://img.shields.io/badge/Live_API-depscope.dev-blue)](https://depscope.dev/api/benchmark/hallucinations)
[![Updated daily](https://img.shields.io/badge/Updated-daily-green)](https://depscope.dev/benchmark)

## What this is

When AI coding agents (Claude, GPT, Cursor, Aider, Copilot) generate code, they sometimes invent package names that don't exist. Attackers exploit this by pre-registering plausible-sounding fake names — a class of attack called **slopsquatting**.

This repository tracks **verified** hallucinated package names observed in production [DepScope](https://depscope.dev) traffic. As of May 2026 the corpus contains **161 entries** across **18 ecosystems**: 133 from real agent traffic + 28 from research literature, with new entries added daily after a re-verification step.

## Context: DepScope at scale

DepScope is the infrastructure layer this dataset is built on:

- **8.5M+ packages** indexed across 19 ecosystems
- **45K+ vulnerabilities** tracked (OSV mirror, daily refresh)
- **330,000+ EPSS-enriched advisories**
- **1,587 KEV entries** (CISA actively-exploited list)
- **22 MCP tools** exposed at `mcp.depscope.dev/mcp`
- **Free, no auth, no rate limit**

The hallucination corpus is one of several open datasets we publish. The full intelligence layer is at [depscope.dev](https://depscope.dev).

## Methodology — and why our number isn't bigger

When a coding agent calls `/api/check` and we return 404, that's a **candidate** hallucination, not a confirmed one. Our raw 404 stream contained 1,446 candidates over 30 days — but **97.6% of those were real packages** that 404'd transiently due to upstream registry rate limits, image-URL crawls, or cache races.

We run a daily re-verifier (`scripts/reverify_halluc.py`) that re-checks every flagged entry. If it now resolves, the flag is flipped and the entry is removed from this corpus. **What you see here is what survived re-verification.**

If you publish a hallucination dataset without telling readers what you filtered out, the data is probably 95% noise. We're explicit.

## Why it matters

- **Cross-validated**: many entries are invented by 5–7 different AI agent fingerprints independently → structurally plausible to neural networks → predictable for attackers
- **Multi-ecosystem**: covers npm, PyPI, Cargo, Go, Maven, NuGet, RubyGems, Composer, Pub, Hex, Swift, CocoaPods, CPAN, Hackage, CRAN, Conda, Homebrew, Julia
- **Production-grade**: harvested from real AI agent traffic (anthropic-bot, openai-bot, amazon-bot, applebot, facebookbot, googlebot), not synthetic
- **Reproducible**: stable JSON schema + daily snapshots in `snapshots/`

## Quick start

### Live API
```bash
curl https://depscope.dev/api/benchmark/hallucinations
```

### Daily snapshots
```bash
git clone https://github.com/cuttalo/depscope-hallucinations-dataset.git
cd depscope-hallucinations-dataset
ls snapshots/  # 2026-05-05.json, 2026-05-04.json, …
```

### Schema
```json
{
  "ecosystem": "conda",
  "package_name": "torch-lightning-easy",
  "source": "observed",
  "evidence": "Seen 12 times across 7 agents — top slopsquat",
  "first_seen_at": "2026-04-23T14:52:11Z",
  "hit_count": 25,
  "likely_real_alternative": "pytorch-lightning"
}
```

| Field | Meaning |
|---|---|
| `ecosystem` | One of 19 supported registries |
| `package_name` | The exact name the agent hallucinated |
| `source` | `observed` (live agent traffic) / `research` (literature) |
| `evidence` | Short prose describing the entry |
| `first_seen_at` | ISO 8601 timestamp of first observation |
| `hit_count` | Total times we observed this name (only counted before reverification flips real → false) |
| `likely_real_alternative` | The actual package the agent likely meant |

## Top slopsquat patterns (May 2026)

| Suffix | Examples |
|---|---|
| `-easy` | `torch-lightning-easy`, `jwt-token-validator-easy` |
| `-pro` | `typescript-utility-pack-pro`, `axum-middleware-pro`, `laravel/auth-pro` |
| `-turbo` | `fastapi-turbo` |
| `-plus` | `numpy-extensions-plus` |
| `-extras` | `tokio-stream-extras`, `symfony/components-extra` |
| `-essential` | `react-hooks-essential` |
| typo | `reqeusts` (→ `requests`), `lodsh` (→ `lodash`) |

## Citation

```bibtex
@misc{depscope_hallucinations_2026,
  title   = {DepScope Hallucinations Dataset},
  author  = {Rubino, Vincenzo},
  year    = {2026},
  url     = {https://depscope.dev/api/benchmark/hallucinations},
  github  = {https://github.com/cuttalo/depscope-hallucinations-dataset},
  license = {CC-BY-NC-SA-4.0},
  note    = {Public corpus of verified LLM-generated package-name hallucinations from real AI coding agent traffic across 19 ecosystems. Re-verified daily. Built on DepScope: 8.5M+ packages, 250K+ vulnerabilities indexed.}
}
```

## License

[Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/)

You may:
- Use this dataset for **research, education, security analysis**
- Adapt and redistribute under same license
- Cite in academic papers, blog posts, security reports

You may not:
- Use for **commercial purposes** without permission ([licensing@depscope.dev](mailto:licensing@depscope.dev))

## Build your own protection

If you write or maintain an AI coding tool, integrate the [DepScope MCP server](https://github.com/cuttalo/depscope-mcp) — every blocked hallucination is one less compromised developer machine.

```json
{
  "mcpServers": {
    "depscope": { "url": "https://mcp.depscope.dev/mcp" }
  }
}
```

22 tools available: `check_package`, `package_exists`, `find_alternatives`, `check_typosquat`, `check_malicious`, `scan_project`, +16 more.

## Related

- [DepScope main site](https://depscope.dev) — 8.5M+ packages indexed
- [LLM Hallucination Benchmark (10 LLMs × slopsquat)](https://depscope.dev/benchmark) — measured baseline 87% install rate
- [Integration guide](https://depscope.dev/integrate)

## Contributing

This is a **read-only mirror of production data**. To contribute:
- Report dataset errors via [GitHub Issues](https://github.com/cuttalo/depscope-hallucinations-dataset/issues) — every false positive caught makes the dataset more useful
- Propose research collaborations: [research@depscope.dev](mailto:research@depscope.dev)
- Cite us in your papers — drop a link in Issues so we can list your work

---

**Maintained by [DepScope](https://depscope.dev) · Snapshots updated daily at 05:00 UTC**
