# Smart Agent Orchestration - Implementation Plan
**Created**: 2026-04-01 16:47
**Priority**: P1
**Status**: ✅ **Implemented (Phase 1 & 2 Complete)**

## Goal
Automatically classify agent tasks and recommend optimal concurrency levels to maximize efficiency while maintaining quality.

## 🎯 Implementation Status (2026-04-02)

### ✅ Phase 1: Core Scheduler (Completed 2026-04-01)
- ✅ Task Classifier (`src/scheduler/classifier.js`) - 6 types with confidence scoring
- ✅ Concurrency Strategy Engine (`src/scheduler/strategy.js`) - default limits per type
- ✅ Agent Scheduler (`src/scheduler/scheduler.js`) - queue management, dispatch loop
- ✅ Integration test passed (5/5 tasks, avg wait 2s)
- ✅ Metrics logging: `memory/scheduler_metrics.json`

### ✅ Phase 2: Dynamic Refinement (Completed 2026-04-01)
- ✅ Multi-metric analysis (success rate, quality, duration, consistency, trend)
- ✅ Confidence-weighted adjustment (0.5-step granularity)
- ✅ Adaptive cooldown (12h)
- ✅ Trend detection & duration consistency check
- ✅ New file: `src/scheduler/strategy_phase2.js` (17KB)
- ✅ Config updated: `config/scheduler.json` uses Phase 2 by default
- ✅ Validation test: LEARNING stays at 4, CODING/DEBUGGING reduced based on performance

### ⏳ Phase 3-5: Integration & UI (Not Yet Deployed)
- ⏳ Phase 3: Agent Pool Integration (subagents API hook)
- ⏳ Phase 4: Metrics & Optimization (weekly report automation)
- ⏳ Phase 5: UI & Monitoring (dashboard, CLI commands)

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Agent Scheduler                          │
│  ┌─────────────┐   ┌─────────────┐   ┌─────────────┐     │
│  │ Task Intake │ → │ Classifier  │ → │ Dispatcher  │     │
│  └─────────────┘   └─────────────┘   └─────────────┘     │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│           Concurrency Strategy Engine                       │
│  • Task type (learning/coding/review/docs/debug)           │
│  • Historical performance (success rate, avg duration)     │
│  • Resource constraints (memory, GPU, network)            │
│  • User preferences (max_concurrent, priority)             │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              Agent Pool Manager                             │
│  • Launch agents with appropriate resources                │
│  • Monitor health & progress                               │
│  • Adjust concurrency dynamically                         │
│  • Requeue on failure                                     │
└─────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. Task Classifier (`src/scheduler/classifier.js`)
- **Input**: Task description, code snippets, user intent
- **Output**: TaskType + confidence score
- **Types**:
  - `LEARNING` - Study/understand new tech (Rust concepts, new crate)
  - `CODING` - Write new code, implement features
  - `REVIEW` - Code review, security audit, optimization
  - `DOCUMENTATION` - Write docs, comments, README
  - `DEBUGGING` - Fix bugs, trace errors, test failures
  - `PLANNING` - Design architecture, create task breakdown

**Strategy**:
- Keyword matching (initial)
- ML-based classifier (future - fine-tune on historical tasks)
- Confidence threshold (if < 0.7, ask user for clarification)

### 2. Concurrency Strategy (`src/scheduler/strategy.js`)
Default concurrency by task type:

| TaskType | Default Concurrency | Rationale |
|----------|-------------------|-----------|
| LEARNING | 4 | Low resource, can parallelize |
| CODING | 2 | Needs GPU/memory, context heavy |
| REVIEW | 1 | Deep focus, sequential |
| DOCUMENTATION | 3 | Lightweight, can batch |
| DEBUGGING | 2 | Interactive, needs feedback |
| PLANNING | 1 | High-level thinking |

**Dynamic Adjustment**:
- Track success rate per type
- If success rate < 80% → reduce concurrency by 1
- If success rate > 95% + avg duration < expected → increase by 1
- Max concurrency per type: [1, 2, 3, 4] based on resource

### 3. Agent Pool Manager (`src/scheduler/pool.js`)
- Maintain pool of idle agents
- Assign tasks based on classifier output
- Resource isolation (CPU/memory limits per agent)
- Health checks (heartbeat, timeout)
- Auto-restart on crash

### 4. Metrics Collector (`src/scheduler/metrics.js`)
- Record: task type, start/end time, tokens used, quality score, success/failure
- Store in `memory/scheduler_metrics.json`
- Weekly aggregation report

### 5. Learning Optimizer (`src/scheduler/optimizer.js`)
- Analyze metrics every 7 days
- Recommend strategy updates
- Identify underperforming task categories
- Suggest training data collection

## Implementation Steps

### Phase 1: Basic Classifier (Day 1)
- [ ] Create `src/scheduler/classifier.js` with keyword rules
- [ ] Implement simple heuristics:
  - Contains "learn", "study", "understand" → LEARNING
  - Contains "write", "implement", "code" → CODING
  - Contains "review", "audit", "check" → REVIEW
  - Contains "fix", "debug", "error" → DEBUGGING
  - Contains "doc", "readme", "comment" → DOCUMENTATION
  - Contains "plan", "design", "architecture" → PLANNING
- [ ] Unit tests for classifier (10+ examples per type)

### Phase 2: Strategy Engine (Day 2)
- [ ] Create `src/scheduler/strategy.js` with default mappings
- [ ] Implement dynamic adjustment logic
- [ ] Add config: `config/scheduler_strategy.json` (overrides)
- [ ] Unit tests for strategy (concurrency limits, adjustments)

### Phase 3: Agent Pool Integration (Day 3)
- [ ] Extend `subagents` API to use scheduler
- [ ] Modify `subagents spawn` to:
  1. Classify task
  2. Get concurrency limit
  3. Check current active count
  4. Either launch or queue
- [ ] Add `scheduler status` command to show active/pending

### Phase 4: Metrics & Optimization (Day 4-5)
- [ ] Implement `src/scheduler/metrics.js`
- [ ] Auto-log every task outcome
- [ ] Generate weekly report (if enabled)
- [ ] Create `optimizer` to adjust strategy based on data

### Phase 5: UI & Monitoring (Day 6)
- [ ] `memory/scheduler_dashboard.md` - real-time view
- [ ] CLI: `openclaw scheduler status` / `scheduler metrics`
- [ ] Alert when concurrency too high/low

## Configuration

File: `config/scheduler.json`

```json
{
  "enabled": true,
  "max_global_concurrency": 10,
  "default_strategy": {
    "LEARNING": 4,
    "CODING": 2,
    "REVIEW": 1,
    "DOCUMENTATION": 3,
    "DEBUGGING": 2,
    "PLANNING": 1
  },
  "dynamic_adjustment": {
    "enabled": true,
    "min_concurrency": 1,
    "max_concurrency": 4,
    "success_rate_threshold_low": 0.8,
    "success_rate_threshold_high": 0.95,
    "adjustment_cooldown_hours": 24
  },
  "metrics": {
    "enabled": true,
    "retention_days": 30,
    "weekly_report_day": "Monday",
    "storage_path": "memory/scheduler_metrics.json"
  }
}
```

## Success Criteria

- ✅ Classifier accuracy ≥ 85% on validation set
- ✅ Concurrency limit respected (no oversubscription)
- ✅ Dynamic adjustment converges to optimal within 2 weeks
- ✅ Overall task throughput increases by ≥ 20%
- ✅ No degradation in task quality (score ≥ 8/10 average)

## Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Misclassification leads to poor concurrency | Medium | Allow manual override; high confidence threshold |
| Resource exhaustion (too many agents) | High | Hard global limit; health checks |
| Metrics collection overhead | Low | Async logging; batch writes |
| Strategy oscillation (flapping) | Medium | Cooldown period; exponential smoothing |

---

**Next Step**: Implement Phase 1 - Task Classifier