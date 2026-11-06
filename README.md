# AWS Certified Solutions Architect – Associate (SAA-C03): Study Guide

A 5-week, about 75-hour study guide for an experienced software developer moving into architecture. It assumes you already build on AWS (EC2, S3, Lambda, RDS). It spends the most time on what developers usually haven't had to design: **networking, identity across accounts, hybrid connectivity, resilience/DR and cost**.

> Exam facts last verified against official AWS pages on **2026-11-05**. Re-check the [official exam page](https://aws.amazon.com/certification/certified-solutions-architect-associate/) before you book.

---

## Exam at a glance

| Item | Detail |
|---|---|
| Exam code | **SAA-C03** (current; no successor announced) |
| Questions | **65**: 50 scored + 15 unscored (you can't tell which) |
| Types | Multiple choice (1 of 4) · Multiple response (2+ of 5+) |
| Duration | **130 minutes** (+30 min with the [ESL +30 accommodation](01-exam-strategy.md#esl-30-accommodation) if English isn't your first language) |
| Cost | 150 USD |
| Scoring | Scaled 100–1,000 · **pass = 720** · compensatory (you don't need to pass each domain) · no penalty for guessing |
| Delivery | Pearson VUE: test center or online proctored |
| Validity | 3 years |

| Domain | Weight | Task statements |
|---|---|---|
| D1 Design Secure Architectures | **30%** | 1.1 Secure access · 1.2 Secure workloads · 1.3 Data security controls |
| D2 Design Resilient Architectures | **26%** | 2.1 Scalable & loosely coupled · 2.2 Highly available / fault tolerant |
| D3 Design High-Performing Architectures | **24%** | 3.1 Storage · 3.2 Compute · 3.3 Database · 3.4 Network · 3.5 Data ingestion & transformation |
| D4 Design Cost-Optimized Architectures | **20%** | 4.1 Storage · 4.2 Compute · 4.3 Database · 4.4 Network |

### What changed recently (exam guide v1.1, published Oct 29, 2026)
AWS says exam-guide updates reach the live exam about **one month** after publication, so expect these from roughly late November 2026:
- **Amazon Elastic Transcoder** was removed from the in-scope services.
- Amazon QuickSight now appears as **Amazon Quick**. It is the same BI/dashboard service; older practice material still says "QuickSight".
- The exam uses **official short service names** (e.g., "Amazon SNS", not "Amazon Simple Notification Service"). The in-exam **Help** feature lists the full names. This guide uses short names throughout.

---

## How to use this guide

Each topic follows the same loop:

```
Read the notes module  →  Do the matching lab  →  Answer the domain questions  →  Log every miss
        ↑                                                                            |
        └──────────────── review your error log every Sunday ────────────────────────┘
```

1. Start with the **[Study plan](00-study-plan.md)**. It tells you what to do each day.
2. Read **[Exam strategy](01-exam-strategy.md)** once in week 1 and again in week 5.
3. Do **[Lab safety](labs/README.md)** before any lab: MFA, a budget alert, and the cleanup habit.
4. In week 5, take the **[65-question mock](practice/mock-exam-65.md)** timed, plus at least two reputable external mocks.
5. Book the exam when you score **≥ 80% on two full mocks taken cold**.

---

## File index

| Area | Files |
|---|---|
| Plan & strategy | [00 Study plan](00-study-plan.md) · [01 Exam strategy](01-exam-strategy.md) |
| **Notes** (topic modules) | [01 IAM & multi-account](notes/01-iam-multi-account.md) · [02 Networking & VPC](notes/02-networking-vpc.md) · [03 Compute & containers](notes/03-compute-containers.md) · [04 Storage](notes/04-storage.md) · [05 Databases](notes/05-databases.md) · [06 Decoupling & serverless](notes/06-decoupling-serverless.md) · [07 Security & data protection](notes/07-security-data-protection.md) · [08 HA, DR, edge & operations](notes/08-ha-dr-edge-ops.md) · [09 Data, analytics & migration](notes/09-data-analytics-migration.md) · [10 Cost optimization](notes/10-cost-optimization.md) |
| **Cheat sheets** | [Decision tables](cheatsheets/decision-tables.md) · [Exam keywords → answers](cheatsheets/exam-keywords.md) · [Numbers to remember](cheatsheets/numbers-to-remember.md) |
| **Practice** | [How to practice](practice/README.md) · [D1 Secure](practice/d1-secure.md) · [D2 Resilient](practice/d2-resilient.md) · [D3 High-performing](practice/d3-high-performing.md) · [D4 Cost-optimized](practice/d4-cost-optimized.md) · [Full 65-Q mock](practice/mock-exam-65.md) |
| **Labs** | [Lab safety & index](labs/README.md) · [01 Account & IAM](labs/lab-01-account-iam.md) · [02 Custom VPC](labs/lab-02-custom-vpc.md) · [03 ALB + Auto Scaling](labs/lab-03-alb-asg.md) · [04 S3 + CloudFront](labs/lab-04-s3-cloudfront.md) · [05 RDS high availability](labs/lab-05-rds-ha.md) · [06 Serverless & decoupled](labs/lab-06-serverless-decoupled.md) · [07 Data lake](labs/lab-07-data-lake.md) · [08 IaC, backup & DR](labs/lab-08-iac-backup-dr.md) |
| **Career** | [Next steps after SAA](career/next-steps.md) |

---

## Coverage matrix: official task statements → modules

The notes are organized **by topic**, because S3, VPC and databases show up in every domain. This matrix maps each official task statement to the modules that cover it, so you can still track blueprint coverage. **Bold** marks the primary module.

| Task statement | Notes modules | Labs | Practice |
|---|---|---|---|
| **1.1** Design secure access to AWS resources | **[01](notes/01-iam-multi-account.md)**, [07](notes/07-security-data-protection.md) | [01](labs/lab-01-account-iam.md) | [D1](practice/d1-secure.md) |
| **1.2** Design secure workloads and applications | **[02](notes/02-networking-vpc.md)**, **[07](notes/07-security-data-protection.md)**, [01](notes/01-iam-multi-account.md), [06](notes/06-decoupling-serverless.md) | [02](labs/lab-02-custom-vpc.md), [04](labs/lab-04-s3-cloudfront.md) | [D1](practice/d1-secure.md) |
| **1.3** Determine appropriate data security controls | **[07](notes/07-security-data-protection.md)**, [04](notes/04-storage.md), [05](notes/05-databases.md) | [04](labs/lab-04-s3-cloudfront.md), [05](labs/lab-05-rds-ha.md) | [D1](practice/d1-secure.md) |
| **2.1** Design scalable and loosely coupled architectures | **[06](notes/06-decoupling-serverless.md)**, [03](notes/03-compute-containers.md), [05](notes/05-databases.md), [08](notes/08-ha-dr-edge-ops.md) | [03](labs/lab-03-alb-asg.md), [06](labs/lab-06-serverless-decoupled.md) | [D2](practice/d2-resilient.md) |
| **2.2** Design highly available and/or fault-tolerant architectures | **[08](notes/08-ha-dr-edge-ops.md)**, [03](notes/03-compute-containers.md), [04](notes/04-storage.md), [05](notes/05-databases.md), [02](notes/02-networking-vpc.md) | [03](labs/lab-03-alb-asg.md), [05](labs/lab-05-rds-ha.md), [08](labs/lab-08-iac-backup-dr.md) | [D2](practice/d2-resilient.md) |
| **3.1** High-performing and/or scalable storage | **[04](notes/04-storage.md)**, [09](notes/09-data-analytics-migration.md) | [04](labs/lab-04-s3-cloudfront.md) | [D3](practice/d3-high-performing.md) |
| **3.2** High-performing and elastic compute | **[03](notes/03-compute-containers.md)**, [06](notes/06-decoupling-serverless.md) | [03](labs/lab-03-alb-asg.md), [06](labs/lab-06-serverless-decoupled.md) | [D3](practice/d3-high-performing.md) |
| **3.3** High-performing database solutions | **[05](notes/05-databases.md)** | [05](labs/lab-05-rds-ha.md) | [D3](practice/d3-high-performing.md) |
| **3.4** High-performing and/or scalable network architectures | **[02](notes/02-networking-vpc.md)**, [08](notes/08-ha-dr-edge-ops.md), [03](notes/03-compute-containers.md) | [02](labs/lab-02-custom-vpc.md), [03](labs/lab-03-alb-asg.md) | [D3](practice/d3-high-performing.md) |
| **3.5** High-performing data ingestion and transformation | **[09](notes/09-data-analytics-migration.md)** | [07](labs/lab-07-data-lake.md) | [D3](practice/d3-high-performing.md) |
| **4.1** Cost-optimized storage | **[10](notes/10-cost-optimization.md)**, [04](notes/04-storage.md), [09](notes/09-data-analytics-migration.md) | [04](labs/lab-04-s3-cloudfront.md) | [D4](practice/d4-cost-optimized.md) |
| **4.2** Cost-optimized compute | **[10](notes/10-cost-optimization.md)**, [03](notes/03-compute-containers.md) | [03](labs/lab-03-alb-asg.md) | [D4](practice/d4-cost-optimized.md) |
| **4.3** Cost-optimized databases | **[10](notes/10-cost-optimization.md)**, [05](notes/05-databases.md) | [05](labs/lab-05-rds-ha.md) | [D4](practice/d4-cost-optimized.md) |
| **4.4** Cost-optimized network architectures | **[10](notes/10-cost-optimization.md)**, [02](notes/02-networking-vpc.md), [08](notes/08-ha-dr-edge-ops.md) | [02](labs/lab-02-custom-vpc.md) | [D4](practice/d4-cost-optimized.md) |

---

## Progress tracker

Tick these off as you go. VS Code's Markdown preview and GitHub both render checkboxes.

**Week 1: Identity & networking**
- [ ] Baseline: AWS Skill Builder Official Practice Question Set, taken cold (score: ___ )
- [ ] Lab safety setup (MFA, budget alert)
- [ ] Notes 01 IAM & multi-account · [ ] Lab 01
- [ ] Notes 02 Networking & VPC · [ ] Lab 02
- [ ] D1 questions 1–8

**Week 2: Compute & storage**
- [ ] Notes 03 Compute & containers · [ ] Lab 03
- [ ] Notes 04 Storage · [ ] Lab 04
- [ ] D3 questions 1–8

**Week 3: Databases, decoupling, security**
- [ ] Notes 05 Databases · [ ] Lab 05
- [ ] Notes 06 Decoupling & serverless · [ ] Lab 06
- [ ] Notes 07 Security & data protection
- [ ] D1 questions 9–15 · [ ] D2 questions 1–8

**Week 4: Resilience, data, cost**
- [ ] Notes 08 HA, DR, edge & ops · [ ] Lab 08
- [ ] Notes 09 Data, analytics & migration · [ ] Lab 07
- [ ] Notes 10 Cost optimization
- [ ] D2 9–15 · [ ] D3 9–15 · [ ] D4 1–15
- [ ] Exam booked (date: ___ )

**Week 5: Exam mode**
- [ ] 65-Q mock, timed (score: ___ %)
- [ ] External full mock #1 (score: ___ %)
- [ ] External full mock #2 (score: ___ %)
- [ ] Error log reviewed end to end · [ ] Cheat sheets reviewed

---

## Readiness criteria (book or sit the exam when all are true)
1. **≥ 80%** on at least **two** full-length, reputable mocks taken **cold** (first attempt, not a retake).
2. No domain below **70%** in your last two mocks.
3. You can explain **why each wrong option is wrong** for at least 9 of 10 questions you review.
4. You can sketch from memory: a 2-AZ VPC with public/private subnets and NAT, a fan-out (SNS → SQS) design, and the four DR strategies with their RPO/RTO.

---

## Official sources
- [Exam page](https://aws.amazon.com/certification/certified-solutions-architect-associate/) · [Exam guide (SAA-C03)](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/solutions-architect-associate-03.html) · [In-scope services](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/saa-03-in-scope-services.html) · [Revisions](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03/saa-03-revisions.html)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/): the lens the exam is written through
- [AWS Skill Builder](https://skillbuilder.aws/): free Official Practice Question Set and exam-prep courses
- [AWS Certification policies](https://aws.amazon.com/certification/policies/) · [FAQs](https://aws.amazon.com/certification/faqs/)
