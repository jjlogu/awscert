# Study Plan: 5 Weeks (+1 Buffer) at ~15 Hours/Week
> [Home](README.md) · [Exam strategy](01-exam-strategy.md) · [Labs](labs/README.md) · [Practice](practice/README.md)

**Total:** about 75 hours over 5 weeks, plus an optional buffer week. Weekdays are ~1.5–2 h blocks; weekends are ~3–4 h.
**Weekly mix:** about 6 h reading, 5 h labs, 4 h practice/review.
**Order is deliberate:** weeks 1–3 cover the areas developers usually haven't designed (identity across accounts, VPC networking, hybrid, data protection). Weeks 3–4 cover resilience, data and cost. Week 5 is exam mode.

> **Shift days freely.** Keep the sequence (notes → lab → questions) and the Sunday review. Don't skip the Sunday review.

---

## Before you start (30 min)
- [ ] Read [README](README.md) and skim [Exam strategy](01-exam-strategy.md).
- [ ] Create a study log: a simple spreadsheet or `error-log.md` with columns **Date · Source · Q · Concept · Why I was wrong · Keyword missed · Notes link**.
- [ ] Decide your target exam date: end of week 5 (aggressive) or end of week 6 (comfortable). You'll **book it in week 4**.

---

## Week 1: Identity, multi-account & VPC networking (the biggest developer gap)

| Day | Time | Do |
|---|---|---|
| Mon | 2 h | **Baseline:** take the free AWS Skill Builder *Official Practice Question Set* cold (~40 min) and record the score. It's not a judgement, just a starting point. Then do [Lab safety](labs/README.md): root MFA, budget alert, pick one Region (~1 h). |
| Tue | 2 h | [Notes 01](notes/01-iam-multi-account.md): shared responsibility, IAM building blocks, **policy evaluation logic**, roles & STS |
| Wed | 2 h | [Notes 01](notes/01-iam-multi-account.md): Identity Center, federation, Directory Service, Organizations/SCPs, Control Tower, RAM, Cognito → self-check |
| Thu | 2 h | [Lab 01: Account & IAM](labs/lab-01-account-iam.md) |
| Fri | 1.5 h | [Notes 02](notes/02-networking-vpc.md): CIDR, subnets, route tables, IGW, NAT, SG vs NACL |
| Sat | 3.5 h | [Notes 02](notes/02-networking-vpc.md): endpoints, PrivateLink, peering vs Transit Gateway, VPN, Direct Connect, hybrid DNS → self-check |
| Sun | 2.5 h | [Lab 02: Custom VPC](labs/lab-02-custom-vpc.md) (1.5 h) · [D1 questions](practice/d1-secure.md) Q1–8 + error log (1 h) |

**Gate (Sunday):** without notes, can you…
- draw a 2-AZ VPC with public/private subnets, per-AZ NAT gateways and the route tables?
- explain why a security group can't block one specific IP, and what to use instead?
- say what an SCP can and can't do, and to whom it doesn't apply?

---

## Week 2: Compute, containers & storage

| Day | Time | Do |
|---|---|---|
| Mon | 2 h | [Notes 03](notes/03-compute-containers.md): EC2 families, purchasing options, placement groups, instance store vs EBS |
| Tue | 2 h | [Notes 03](notes/03-compute-containers.md): Auto Scaling policies, ALB/NLB/GWLB, Lambda nuances, ECS/EKS/Fargate, Batch, Outposts/Wavelength → self-check |
| Wed | 2 h | [Lab 03: ALB + Auto Scaling](labs/lab-03-alb-asg.md) |
| Thu | 2 h | [Notes 04](notes/04-storage.md): S3 classes, lifecycle, versioning, replication, Object Lock, encryption, performance |
| Fri | 1.5 h | [Notes 04](notes/04-storage.md): EBS types, EFS, FSx family, Storage Gateway, AWS Backup → self-check |
| Sat | 3.5 h | [Lab 04: S3 + CloudFront](labs/lab-04-s3-cloudfront.md) (2.5 h) · [D3 questions](practice/d3-high-performing.md) Q1–8 (1 h) |
| Sun | 2 h | **Review:** error log · [Decision tables](cheatsheets/decision-tables.md) for S3 classes, EBS, storage choice, compute choice, purchasing, load balancers · redo the Week 1 self-checks (spaced repetition) |

**Gate:** can you pick the storage service and class from three facts (access pattern, protocol/OS, latency/durability)? Can you explain when to use target tracking vs step vs scheduled vs predictive scaling?

---

## Week 3: Databases, decoupling & security

| Day | Time | Do |
|---|---|---|
| Mon | 2 h | [Notes 05](notes/05-databases.md): RDS Multi-AZ vs read replicas, Aurora (Global, Serverless v2), RDS Proxy |
| Tue | 2 h | [Notes 05](notes/05-databases.md): DynamoDB (capacity, GSIs, DAX, global tables), ElastiCache, Redshift, purpose-built DBs, DMS → self-check |
| Wed | 2 h | [Lab 05: RDS high availability](labs/lab-05-rds-ha.md). ⚠️ Do it **in one sitting** because Multi-AZ costs money per hour. |
| Thu | 2 h | [Notes 06](notes/06-decoupling-serverless.md): SQS, SNS fan-out, EventBridge, MQ, Step Functions, API Gateway → self-check |
| Fri | 1.5 h | [Notes 07](notes/07-security-data-protection.md): KMS, CloudHSM, ACM, Secrets Manager vs Parameter Store |
| Sat | 4 h | [Notes 07](notes/07-security-data-protection.md): WAF, Shield, GuardDuty/Macie/Inspector/Detective/Security Hub, CloudTrail vs Config → self-check (1.5 h) · [Lab 06: Serverless & decoupled](labs/lab-06-serverless-decoupled.md) (2.5 h) |
| Sun | 2 h | [D1](practice/d1-secure.md) Q9–15 · [D2](practice/d2-resilient.md) Q1–8 · error log review |

**Gate:** choose between SQS standard, SQS FIFO, SNS, EventBridge and Kinesis from one sentence of requirements. Explain envelope encryption, and why encrypting an existing RDS database means snapshot → encrypted copy → restore.

---

## Week 4: HA/DR, edge, data & cost

| Day | Time | Do |
|---|---|---|
| Mon | 2 h | [Notes 08](notes/08-ha-dr-edge-ops.md): RPO/RTO, the four DR strategies, Route 53 routing policies & health checks |
| Tue | 2 h | [Notes 08](notes/08-ha-dr-edge-ops.md): CloudFront, Global Accelerator, CloudFormation, CloudWatch/X-Ray/Systems Manager, quotas → self-check |
| Wed | 2 h | [Notes 09](notes/09-data-analytics-migration.md): Kinesis/Firehose/MSK, Glue/Athena/Lake Formation/EMR/Redshift, DataSync/Snow/Transfer Family/MGN → self-check |
| Thu | 2 h | [Lab 07: Data lake](labs/lab-07-data-lake.md) (1.5 h) · start [Notes 10](notes/10-cost-optimization.md) (0.5 h) |
| Fri | 1.5 h | [Notes 10](notes/10-cost-optimization.md): purchasing, storage/database/network cost, cost tools → self-check |
| Sat | 3.5 h | [Lab 08: IaC, backup & DR](labs/lab-08-iac-backup-dr.md) (2 h) · [D2](practice/d2-resilient.md) Q9–15 · [D3](practice/d3-high-performing.md) Q9–15 (1.5 h) |
| Sun | 2 h | [D4](practice/d4-cost-optimized.md) all 15 (1 h) · [Exam keywords](cheatsheets/exam-keywords.md) (1 h) · **Book the exam** for the end of week 5 or week 6 (see [booking](01-exam-strategy.md#4-booking-the-exam)) |

**Gate:** given an RPO/RTO and a budget, can you name the DR strategy and the services it needs? Can you list five ways to cut a data-transfer bill?

---

## Week 5: Exam mode

| Day | Time | Do |
|---|---|---|
| Mon | 2.25 h | **[65-question mock](practice/mock-exam-65.md)**: strictly timed, 130 min, no notes |
| Tue | 2 h | Review **every** mock question, including the ones you got right (were you right for the right reason?). Update the error log. |
| Wed | 2 h | Re-read the *How the exam tests this* and *Common traps* sections of your **2 weakest** modules |
| Thu | 2.25 h | **External full mock #1**, timed (e.g., AWS Skill Builder Official Practice Exam or Tutorials Dojo; see [practice/README](practice/README.md)) |
| Fri | 1.5 h | Review mock #1 and update the error log |
| Sat | 4 h | **External full mock #2**, timed (2.25 h) + review (1.75 h) |
| Sun | 2 h | [Decision tables](cheatsheets/decision-tables.md) · [Numbers](cheatsheets/numbers-to-remember.md) · whole error log · **Go / no-go decision** using the [readiness criteria](README.md#readiness-criteria-book-or-sit-the-exam-when-all-are-true) |

---

## Week 6 (buffer): use it only if you need it
- **Go** (≥ 80% twice, no domain < 70%): sit the exam early in the week. The day before, do light review only (cheat sheets + error log, ≤ 2 h) and get a full night's sleep.
- **Not yet:** spend 2 days on the weakest domain (notes → redo that domain's questions → one more lab), 2 days on one more external mock + review, then reassess. Reschedule if needed: changes are allowed up to 24 h before the exam (see [policies](01-exam-strategy.md#policies-you-should-know)).

---

## 4-week compressed variant (~19 h/week)
For a hard deadline. Skips or shortens 3 labs; everything else stays.

| Week | Notes | Labs | Practice |
|---|---|---|---|
| 1 | 01, 02, 03 | 01, 02 | D1 Q1–8 |
| 2 | 04, 05, 06 | 04, 06 (Lab 03 and 05: read the steps and *Exam takeaways* only) | D3 Q1–8, D2 Q1–8 |
| 3 | 07, 08, 09, 10 | 08 (Lab 07: read only) | D1 Q9–15, D2 Q9–15, D3 Q9–15, D4 all |
| 4 | Cheat sheets | — | 65-Q mock + 2 external mocks, each with full review |

---

## Habits that move the score
- **Explain the wrong options.** For each practice question, say in one line why each distractor fails. That's the skill the exam measures.
- **Error log over re-reading.** Re-reading notes feels productive; reviewing *why you were wrong* actually is.
- **Spaced repetition.** Each Sunday, redo the previous weeks' module self-checks (about 10 min each).
- **Think like a reviewer, not a builder.** On the exam, "it works" isn't enough. The best answer meets the qualifier (*least operational overhead*, *most cost-effective*…) with managed services.
- **Labs: clean up the same day.** Use the checklist at the end of each lab.
