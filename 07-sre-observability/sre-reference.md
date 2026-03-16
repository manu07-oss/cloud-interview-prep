# 📊 SRE & Observability — Complete Reference

---

## 1. SRE Core Concepts

### SLI, SLO, SLA — The Holy Trinity

```
SLI = Service Level Indicator
    → The actual measurement
    → Examples: availability %, latency p99, error rate %

SLO = Service Level Objective
    → Your internal TARGET for the SLI
    → Example: "99.9% availability per month"
    → This is what your TEAM commits to

SLA = Service Level Agreement
    → Legal CONTRACT with customers
    → Usually LOWER than SLO (buffer!)
    → Example: SLO = 99.9%, SLA = 99.5% (give buffer for compensation)

If you BREACH SLA → you pay penalties (credits, refunds)
If you BREACH SLO → internal alert, fix before it becomes SLA breach
```

### Error Budget
```
Error Budget = 100% - SLO

If SLO = 99.9% → Error Budget = 0.1%
In a month (30 days = 43,200 minutes):
  0.1% of 43,200 = 43.2 minutes of allowed downtime per month

Error budget is SHARED between:
  - Planned maintenance
  - Unexpected incidents
  - Bad deployments

If error budget is LOW:
  → Slow down releases
  → Focus on reliability improvements

If error budget is HIGH:
  → Can move faster, take more risks
  → Ship features quickly

This creates healthy tension between Dev (ship fast) and Ops (stay stable)
```

### Toil
```
Toil = Manual, repetitive, automatable work with no lasting value

Examples of toil:
  - Manually restarting services when they fail
  - Manually scaling servers before big events
  - Manually running deployment scripts
  - Answering same on-call alerts repeatedly

SRE Rule: Keep toil below 50% of work time
Rest = engineering work (automation, improvements)

Reduce toil by:
  - Automating runbooks
  - Self-healing systems (liveness probes, auto-restart)
  - Predictive auto-scaling
  - Better alerting (fix root cause, not symptom)
```

---

## 2. Observability — The Three Pillars

```
Observability = ability to understand system state from its OUTPUTS

Three Pillars:

1. METRICS    → Numbers over time
               → CPU 85%, request rate 1000/sec, error rate 2%
               → What: aggregated numbers
               → Tools: Prometheus, CloudWatch, Azure Monitor

2. LOGS       → Event records
               → "2024-01-15 14:32:01 ERROR: DB connection failed"
               → What: detailed individual events
               → Tools: Elasticsearch, CloudWatch Logs, Log Analytics

3. TRACES     → Request journey through system
               → Request → API Gateway → Service A → DB → Cache → Response
               → What: end-to-end request path + timing
               → Tools: Jaeger, Zipkin, X-Ray, App Insights, Tempo
```

### What to Monitor — Golden Signals (Google SRE Book)
```
1. LATENCY    → How long requests take
               → Track p50, p95, p99 (not just average!)
               → p99 = 99% of requests are faster than this

2. TRAFFIC    → How many requests per second
               → HTTP requests, DB queries, queue depth

3. ERRORS     → Rate of failed requests
               → HTTP 5xx, failed DB queries, exceptions

4. SATURATION → How "full" is the system
               → CPU %, memory %, disk %, connection pool usage

Monitor ALL FOUR for every service!
```

### USE Method (for infrastructure)
```
U = Utilization → How busy is the resource (CPU 80%)
S = Saturation  → How much work is queued/waiting
E = Errors      → Rate of errors for the resource

Apply to: CPU, Memory, Disk, Network, Connection Pools
```

---

## 3. Alerting Best Practices

```
Good alerts:
  ✓ Alert on SYMPTOMS not causes
    BAD: "CPU > 90%"  ← cause
    GOOD: "Error rate > 1% for 5 minutes"  ← symptom

  ✓ Every alert should have a runbook
  ✓ Alert should be ACTIONABLE (human must do something)
  ✓ Don't alert on things that can auto-heal
  ✓ Tune alert thresholds (avoid alert fatigue)

Alert Levels:
  PAGE (wake someone up): user-facing impact, SLO at risk
  TICKET (fix next business day): internal issue, no user impact
  LOG (just record it): informational

Alert Fatigue:
  → Too many alerts → engineers ignore them
  → Regular review: which alerts fired but required no action?
  → Those should be deleted or downgraded
```

### Alerting Tools by Cloud
```
Azure:   Azure Monitor Alerts → Action Groups → PagerDuty / OpsGenie / Slack
AWS:     CloudWatch Alarms → SNS → PagerDuty / OpsGenie / Slack
GCP:     Cloud Monitoring Alerts → Notification Channels → PagerDuty

On-call Management:
  PagerDuty  → industry standard, escalation policies
  OpsGenie   → Atlassian, good Jira integration
  VictorOps  → Splunk product
```

---

## 4. Incident Management — Full Process

```
DETECT
  → Alert fires OR user reports issue
  → Automated health check fails

RESPOND (first 5 minutes)
  → On-call engineer acknowledges alert
  → Create incident channel (#incident-2024-01-15)
  → Assess severity (SEV1/2/3/4)
  → Notify stakeholders if SEV1/SEV2

INVESTIGATE
  → Check dashboards (metrics, logs, traces)
  → Identify what changed (recent deployments?)
  → Correlate symptoms
  → Form hypotheses and test

MITIGATE (reduce user impact FIRST)
  → Rollback bad deployment
  → Redirect traffic to healthy region
  → Enable maintenance mode
  → Scale up resources
  → Goal: restore service, root cause analysis comes LATER

RESOLVE
  → Service restored
  → Update stakeholders
  → Close incident

POST-MORTEM (within 48 hours)
  → Blameless review
  → What happened? Timeline.
  → Why did it happen? (5 Whys)
  → What could we detect it faster?
  → Action items to prevent recurrence
```

### Severity Levels
```
SEV1 (Critical): Production down, all users affected
  → Response: immediately (24/7)
  → Update: every 15 minutes

SEV2 (Major): Significant feature broken, many users affected
  → Response: within 30 minutes
  → Update: every 30 minutes

SEV3 (Minor): Some users affected, workaround exists
  → Response: within 2 hours (business hours)
  → Update: daily

SEV4 (Informational): Degraded performance, minimal user impact
  → Response: next business day
  → Creates a ticket
```

### Post-Mortem — 5 Whys Example
```
Incident: Database went down → 45 minutes downtime

Why 1: Why did the database go down?
  → Disk full

Why 2: Why was the disk full?
  → Log files grew unexpectedly large

Why 3: Why did log files grow so large?
  → Debug logging was accidentally left enabled in production

Why 4: Why was debug logging left on?
  → Deployment script didn't reset log level after debugging

Why 5: Why didn't the script reset it?
  → No check in CI/CD pipeline for log level configuration

Root Cause: No automated config validation in CI/CD
Fix: Add config validation step to pipeline
```

---

## 5. Chaos Engineering

```
"Practice failure before it practices you"

Chaos Engineering = deliberately inject failures in controlled way
  to verify your system handles them gracefully

Tools:
  Azure: Azure Chaos Studio
  AWS:   AWS Fault Injection Simulator (FIS)
  GCP:   No native tool (use Chaos Monkey / Litmus Chaos)
  K8s:   Chaos Monkey for K8s, Litmus Chaos, Chaos Mesh

Common Experiments:
  - Kill random pods (does K8s restart them? Does traffic continue?)
  - Terminate an AZ (does traffic fail over?)
  - Add network latency (does app degrade gracefully or fail?)
  - Simulate DB slowdown (does connection pooling handle it?)
  - Kill a region (does DR kick in within RTO?)

Process:
  1. Define hypothesis: "If we kill Zone A, users in Zone B are unaffected"
  2. Run experiment in non-prod first
  3. Observe metrics
  4. Validate or disprove hypothesis
  5. Fix weaknesses found
  6. Graduate to production experiment
```

---

## 6. SRE Interview Q&A

**Q: What is an SLO and how do you set one?**
> SLO = Service Level Objective, your internal target for service reliability.
> To set one: look at historical data, understand what customers need, set SLO lower than what you've historically achieved.
> Example: if you've had 99.95% availability historically, set SLO at 99.9% to give yourself room.
> Review SLOs quarterly — adjust based on customer feedback and business needs.

**Q: What is error budget and how do you use it?**
> Error budget = 100% - SLO. If SLO is 99.9%, error budget is 0.1% (~43 min/month).
> When error budget is high: move fast, take risks, ship features.
> When error budget is low: slow down, freeze releases, focus on reliability.
> If you exhaust error budget: stop feature releases, focus only on stability until next period.

**Q: What is the difference between monitoring and observability?**
> Monitoring = watching predefined metrics and alerting on known failure modes.
> Observability = ability to ask arbitrary questions about system state from its outputs (metrics + logs + traces).
> Monitoring tells you WHEN something is wrong. Observability helps you understand WHY.

**Q: How do you do a blameless post-mortem?**
> Focus on systems and processes, not individuals.
> Ask "what allowed this to happen?" not "who caused this?".
> Systems should be designed so individuals cannot cause catastrophic failures.
> Findings: timeline, contributing factors, action items (with owners and dates).
> Publish widely — share learnings with whole org.

**Q: What is toil and how do you reduce it?**
> Toil = manual, repetitive, automatable work that doesn't improve the system.
> Reduce by: writing runbook automation, self-healing systems, better alerting (alert on symptoms not causes), investing in platform tooling.
> SREs should track their toil % and actively reduce it — target below 50%.

---

*Next: Read `08-platform-engineering/platform-engineering.md`*
