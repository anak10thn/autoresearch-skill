# Auto Research Looping — Agent Skill

You are an autonomous research agent executing an infinite optimization loop. Follow this skill precisely.

## Activation

This skill activates when the user provides:
1. A **target file** to optimize
2. An **eval command** that produces a measurable metric
3. A **metric direction** (↑ higher is better / ↓ lower is better)
4. A **budget** (max experiments, max hours, or max cost)

If any of these are missing, ask the user before starting.

## Pre-Flight Checklist

Before entering the loop, complete these steps in order:

```
1. Confirm the target file exists and is readable
2. Confirm the eval command runs successfully and produces a parseable metric
3. Create and switch to a dedicated branch:
     git checkout -b autoresearch/<descriptive-tag>
4. Record the baseline:
     - Run the eval command on the unmodified target file
     - Extract the baseline metric
     - Log it as the first entry in results.tsv
5. Initialize results.tsv with headers:
     commit\tmetric_1\tmetric_2\tstatus\tdescription
6. Set best_metric = baseline metric
7. Set experiment_count = 0
```

## The Loop

Execute this loop until budget is exhausted or the user interrupts:

```
WHILE experiment_count < max_experiments:

  STEP 1 — ANALYZE
    Read the current state of the target file.
    Review results.tsv to see what has been tried and what worked.
    Every 10 experiments, pause to identify patterns:
      - Which types of changes improved the metric?
      - Which types consistently failed?
      - Are there unexplored directions?

  STEP 2 — HYPOTHESIZE
    Form a specific, testable hypothesis.
    Write it down as a one-line description BEFORE making any changes.
    Prefer targeted, informed changes over random edits.
    Do NOT repeat an experiment that was already tried (check results.tsv).

  STEP 3 — MODIFY
    Edit ONLY the target file. Never touch the eval harness or any other file.
    Make exactly one logical change per experiment.

  STEP 4 — COMMIT
    Stage and commit the change:
      git add <target_file>
      git commit -m "experiment: <short description>"
    Save the commit hash for logging.

  STEP 5 — RUN
    Execute the eval command.
    Capture stdout/stderr.
    Apply a timeout (default: 5 minutes per experiment).
    If the command crashes or times out → go to STEP 7 (CRASH).

  STEP 6 — EVALUATE
    Extract the metric(s) from the eval output.
    Compare against best_metric.

    IF metric improved (or equals best in the desired direction):
      best_metric = new metric
      Log: commit, metric, "keep", description
      Continue (do NOT revert)

    IF metric did NOT improve:
      Log: commit, metric, "discard", description
      Revert: git reset --hard HEAD~1

  STEP 7 — CRASH HANDLING
    If the experiment crashed or timed out:
      Log: commit, "crash"/"timeout", 0.0, description
      Revert: git reset --hard HEAD~1
      Increment consecutive_crash_count
      If consecutive_crash_count >= 3:
        STOP and report: "3 consecutive crashes. Review the last 3 attempts."
    If the experiment did NOT crash:
      Reset consecutive_crash_count = 0

  STEP 8 — INCREMENT
    experiment_count += 1
    Print progress: "Experiment {n}/{max}: {status} | metric: {value} | best: {best_metric}"

END WHILE
```

## Results File Format

Maintain `results.tsv` — tab-separated, one row per experiment:

```
commit	metric_1	metric_2	status	description
a1b2c3d	0.9979	—	keep	baseline
b2c3d4e	0.9932	—	keep	increased learning rate to 0.04
c3d4e5f	1.0050	—	discard	switched to GeLU activation
d4e5f6g	0.0000	—	crash	doubled model width (OOM)
```

Valid statuses: `keep` | `discard` | `crash` | `timeout` | `explore`

If metric_2 is not used, write `—`.

## Metric Comparison Rules

### Single Metric
- Direction ↑: new > best → keep
- Direction ↓: new < best → keep

### Dual Metric (choose one strategy)

**Primary/Secondary**: Improve primary; secondary must not degrade beyond a threshold.
```
keep IF primary_improved AND secondary >= (best_secondary * threshold)
```

**Pareto**: Keep if better on at least one metric without regressing on the other.
```
keep IF (m1_improved AND m2_not_worse) OR (m2_improved AND m1_not_worse)
```

**Weighted**: Collapse to single scalar.
```
score = w1 * normalize(metric_1) + w2 * normalize(metric_2)
keep IF score > best_score
```

## Exploration Mode

Every 5th experiment, optionally enter exploration mode:
- Make a bolder or more creative change than usual
- Keep the change regardless of metric outcome
- Log with status `explore` instead of `keep`/`discard`
- This prevents getting stuck in local optima

## Constraints

### What You MUST Do
- Only modify the designated target file
- Git commit before every eval run
- Log every experiment to results.tsv, no exceptions
- Revert failed experiments immediately
- Respect the budget (experiments, time, cost)
- Print progress after each experiment

### What You MUST NOT Do
- Modify the eval harness or any file other than the target
- Skip logging (even crashes must be logged)
- Run experiments beyond the budget
- Make multiple unrelated changes in a single experiment
- Repeat an experiment that was already tried with the same parameters
- Stop the loop voluntarily (only budget exhaustion or user interrupt)

## Completion

When the loop ends (budget exhausted), produce a summary:

```
=== AUTO RESEARCH COMPLETE ===
Total experiments: {count}
Kept: {keep_count} | Discarded: {discard_count} | Crashed: {crash_count}
Best metric: {best_metric} (experiment #{best_experiment_number})
Improvement over baseline: {percentage}%
Branch: autoresearch/<tag>
Results: results.tsv
```

## Quick Start Examples

### Example 1: Prompt Optimization
```
Target: prompt.txt
Eval: python evaluate.py --prompt prompt.txt
Metric: accuracy ↑
Budget: 50 experiments
```

### Example 2: SQL Query Tuning
```
Target: query.sql
Eval: ./bench_query.sh query.sql
Metric: execution_time_ms ↓
Budget: 30 experiments
```

### Example 3: ML Hyperparameters
```
Target: hparams.json
Eval: python train.py --config hparams.json && python eval.py
Metric: val_loss ↓
Budget: 40 experiments
```

### Example 4: Nginx Config Tuning
```
Target: nginx.conf
Eval: ./reload_and_bench.sh
Metric: requests_per_sec ↑ (constraint: error_rate < 0.1%)
Budget: 30 experiments
```

### Example 5: Feature Engineering
```
Target: features.py
Eval: python train_and_eval.py --features features.py
Metric: model_auc ↑
Budget: 40 experiments
```

### Example 6: Docker Image Optimization
```
Target: Dockerfile
Eval: docker build -t test . && docker images test --format '{{.Size}}'
Metric: image_size_mb ↓
Budget: 40 experiments
```

### Example 7: Regex Accuracy
```
Target: patterns.yaml
Eval: python test_patterns.py --rules patterns.yaml
Metric: f1_score ↑
Budget: 60 experiments
```
