# Auto Research Looping

An autonomous AI agent skill that runs an infinite optimization loop: **modify → run → measure → keep or revert → repeat**. Set it up, walk away, come back to a table of completed experiments.

Inspired by [Karpathy's autoresearch](https://github.com/karpathy/autoresearch), generalized for any domain with a measurable metric.

## How It Works

```
┌─────────────────────────────────────────┐
│              DEFINE                      │
│  target file + eval command + metric     │
└──────────────────┬──────────────────────┘
                   ▼
┌─────────────────────────────────────────┐
│           BASELINE                       │
│  branch off, run eval, record baseline   │
└──────────────────┬──────────────────────┘
                   ▼
         ┌────────────────┐
         │  READ current   │◄──────────────┐
         │  state + logs   │               │
         └───────┬────────┘               │
                 ▼                         │
         ┌────────────────┐               │
         │  HYPOTHESIZE    │               │
         │  a change       │               │
         └───────┬────────┘               │
                 ▼                         │
         ┌────────────────┐               │
         │  EDIT target    │               │
         │  file only      │               │
         └───────┬────────┘               │
                 ▼                         │
         ┌────────────────┐               │
         │  GIT COMMIT     │               │
         └───────┬────────┘               │
                 ▼                         │
         ┌────────────────┐               │
         │  RUN eval       │               │
         └───────┬────────┘               │
                 ▼                         │
         ┌────────────────┐    improved    │
         │  COMPARE metric │──────────────►│
         └───────┬────────┘  keep + log    │
                 │ worse                   │
                 ▼                         │
         ┌────────────────┐               │
         │  REVERT + LOG   │───────────────┘
         └────────────────┘
```

The loop runs until the budget is exhausted or you interrupt it.

## Requirements

- Git (for commit-per-experiment audit trail)
- A target file to optimize
- An eval command that outputs a parseable metric
- A budget (max experiments, hours, or cost)

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Agent instructions — the executable skill definition |
| `results.tsv` | Generated at runtime — experiment log |

## Quick Start

Give the agent these four things:

```
Target:  prompt.txt
Eval:    python evaluate.py --prompt prompt.txt
Metric:  accuracy ↑
Budget:  50 experiments
```

The agent will:
1. Create branch `autoresearch/prompt-tuning`
2. Run baseline eval and record the starting metric
3. Enter the loop — modify, eval, keep/revert, log
4. Stop at 50 experiments and print a summary

## Supported Domains

| Domain | Target | Metric |
|--------|--------|--------|
| ML training | `train.py` | val_loss ↓ |
| Prompt engineering | `prompt.txt` | accuracy ↑ |
| RAG pipelines | `config.yaml` | retrieval_precision ↑ |
| SQL queries | `query.sql` | exec_time_ms ↓ |
| API optimization | `handler.py` | p99_latency ↓ |
| Hyperparameters | `hparams.json` | val_metric ↑ |
| Feature engineering | `features.py` | model_auc ↑ |
| Networking config | `nginx.conf` | requests_per_sec ↑ |
| Docker images | `Dockerfile` | image_size_mb ↓ |
| Regex patterns | `patterns.yaml` | f1_score ↑ |
| Compression | `compress.conf` | compression_ratio ↑ |
| Database indexing | `indexes.sql` | avg_query_time_ms ↓ |
| Search ranking | `ranking.py` | ndcg@10 ↑ |
| Caching strategy | `cache_config.yaml` | hit_rate ↑ |
| Data pipeline ETL | `transform.py` | rows_per_second ↑ |

Anything with a scalar metric and a fast feedback loop works.

## Output

Every run produces a `results.tsv`:

```
commit    metric_1  status   description
a1b2c3d   0.9979    keep     baseline
b2c3d4e   0.9932    keep     increased learning rate to 0.04
c3d4e5f   1.0050    discard  switched to GeLU activation
d4e5f6g   0.0000    crash    doubled model width (OOM)
```

And a final summary:

```
=== AUTO RESEARCH COMPLETE ===
Total experiments: 50
Kept: 12 | Discarded: 35 | Crashed: 3
Best metric: 0.9710 (experiment #38)
Improvement over baseline: 2.7%
Branch: autoresearch/prompt-tuning
```

## Key Principles

- **One file only** — the agent can only modify the target file, nothing else
- **Commit every experiment** — full git audit trail, trivial rollback
- **Log everything** — even crashes and timeouts go into results.tsv
- **Never stop early** — the agent runs until budget is exhausted
- **Revert failures** — bad experiments are rolled back immediately
- **Review periodically** — every 10 experiments, the agent analyzes what worked

## Budget Controls

```
max_experiments: 50
max_wall_clock_hours: 8
max_cost_usd: 10.00
abort_on_consecutive_crashes: 3
```

## Advanced Usage

See `autoexp-generic-expanded.md` for:
- **Explore/Exploit mode** — periodic random exploration to escape local optima
- **Batch mode** — run each experiment K times for noisy metrics
- **Multi-stage pipelines** — sequential optimization of dependent stages
- **Dual/triple metric strategies** — Pareto, weighted, threshold-gated, lexicographic
- **Warm starting** — seed with results from previous runs
- **Parallel branches** — multiple agents optimizing different aspects simultaneously
- **Meta-optimization** — optimizing the agent's own exploration strategy

## License

Public domain. Use however you want.
