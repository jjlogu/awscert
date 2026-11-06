# Practice: How to Use the Questions
> [Home](../README.md) · [Study plan](../00-study-plan.md) · [Exam strategy](../01-exam-strategy.md) · [Exam keywords](../cheatsheets/exam-keywords.md)

This folder has 125 original questions: four 15-question **domain sets** for learning and one **65-question mock** for measurement. Use the domain sets to learn how AWS writes questions, and the mock (plus external mocks) to decide whether you're ready. Mixing up those two purposes is the most common way to fool yourself about readiness.

---

## What's here

| File | Questions | Domain (weight) | Mode | When |
|---|---|---|---|---|
| [d1-secure.md](d1-secure.md) | 15 | D1 Secure (30%) | Untimed, learning | Q1–8 week 1 · Q9–15 week 3 |
| [d2-resilient.md](d2-resilient.md) | 15 | D2 Resilient (26%) | Untimed, learning | Q1–8 week 3 · Q9–15 week 4 |
| [d3-high-performing.md](d3-high-performing.md) | 15 | D3 High-performing (24%) | Untimed, learning | Q1–8 week 2 · Q9–15 week 4 |
| [d4-cost-optimized.md](d4-cost-optimized.md) | 15 | D4 Cost-optimized (20%) | Untimed, learning | All 15 in week 4 |
| [mock-exam-65.md](mock-exam-65.md) | 65 | All four, weighted like the exam | **Timed, 130 min, cold** | Week 5, Monday |

Each domain set is ordered to match the [study plan](../00-study-plan.md), so the first half draws on the modules you've read by then:

| Questions | Draws mainly on | Pair with lab |
|---|---|---|
| D1 Q1–8 | [01 IAM & multi-account](../notes/01-iam-multi-account.md), [02 Networking & VPC](../notes/02-networking-vpc.md) | [Lab 01](../labs/lab-01-account-iam.md), [Lab 02](../labs/lab-02-custom-vpc.md) |
| D1 Q9–15 | [07 Security & data protection](../notes/07-security-data-protection.md) | [Lab 04](../labs/lab-04-s3-cloudfront.md), [Lab 05](../labs/lab-05-rds-ha.md) |
| D2 Q1–8 | [05 Databases](../notes/05-databases.md), [06 Decoupling & serverless](../notes/06-decoupling-serverless.md) | [Lab 05](../labs/lab-05-rds-ha.md), [Lab 06](../labs/lab-06-serverless-decoupled.md) |
| D2 Q9–15 | [08 HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md), [03 Compute & containers](../notes/03-compute-containers.md), [04 Storage](../notes/04-storage.md) | [Lab 03](../labs/lab-03-alb-asg.md), [Lab 08](../labs/lab-08-iac-backup-dr.md) |
| D3 Q1–8 | [03 Compute & containers](../notes/03-compute-containers.md), [04 Storage](../notes/04-storage.md) (Q7 previews CloudFront from module 08, Q8 previews DynamoDB from module 05) | [Lab 03](../labs/lab-03-alb-asg.md), [Lab 04](../labs/lab-04-s3-cloudfront.md) |
| D3 Q9–15 | [05 Databases](../notes/05-databases.md), [08 HA, DR, edge & ops](../notes/08-ha-dr-edge-ops.md), [09 Data, analytics & migration](../notes/09-data-analytics-migration.md) | [Lab 07](../labs/lab-07-data-lake.md) |
| D4 Q1–15 | [10 Cost optimization](../notes/10-cost-optimization.md), plus [02](../notes/02-networking-vpc.md)–[05](../notes/05-databases.md) | The cost section of every lab |

Every answer block ends with the official **task statement** it tests and a **Review** link to the notes module that teaches it.

---

## How to use the domain sets (learning mode)

1. **Do them right after the matching module**, untimed and closed-book. The point is to practise reasoning, not speed.
2. **Commit before you look.** Write down your letter (or two letters) *and a one-line reason* before you open "Show answer". If you open it first, you're only recognizing the answer, not producing it.
3. **Read every explanation, including "Why not the others".** Distractors are real services used wrongly, and the wrong option in one question is often the right answer in the next one. Being able to say why each wrong option fails is the skill the exam measures.
4. **Count a lucky guess as a miss.** If you got it right but couldn't have explained why, log it.
5. **Score the set:** **13+/15 strong · 10–12 review misses · <10 re-read the module** (then redo its self-check and the missed questions).
6. **Redo missed questions 3–7 days later.** Repeat attempts are good for learning, but don't count them as readiness evidence, because you'll remember the answers.

---

## The 65-question mock (exam mode, week 5)

[mock-exam-65.md](mock-exam-65.md) is this guide's full-length mock, weighted like the exam (D1 20 · D2 17 · D3 15 · D4 13). Its questions are harder and more integrated than the domain sets, and some are "Choose three".

- **Simulate the exam.** One sitting, a **130-minute** timer, no notes, no pausing, phone in another room. If you have ESL +30, you can use 160 minutes, but aim to finish your first pass within the normal pacing.
- **Pace yourself at 2 minutes per question.** Rough checkpoints: question 20 by about 40 min, question 40 by about 80 min, and every question answered by about 120 min, which leaves 10 minutes for flagged questions. Use the flag-and-move method from [Exam strategy](../01-exam-strategy.md).
- **Answer everything.** There's no penalty for guessing on the real exam, so don't practise leaving blanks.
- **Score it two ways:** an overall percentage and a percentage **per domain** (each answer in the key is tagged with its domain and task statement).
- **Review every question the next day**, including the ones you got right. Were you right for the right reason?

Taken cold, this mock counts as one of your two readiness mocks. Take at least one **external** full-length mock as well, so you calibrate against a question style other than ours (see the [study plan](../00-study-plan.md), week 5).

---

## Readiness criteria

Book (or sit) the exam when **both** numeric criteria are true:

| Criterion | Target |
|---|---|
| Full-length mocks | **≥ 80%** on at least **2 reputable full-length mocks taken cold** (first attempt, questions never seen before) |
| Domain floor | **≥ 70%** in **every** domain on those mocks |

The [readiness criteria](../README.md#readiness-criteria-book-or-sit-the-exam-when-all-are-true) on the home page add two qualitative checks: you can explain why each wrong option is wrong, and you can sketch the core designs (2-AZ VPC, fan-out, the four DR strategies) from memory.

**Why 80% and not 72%?** The real exam is scored on a 100–1,000 scale with a pass mark of **720**, and there's no published mapping from percentage correct to scaled score. It also contains 15 unscored questions you can't identify, its wording differs from any practice bank, and nerves cost a few points. An 80% result on unfamiliar questions leaves a safety margin.

**Why a domain floor if scoring is compensatory?** You don't need to pass each domain, but a domain below 70% usually points to a whole category of questions you can't reason about. D1 is 30% of the exam, which is too big to be weak in.

**If you're not there yet:**

| Mock result | Do this |
|---|---|
| < 70% overall | Go back to the notes and labs for your two weakest domains, redo those domain sets, then take a new external mock. Use the week 6 buffer, and reschedule if needed (up to 24 h before the appointment, at most 2 reschedules per appointment). |
| 70–79% overall | Work through the error log by concept, re-read the *Common traps* sections for the weakest domain, and take another **new** cold mock after 3–4 days. |
| ≥ 80% but one domain < 70% | Spend 2 days on that domain only (notes → domain set misses → one lab), then take another cold mock. |

---

## The error log

Keep an `error-log.md` or spreadsheet, as the [study plan](../00-study-plan.md) describes. **For every miss (and every lucky guess), write four things:**

1. **The concept:** the rule being tested, in one line.
2. **Why my answer was wrong:** the specific flaw in the option you chose, not "I didn't know".
3. **The keyword I missed:** the phrase in the stem that should have pointed to the right answer ("including the root user", "within milliseconds", "UDP", "no code changes", "LEAST operational overhead").
4. **A link to the notes:** the module (and section) to re-read.

Template:

| Date | Source | Q | Concept | Why my answer was wrong | Keyword I missed | Notes link | Re-tested |
|---|---|---|---|---|---|---|---|
| *W3 Sun* | d1-secure | Q14 | Object Lock compliance mode: nobody, including root, can delete a locked version or shorten its retention | Chose governance mode: users with `s3:BypassGovernanceRetention` can still delete | "including … the root user", "shorten the retention period" | [07 Security](../notes/07-security-data-protection.md) | ✅ W4 |
| *W4 Sat* | d3-high-performing | Q11 | CloudFront doesn't carry UDP; static anycast IPs + UDP → Global Accelerator | Picked CloudFront because it's "the edge service" | "custom UDP protocol", "fixed set of IP addresses" | [08 HA/DR/edge](../notes/08-ha-dr-edge-ops.md) | ❌ redo W5 |
| | | | | | | | |

**Weekly review (Sunday):** sort the log by concept. Three or more entries on the same concept means a gap in understanding, not bad luck, so re-read that section and do its lab steps again. Add the keywords you keep missing to your own list, next to the [exam keywords cheat sheet](../cheatsheets/exam-keywords.md).

---

## Score log

Track every set and mock in one place. The "Actions" column is the important one: every score should lead to a concrete next step.

| Date | Set | Score | Weakest domain | Actions |
|---|---|---|---|---|
| | Skill Builder Official Practice Question Set (baseline) | __ / __ | | |
| | D1 Q1–8 | __ / 8 | D1 | |
| | D3 Q1–8 | __ / 8 | D3 | |
| | D1 Q9–15 | __ / 7 | D1 | |
| | D2 Q1–8 | __ / 8 | D2 | |
| | D2 Q9–15 | __ / 7 | D2 | |
| | D3 Q9–15 | __ / 7 | D3 | |
| | D4 Q1–15 | __ / 15 | D4 | |
| | mock-exam-65 (cold, timed) | __ % (D1 __ · D2 __ · D3 __ · D4 __) | | |
| | External full mock #1 (cold, timed) | __ % (D1 __ · D2 __ · D3 __ · D4 __) | | |
| | External full mock #2 (cold, timed) | __ % (D1 __ · D2 __ · D3 __ · D4 __) | | |

---

## External practice resources

These are widely used resources, listed here without endorsement. Content, format, availability, and pricing change, so check each one yourself before paying for anything.

| Resource | What it is | Best use |
|---|---|---|
| **AWS Skill Builder: Official Practice Question Set** (free) | A short set of exam-style questions written by AWS, with explanations | A cold **baseline** in week 1 (as the study plan schedules), and a reference for AWS's own wording |
| **AWS Skill Builder: Official Practice Exam** (requires a Skill Builder subscription) | A full-length practice exam from AWS in the real exam's format | One of your **readiness mocks** in week 5. Because AWS writes it, it's a good check on whether you can handle AWS's own question style |
| **AWS Skill Builder: Exam Prep courses** for SAA-C03 | AWS-produced review courses that walk through the exam domains | Structured domain review if a mock shows a broad weakness |
| **Tutorials Dojo (Jon Bonso) practice exams** | Third-party full-length practice exams, known for detailed explanations | External full mocks in week 5, and review mode for weak domains |
| **Stephane Maarek practice tests (Udemy)** | Third-party practice tests, often used alongside his video course | An additional source of cold, full-length mocks |
| **Adrian Cantrill's SAA-C03 course** | A deep, architecture-focused video course with demos (a course, not mainly a question bank) | Rebuilding understanding when your misses are conceptual rather than recall |

⚠️ **Third-party material lags behind AWS changes.** You may see questions that use the old SQS maximum message size (256 KB; it's now 1 MiB), call Amazon Quick "QuickSight", or still include Elastic Transcoder (removed from scope in exam guide v1.1). When a third-party answer conflicts with current AWS documentation, trust the documentation, and log the difference in your error log so it doesn't confuse you on exam day.

**Mixing sources:** use each external bank for **one cold full mock first**, before you use its review or section modes. Once you've seen a bank's questions, its scores no longer count as cold.

---

## ❌ Don't use brain dumps

"Dumps" are sites, files, or chat groups that claim to share **actual exam questions** ("real exam questions", "exam collection", "verified answers").

- **They violate the AWS Certification Program Agreement.** Exam content is confidential. AWS can **revoke certifications** and **ban candidates** from future exams for using or sharing it.
- **They're often wrong.** Answers are crowd-voted and frequently out of date, so you end up memorizing mistakes.
- **They don't teach the skill.** The exam rewards reasoning about trade-offs in scenarios you haven't seen. Memorized answers don't transfer, and question pools change with each exam guide update.

If a "practice" source promises the questions you'll actually see, walk away. All questions in this guide are original and written to teach the reasoning, not to imitate real exam items.
