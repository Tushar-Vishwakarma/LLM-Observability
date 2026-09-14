# LLM-Observability
<img width="800" height="440" alt="Screen Recording 2026-09-14 at 3 59 16 PM (1)" src="https://github.com/user-attachments/assets/749d0cc1-33ed-40ed-bca8-8fb1908145ea" />

## 1. Problem Statement

As organizations scale LLM adoption across hundreds of applications and teams, quality control breaks down. Each team evaluates their LLM outputs independently — different metrics, different thresholds, different tooling, often only *before* launch (offline eval). Once live, nobody has visibility into:

- Whether output quality is degrading over time (model updates, prompt drift, data drift)
- Whether one team's "good" is the same as another team's "good"
- Who owns fixing a quality regression when it's found

At scale (500+ apps, 200+ teams), this isn't a tooling gap, it's a governance gap. There's no shared language for "quality," no live signal, and no clear accountability.

## 2. Goal

Build a centralized platform that gives every team a **live, standardized view of LLM output quality**: replacing ad hoc, offline, team-specific evaluation with continuous, comparable, organization-wide observability.

## 3. Guiding Principles (the Three Pillars)

**Data-Driven**
Every decision, rollout, rollback, flagging a team as at-risk, is backed by a measured score, not a gut call. The platform's core value is turning "is this LLM output good?" from a subjective judgment into a number everyone agrees on.

**Experiment & Measure**
Change should never ship blind. Any prompt, model, or config change gets measured against the current version, on live traffic, before it fully rolls out. Measurement isn't a one-time gate, it's continuous.

**Ownership**
Central teams shouldn't be the bottleneck for quality. Each team owns their own thresholds, sees their own scorecard, and gets alerted directly. Central platform provides the *infrastructure and standard*; teams own the *outcome*.

## 4. North Star Metric

### Trustworthy Response Rate (TRR)

**Definition:** The percentage of LLM responses, across an app or the whole org, that meet a combined bar for accuracy, relevance, and safety — as measured by an automated LLM-judge, sampled continuously against live traffic.

**Why this is the North Star:**
- It's a single number leadership, teams, and engineers can all rally around, avoids the trap of optimizing one metric (e.g. low hallucination) while ignoring another (e.g. irrelevant but "safe" answers)
- It directly reflects user trust, which is the actual business risk of ungoverned LLM outputs, not a proxy metric like "number of evals run"
- It's comparable across teams and app types, which solves the actual hard problem stated: centralizing evaluation only works if the same score means the same thing everywhere

**Target: ≥95%**
Reasoning: at 200+ teams' scale, even a 5% failure rate compounds into a large absolute number of bad user-facing responses per day. 95% is an aggressive but achievable bar for mature products (comparable to enterprise SLA-style thresholds used for other trust/quality metrics like uptime or CSAT), while leaving room for teams to flag <95% as "needs attention" without treating it as an outage-level failure.

## 5. Supporting Metrics

TRR is a composite. Each supporting metric isolates one failure mode so teams know *what* to fix, not just *that* something's wrong.

| Metric | Definition | Target | Why this threshold |
|---|---|---|---|
| **Hallucination Rate** | % of responses containing fabricated or unverifiable claims | <2% | Hallucination is the highest-risk failure mode (erodes trust fastest, hardest to catch downstream); kept near-zero deliberately |
| **Relevance Score** | 0–1 scale, LLM-judged: does the response address the actual query | ≥0.90 | Below 0.9, users typically perceive answers as "off-topic" even if factually correct |
| **Safety Violation Rate** | % of responses flagged for harmful or policy-violating content | <0.1% | Zero-tolerance category; even rare violations carry outsized reputational/legal risk |
| **Latency (P95)** | 95th percentile response time | <2s, or app-specific SLA | Anchored to typical user patience thresholds for interactive AI features; app teams can override for batch/async use cases |
| **Cost per Call** | Average $ per LLM call (input + output tokens) | Team-defined budget | No universal target — cost tolerance varies by app's business value; owned by team, tracked centrally for visibility |
| **User Feedback Score** | % positive signal from thumbs up/down or CSAT, where instrumented | ≥85% | Real-world validation layer against the automated judge — catches cases where LLM-judge and actual users disagree |
| **Drift Score** | Change in output distribution vs. a rolling baseline | Flag if >15% week-over-week shift | Early-warning signal — catches silent degradation (e.g. after a model version update) before TRR itself drops |

**How they roll up:** TRR is computed as a weighted composite of Hallucination Rate, Relevance Score, and Safety Violation Rate (the three dimensions the LLM-judge scores directly per response). Latency, Cost, and User Feedback are tracked alongside as guardrail/context metrics — they inform whether a high TRR is also efficient and validated by real users, but don't dilute the core trust signal.

## 6. Core Features

### 6.1 Org-Wide Dashboard (Data-Driven pillar)
Single view across all apps/teams: current TRR, trend, last-evaluated timestamp. Lets leadership and platform teams spot at-risk apps in seconds instead of chasing individual team reports.

### 6.2 Live Evaluation Engine
An LLM-as-judge pipeline scores prompt/response pairs against Hallucination, Relevance, and Safety in real time (or on a continuous sample of live traffic), replacing manual, one-off offline review.

### 6.3 Experimentation Framework (Experiment & Measure pillar)
Any change (prompt, model, config) can be tested as Version A vs. B against live or shadow traffic, with a measured score difference and confidence indication before full rollout. Includes an auto-pause safeguard if a rollout's quality drops below threshold.

### 6.4 Team Scorecards (Ownership pillar)
Each team sees only their own apps' scores, sets their own alert thresholds (within org-wide guardrails), and gets a self-serve audit trail connecting changes to quality impact — no central team required to investigate.

### 6.5 Metrics Glossary
A single canonical reference for what every metric means, its target, and how it rolls up — solving the "different teams, different definitions of good" problem directly.

## 7. Sampling Strategy (addressing scale)

Evaluating 100% of live traffic across 500+ apps isn't cost-effective. Proposed approach:
- **High-confidence, stable apps** (TRR consistently ≥95%, low drift): sample 5–10% of traffic
- **Low-confidence or newly onboarded apps**: sample up to 100% until a stable baseline is established
- **Any app showing drift >15%**: automatically increase sampling rate until drift resolves

This keeps evaluation cost proportional to risk rather than uniform across all traffic.

## 8. Success Metrics for the Platform Itself

- % of the 200+ teams actively using their scorecard (adoption)
- Reduction in average time-to-detect a quality regression (from ad hoc/manual to automated)
- % of experiments run through the platform before rollout, vs. shipped without measurement
- Org-wide average TRR trend over time

## 9. Open Questions / Future Scope

- How do we handle apps where "relevance" or "safety" definitions differ meaningfully by domain (e.g. legal vs. casual chat)?
- Should teams be allowed to define custom supporting metrics beyond the standard set, and if so, how do they still roll up to a comparable TRR?
- Human-in-the-loop calibration: how often should the LLM-judge itself be audited against human raters to prevent judge drift?

