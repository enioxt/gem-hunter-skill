> **MIGRAÇÃO EGOS — 2026-10-01**
> Este repositório não é mais uma unidade canônica do EGOS.
> Nenhum roadmap, status, porta de entrada ou intenção futura vive aqui.
> O conteúdo válido está sendo absorvido por:
> - público compartilhável: [github.com/enioxt/cinco](https://github.com/enioxt/cinco) (site: [cinco.ia.br](https://cinco.ia.br))
> - o núcleo do EGOS é privado e não faz parte deste repositório.
> Até a migração terminar, este repositório é somente fonte histórica.

# @egosbr/gem-hunter-skill

**Opportunity discovery engine for web3 + emerging tech** — Hybrid search, multi-source aggregation, and signal ranking for real-time trend detection.

> "Find the signals before they become news."

---

## Why this exists

Discovering emerging opportunities in fast-moving domains (web3, AI, tools, startups) requires:

1. **Multi-source aggregation** — Twitter, GitHub, Hacker News, product launches, price signals
2. **Real-time filtering** — Noise is high; signal-to-noise ratio matters
3. **Contextual ranking** — What matters depends on domain (new token price action vs GitHub velocity)
4. **Async scalability** — Hunt runs background; results update incrementally

`@egosbr/gem-hunter-skill` abstracts these patterns into a reusable discovery engine.

---

## What it does

| Signal Source | Detection Method | Latency |
|---------------|------------------|---------|
| **New tokens** | First 48h on-chain activity | 1-5min |
| **Price movers** | Volume spike + % change | Real-time |
| **Social mentions** | Twitter + Discord trending | 5-15min |
| **Code velocity** | GitHub stars, commits, releases | Hourly |
| **Hacker News** | Front page + comments | 5-30min |
| **Product launches** | ProductHunt, public announcements | 10-60min |

---

## Install

```bash
npm install @egosbr/gem-hunter-skill
```

Or from monorepo:
```bash
bun add @egosbr/gem-hunter-skill
```

---

## Quick start

### Programmatic (Node.js)

```ts
import { GemHunter, type HuntOptions } from '@egosbr/gem-hunter-skill';

const hunter = new GemHunter({
  apiUrl: 'https://api.egos.ia.br/v1/gem-hunter',
  apiKey: process.env.GEM_HUNTER_API_KEY,
});

// Trigger async hunt
const job = await hunter.hunt({
  track: 'ai-tools',    // category filter
  quick: true,          // return top 5 only
});

console.log(`Hunt started: ${job.jobId}`);

// Wait for results
const finalJob = await hunter.waitForJob(job.jobId);
console.log(`Status: ${finalJob.status}`);

// Get findings
const findings = await hunter.findings();
console.log(`Found ${findings.latest.totalGems} gems in top signals`);
findings.topSignals.forEach(s => {
  console.log(`${s.name}: ${s.headline} (${s.score}/10)`);
});
```

### CLI

```bash
# Quick hunt (top 5 results)
gem-hunter hunt --quick

# Full hunt with category filter
gem-hunter hunt --track ai-tools --wait

# Get latest findings
gem-hunter findings --json > gems.json

# Watch for new signals
gem-hunter signals --stream
```

### Event-driven (Webhook)

```bash
# Configure webhook to receive hunt results
gem-hunter config --webhook https://your-api.example.com/webhooks/gems

# Server receives: POST /webhooks/gems
# Payload: { jobId, status, findings, timestamp }
```

---

## Configuration

```ts
const options = {
  // API endpoint (required for remote hunts)
  apiUrl: 'https://api.egos.ia.br/v1/gem-hunter',
  
  // API key (optional, default: process.env.GEM_HUNTER_API_KEY)
  apiKey: 'gk_eos_xxxxx',

  // Timeout for waitForJob (default: 10min)
  timeoutMs: 600_000,

  // Signal categories to track
  tracks: ['ai-tools', 'crypto', 'infrastructure'],

  // Minimum relevance score (0-10)
  minScore: 5,
};

const hunter = new GemHunter(options);
```

---

## Data model

```ts
interface GemResult {
  name: string;                   // "Llama 3.2"
  source: string;                 // "github" | "twitter" | "hackernews" | ...
  url: string;                    // https://github.com/meta-llama/llama
  description: string;
  stars?: number;                 // GitHub stars (if applicable)
  downloads?: number;             // npm downloads, etc.
  relevance: "high" | "medium" | "low";
  category: string;               // "ai-tools", "crypto", etc.
  lastUpdated?: string;           // ISO timestamp
  language?: string;              // "typescript", "rust", etc.
  license?: string;               // "MIT", "Apache-2.0", etc.
  structureBonus?: number;        // Architecture quality score
  abstractScore?: number;         // Abstraction level (0-10)
}

interface FindingsResult {
  latest: {
    generatedAt: string;
    totalGems: number;
    byCategory: Record<string, number>;
    bySource: Record<string, number>;
  };
  topSignals: Array<{
    name: string;
    url: string;
    score: number;                // 0-10
    category: string;
    date: string;                 // ISO timestamp
    headline: string;             // Summary of why it matters
  }>;
}
```

---

## How it works

1. **Hunt** — Trigger async discovery across sources
2. **Aggregate** — Normalize results (deduplicate, rank)
3. **Signal** — Emit high-signal items (score > threshold)
4. **Report** — Generate findings document + publish webhooks
5. **Archive** — Store for historical trend analysis

Each phase is independent, allowing parallel execution and graceful degradation (one source down doesn't block others).

---

## Real-world usage

Used in the EGOS core, which is private. The public part lives in [enioxt/cinco](https://github.com/enioxt/cinco).

---

## License

MIT — part of the EGOS ecosystem.

---

## See also

- [Guard Brasil](https://github.com/enioxt/guard-brasil-skill) — AI safety layer
- [Governance Skill](https://github.com/enioxt/egos-governance-skill) — Evidence-driven governance
- EGOS (núcleo privado) — [parte pública em enioxt/cinco](https://github.com/enioxt/cinco)
