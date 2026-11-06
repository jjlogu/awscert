# Exam Strategy: How to Take SAA-C03
> [Home](README.md) · [Study plan](00-study-plan.md) · [Exam keywords](cheatsheets/exam-keywords.md)

Knowing the services gets you to about 650. Reading questions the way AWS writes them gets you past 720. This page covers that second skill.

---

## 1. How SAA-C03 questions are built

Almost every question has the same shape:

```
[Business context]  A company runs … on-premises / on AWS …
[Constraint(s)]     … must remain available if an AZ fails … data must not traverse the internet …
[Qualifier]         Which solution meets these requirements with the LEAST operational overhead?
[4–6 options]       Usually 2 clearly wrong, 2 plausible. The qualifier decides between the plausible two.
```

- **Distractors are real services used wrongly.** They might be the wrong layer, more effort than needed, or missing one constraint.
- **Several options often "work."** You're choosing the one that best meets the *qualifier*.
- **The exam reflects AWS's preferences:** managed over self-managed, serverless when overhead matters, least privilege, encryption by default, multi-AZ for HA, and no over-engineering (don't go multi-Region when multi-AZ meets the requirement).

### The qualifiers and what they usually mean

| Qualifier | Usually points to | Usually wrong |
|---|---|---|
| **LEAST operational overhead / effort** | Managed or serverless (Fargate, Lambda, Aurora Serverless, DynamoDB, managed endpoints), native features over custom code | Self-managed EC2 fleets, cron scripts, custom Lambda glue when a native feature exists |
| **MOST cost-effective** | Right tier/purchase option, lifecycle policies, Spot for interruptible work, gateway endpoints, caching | Over-provisioning, multi-Region "just in case", NAT for S3 traffic |
| **MOST secure** | Private connectivity, least-privilege roles, KMS, no long-lived keys | Access keys on instances, public buckets, wide-open security groups |
| **Minimal application changes** | Lift-and-shift-friendly services (MQ, RDS with the same engine, EFS/FSx, ALB + ASG of the existing AMI) | Re-architecting to DynamoDB, Lambda or Kinesis |
| **Highly available** | Multi-AZ, Auto Scaling, health checks, managed failover | Single instance, single NAT GW, single AZ |
| **As quickly as possible / near real-time** | Streaming (Kinesis), event-driven, managed migration tools | Batch jobs, manual steps |

Full mapping (120+ phrases): **[cheatsheets/exam-keywords.md](cheatsheets/exam-keywords.md)**

---

## 2. The 6-step dissection method (about 90 seconds per question)

1. **Read the last sentence first.** That's the actual question and its qualifier.
2. **Underline the constraints** in the stem: availability, data residency, protocols (NFS/SMB/UDP), latency, budget, "no code changes", "existing on-premises AD"…
3. **Eliminate the clearly wrong two.** Look for the wrong service category, a broken constraint, or something that doesn't exist (e.g., "enable cross-Region replication on an EBS volume").
4. **Compare the remaining two on the qualifier only.** Ask which needs less custom work, costs less, or is more secure.
5. **Sanity check:** prefer native and managed over custom, and private over internet paths. If an option needs you to "write a script that…", be suspicious.
6. **Commit and move on.** Flag it if you're unsure. Don't burn 5 minutes.

### Multiple-response questions ("Choose two" / "Choose three")
- The stem tells you **exactly** how many to pick. There's no partial credit, so treat each option as an independent true/false.
- The correct pair is usually **complementary** (e.g., "create a gateway endpoint" + "update the bucket policy"), not two alternatives to the same step.

---

## 3. Time management

130 minutes for 65 questions is **2 minutes per question**. Long scenario questions take longer; short definition-style ones take 30 seconds.

| Pass | Time budget | What to do |
|---|---|---|
| **Pass 1** | ~90 min | Answer **every** question. If you take more than 2.5 min or feel unsure, pick your best answer, **flag**, and move on. |
| **Pass 2** | ~30 min | Revisit flagged questions only. A later question sometimes jogs your memory. |
| **Final** | ~10 min | Confirm **no blanks**: unanswered questions score zero and guessing costs nothing. |

- ⚠️ Don't change answers without a concrete reason. A first instinct backed by a keyword is usually right; a vague second thought usually isn't.
- If you get ESL +30, you have 160 minutes. Use the extra time for Pass 2, not for a slower Pass 1.

---

## 4. Booking the exam

1. Sign in to your **AWS Certification account** (from [aws.amazon.com/certification](https://aws.amazon.com/certification/)) and schedule with **Pearson VUE**.
2. Choose **test center** or **online proctored**:

| | Test center | Online proctored |
|---|---|---|
| Pros | Controlled environment, no tech risk at home | No travel, more time slots |
| Cons | Travel, fewer slots | Strict room rules (clear desk, no second monitor, webcam on), and connectivity problems can end the session |
| Tips | Arrive early with your ID | Run Pearson VUE's **system test** on the exact machine and network days before. Use a wired connection if possible and close all apps. |

3. **Book in week 4** for a date at the end of week 5 or week 6. A fixed date improves focus, and you can still reschedule.

### ESL +30 accommodation
If English is **not your first language**, you can get **30 extra minutes**. Request it once in your AWS Certification account (**Request Exam Accommodations → ESL +30 MINUTES**) **before you register**. It then applies to all your future registrations. There's no reason not to use it if you're eligible.

### Policies you should know
| Policy | Rule |
|---|---|
| Reschedule / cancel | Up to **24 hours** before the appointment. An appointment can be rescheduled at most **2 times**. A no-show or a cancellation inside 24 h forfeits the fee. |
| Results | Usually **no pass/fail on screen**. Results are posted within **5 business days** in your Certification account. If you pass, the Credly badge email may arrive first. |
| Retake | Wait **14 calendar days** after a failed attempt. There's no attempt limit, and you pay the full fee each time. |
| After passing | **50% discount voucher** for another exam (or recertification). The certification is valid for **3 years**. |

> Verify the current details on [AWS Certification policies](https://aws.amazon.com/certification/policies/) before booking; policies can change.

---

## 5. Exam week and exam day

**Exam week**
- [ ] No new topics in the last 48 hours. Review only the cheat sheets and your error log.
- [ ] Re-read the *Common traps* section of every notes module (about 1 hour total).
- [ ] Online exam: run the system test again and prepare the room.

**Exam day**
- [ ] Eat beforehand and keep water nearby if the rules allow it (test center: lockers; online: check the rules).
- [ ] Bring a valid government ID with your name exactly as registered.
- [ ] Use the **Help** feature for full service names if a short name is unfamiliar.
- [ ] Expect about 15 questions on things you've never seen. Many will be unscored. Don't panic: use elimination and move on.

---

## 6. If the result isn't a pass
- The score report shows **section-level** strengths and weaknesses. Map them to the [coverage matrix](README.md#coverage-matrix-official-task-statements--modules).
- You can retake after 14 days. Spend week 1 on the two weakest domains (notes, labs, domain questions) and week 2 on two new external mocks.
- Remember that compensatory scoring rewards overall breadth: shoring up a weak domain from 55% to 70% is usually worth more than polishing a strong one.

---

## 7. AWS's design defaults (when in doubt, these usually win)
1. **Managed > self-managed.** Use RDS, not MySQL on EC2, unless the stem requires OS access or an unsupported feature.
2. **Roles > keys.** Never use access keys on EC2/Lambda/ECS. Use instance profiles, execution roles and task roles.
3. **Private > public paths.** Use VPC endpoints, PrivateLink and Direct Connect. Avoid NAT or the internet for AWS-service traffic when a private path exists.
4. **Loose coupling.** Put a queue or event bus between tiers that scale differently.
5. **Multi-AZ for HA, multi-Region for DR or global latency.** Don't escalate beyond what the stem asks.
6. **Encrypt by default.** Use KMS at rest, TLS in transit and ACM for certificates.
7. **Automate.** Use IaC (CloudFormation), Auto Scaling, lifecycle policies and AWS Backup plans, not manual runbooks.
