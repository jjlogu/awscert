# Labs: lab safety first

> [Home](../README.md) · [Study plan](../00-study-plan.md) · [Cost optimization notes](../notes/10-cost-optimization.md)

These eight labs are where the exam's trade-offs stop being abstract. You will build a VPC by hand, break a NACL on purpose, fail over an RDS instance, and watch a poison message land in a DLQ. Do this page once before Lab 01 (about 45 minutes). Most of the money people waste on AWS study comes from skipping it.

**The rule:** every lab ends with its cleanup checklist, done the same day. Budgets and anomaly alerts only back you up. They do not replace cleanup.

---

## 1. Account setup

### 1.1 Use a dedicated study account

Do not use your employer's account or the account that runs your side project. A separate account gives you:

- a clean bill, so any charge is something a lab created
- freedom to use broad admin permissions without risking anything real
- the option to close it after the exam

Sign up at the AWS sign-up page with an email you control. A plus-address such as `you+saa@example.com` works with most mail providers and gives you a unique email for later member accounts too.

### 1.2 Free plan or Paid plan?

Accounts created on or after July 15, 2025 choose a plan at sign-up:

| | Free plan | Paid plan |
|---|---|---|
| Credits | $100 at sign-up, plus up to $100 more for onboarding activities | Same $100 + up to $100 |
| What happens when credits run out | Account **closes** (you get 90 days to upgrade and recover it) | Usage is billed to your card |
| Time limit | Ends after **6 months or when credits run out**, whichever comes first | None |
| Service access | **Some services and features are restricted** | Full access |
| Risk profile | Hard spending cap (you cannot be surprised by a bill) | You can overspend, so Budgets matter more |

**The trade-off for these labs.** The Free plan is a strong safety net. However, some lab steps use features that may be restricted on it, such as AWS Organizations (Lab 01's optional part, and IAM Identity Center's multi-account mode). Also, a 5–6 week study plan that keeps NAT gateways, ALBs and Multi-AZ RDS running can use up credits quickly. Two sensible approaches:

- **Start on the Free plan.** If a lab step is blocked, upgrade to the Paid plan at that point. Per AWS, your remaining credits stay usable after the upgrade; check the Free Tier FAQ for the current expiry terms.
- **Choose the Paid plan from day one** and rely on section 2 (Budgets + anomaly detection + same-day cleanup). Credits still apply first.

Either way, labs that use a NAT gateway, ALB or Multi-AZ RDS **consume credits**. The cost-risk table in section 6 shows how fast.

### 1.3 Root user hygiene (do these before anything else)

1. Sign in as root → **account menu (top right) → Security credentials**.
2. **Multi-factor authentication (MFA) → Assign MFA device.** A passkey or security key is best; an authenticator app is fine. AWS now requires root MFA for most accounts, so check that it is on.
3. **Access keys:** there must be **none**. Root access keys are the single most dangerous credential in AWS. If one exists, delete it.
4. **Account → IAM user and role access to Billing information → Edit → Activate.** Without this, your future admin (an IAM Identity Center user) cannot see Billing, Budgets or Cost Explorer, even with AdministratorAccess. Only root can turn this on.
5. **Account → Alternate contacts:** add a billing contact (you).

After Lab 01 step 6, **stop using root**. Lab 01 creates your day-to-day admin through IAM Identity Center. The only things root should do after that are the few tasks that require it, such as closing the account.

---

## 2. Billing guardrails (before any lab)

You are still signed in as root here, because no other identity exists yet. That is fine for these billing tasks.

### 2.1 AWS Budgets: a zero-spend budget plus a monthly budget

**Why two budgets?** Credits offset charges. A budget that counts credits can stay at $0 while a NAT gateway burns through them. The zero-spend budget tells you the moment real money is charged. The monthly budget excludes credits, so it measures *usage* and warns you while credits are still covering it.

**Budget 1: zero spend (template)**

1. **Billing and Cost Management console → Budgets → Create budget**.
2. **Use a template (simplified) → Zero spend budget**.
3. Name: `saa-zero-spend`. Email recipients: your address. **Create budget**.

**Budget 2: monthly usage budget (custom)**

1. **Budgets → Create budget → Customize (advanced) → Cost budget → Next**.
2. Name `saa-monthly-usage`. Period **Monthly**, renewal **Recurring**, method **Fixed**, amount e.g. **10.00 USD**. Pick a number that would annoy you but not hurt you.
3. **Budget scope:** All AWS services. Under **Advanced options**, keep *Unblended costs* and **clear the Credits checkbox** (and Refunds) in the supported charge types. The budget now tracks gross usage.
4. **Configure alerts:** add three thresholds:
   - 50% **Actual**
   - 80% **Actual**
   - 100% **Forecasted**

   Each one emails you.
5. **Create budget**.

CLI equivalent (run in CloudShell after section 4 is set up):

```bash
EMAIL=you@example.com
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)

cat > budget.json <<'EOF'
{
  "BudgetName": "saa-monthly-usage",
  "BudgetLimit": {"Amount": "10", "Unit": "USD"},
  "BudgetType": "COST",
  "TimeUnit": "MONTHLY",
  "CostTypes": {"IncludeCredit": false, "IncludeRefund": false}
}
EOF

cat > notifications.json <<EOF
[
  {"Notification": {"NotificationType": "ACTUAL", "ComparisonOperator": "GREATER_THAN", "Threshold": 50, "ThresholdType": "PERCENTAGE"},
   "Subscribers": [{"SubscriptionType": "EMAIL", "Address": "$EMAIL"}]},
  {"Notification": {"NotificationType": "ACTUAL", "ComparisonOperator": "GREATER_THAN", "Threshold": 80, "ThresholdType": "PERCENTAGE"},
   "Subscribers": [{"SubscriptionType": "EMAIL", "Address": "$EMAIL"}]},
  {"Notification": {"NotificationType": "FORECASTED", "ComparisonOperator": "GREATER_THAN", "Threshold": 100, "ThresholdType": "PERCENTAGE"},
   "Subscribers": [{"SubscriptionType": "EMAIL", "Address": "$EMAIL"}]}
]
EOF

aws budgets create-budget --account-id "$ACCOUNT_ID" \
  --budget file://budget.json \
  --notifications-with-subscribers file://notifications.json
```

⚠️ Trap: Budgets data refreshes only **up to three times a day**. A forgotten NAT gateway can run for most of a day before an email arrives. That delay is why same-day cleanup is the real control.

### 2.2 Cost Anomaly Detection

1. **Billing and Cost Management console → Cost Anomaly Detection → Cost monitors.** Many new accounts already have a default *AWS services* monitor. If yours does not, choose **Create monitor → AWS services**.
2. **Alert subscriptions → Create subscription.**
   - Frequency: **Daily summaries** (email). Individual, immediate alerts go to an Amazon SNS topic instead.
   - Threshold: an absolute amount such as **$3**.
   - Recipient: your email. Attach the subscription to the monitor.

Anomaly detection is ML-based and needs some spending history. Treat it as a second net under Budgets.

### 2.3 Know where your credits are

- **Billing and Cost Management → Credits** shows the remaining balance and the expiry date.
- **Billing and Cost Management → Bills** (current month, by service and Region) is the fastest way to answer "what is costing me money?". Cost Explorer has more detail but lags by up to a day.

> If you get an unexpected charge anyway: delete the resource, then open a free **Account and billing** support case. AWS often grants a one-time concession for a genuine mistake, but do not count on it.

---

## 3. One Region for everything

Pick **one Region** and run every lab there. That way leftovers are in one place and CloudShell's home directory (which is per Region) holds all your saved state.

- **Suggested: `us-east-1` (N. Virginia).** It has the widest service availability and is usually among the cheapest Regions. Some global features live only there:
  - **ACM certificates for CloudFront must be requested in us-east-1** (Lab 04's optional custom domain).
  - Billing metrics in CloudWatch also appear only in us-east-1.
- **Alternative:** your nearest Region, for lower console latency. Every lab works there, except that the ACM-for-CloudFront step must still use `--region us-east-1`.
- Labs 04 (S3 Cross-Region Replication) and 08 (AWS Backup cross-Region copy) need a **second Region**. This guide uses **`us-west-2`** as `DR_REGION`. Pick any other Region if your main one is us-west-2.

---

## 4. AWS CloudShell conventions

Run every CLI snippet in **AWS CloudShell (bash)**: the `>_` icon in the console's top bar.

- **Nothing to install.** AWS CLI v2, Python 3 with boto3, `jq`, `git` and `zip` are preinstalled. CloudShell uses the credentials of the identity you signed in with, and it sets `AWS_REGION` to the console's current Region.
- **Why not PowerShell on your Windows machine?** The snippets use bash syntax: `\` line continuations, `$VAR`, `$(...)`, heredocs, and single-quoted JSON. PowerShell quotes JSON differently (backtick escapes, `--%`, here-strings), and copy-pasting bash commands into it fails in subtle ways. CloudShell avoids all of that. If you want a local shell later, use WSL or Git Bash with the AWS CLI v2 installed.
- **Storage:** 1 GB persistent `$HOME` **per Region**. Data is deleted after 120 days without use. Idle sessions time out after about 20–30 minutes, but files in `$HOME` survive.
- **Saved state:** labs record resource IDs in `~/saa-lab.env`, which loads automatically in every new session.

Run this **once**, signed in as your Identity Center admin (after Lab 01 step 6), in your lab Region:

```bash
[ -f ~/saa-lab.env ] || cat > ~/saa-lab.env <<'EOF'
# ---- saa-lab CloudShell helpers (sourced from ~/.bashrc) ----
export REGION=${AWS_REGION:-us-east-1}
export DR_REGION=us-west-2          # second Region for Labs 04 and 08 (must differ from REGION)
export ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text 2>/dev/null)

# save NAME VALUE -> exports it now and persists it for future sessions
save() { echo "export $1=\"$2\"" >> ~/saa-lab.env; export "$1=$2"; }

# empty_bucket BUCKET [REGION] -> deletes every object version and delete marker
empty_bucket() {
  python3 - "$1" "${2:-$REGION}" <<'PY'
import sys, boto3
b = boto3.resource("s3", region_name=sys.argv[2]).Bucket(sys.argv[1])
b.object_versions.delete()   # covers versioned and unversioned buckets
print("emptied", sys.argv[1])
PY
}
EOF
grep -q saa-lab.env ~/.bashrc || echo 'source ~/saa-lab.env' >> ~/.bashrc
source ~/saa-lab.env && echo "Region=$REGION DR=$DR_REGION Account=$ACCOUNT_ID"
```

Later labs append more helpers and IDs to this file, for example `save VPC_ID vpc-0abc...`. When you finish all the labs, delete it: `rm ~/saa-lab.env`.

---

## 5. Tag everything and find leftovers

**Tag every resource `project=saa-lab`.** The CLI snippets do this through `--tag-specifications` / `--tags`. In the console, add the tag in the *Tags* section of each create form.

How to find leftovers:

1. **Resource Groups & Tag Editor console → Tag Editor** → Regions: your lab Region **and** `DR_REGION` → Resource types: *All supported resource types* → Tag key `project`, value `saa-lab` → **Search resources**.
2. Save it as a group: **Resource Groups → Create resource group → Tag based → project = saa-lab**. You get a one-click view of everything you have built.
3. CLI, per Region:

   ```bash
   for r in $REGION $DR_REGION; do
     echo "== $r"; aws resourcegroupstaggingapi get-resources --region $r \
       --tag-filters Key=project,Values=saa-lab --query 'ResourceTagMappingList[].ResourceARN' --output text | tr '\t' '\n'
   done
   ```

4. **Billing → Cost allocation tags → User-defined → activate `project`.** After activation (up to 24 h, forward-looking only), Cost Explorer can group spend by `project`. The exam calls this cost allocation tagging.

⚠️ Trap: tag searches only find tagged resources. Some resources are created untagged, for example by a service on your behalf: the ENIs behind an ALB, Lambda log groups, RDS automated snapshots. That is why the **leftover finder** in section 7 queries by service as well.

---

## 6. Cost-risk table

All figures are **approx.**, for us-east-1 on-demand prices at the time of writing, before credits. Prices change and differ by Region, so check each service's pricing page. The goal is to show you which resource hurts if you forget it.

| Lab | Riskiest resource(s) | How it bills | Approx. if forgotten **24 h** | Approx. if forgotten **30 days** |
|---|---|---|---|---|
| [01](lab-01-account-iam.md) | None (IAM, Identity Center, Organizations, SCPs are free) | n/a | $0 | $0 |
| [02](lab-02-custom-vpc.md) | **NAT gateway** + its public IPv4; 2 × t4g.micro | Hourly per NAT GW **plus per-GB processed**; hourly per public IPv4 | ≈ $1.60 (≈ $3 with a NAT GW per AZ) | ≈ $50 (≈ $85 with two) |
| [02](lab-02-custom-vpc.md) alt. | **Interface endpoints** (if you use SSM endpoints instead of NAT) | Hourly **per endpoint per AZ** + per GB | ≈ $1.45 (3 endpoints × 2 AZs) | ≈ $43 |
| [03](lab-03-alb-asg.md) | **ALB** (hourly + LCU + a public IPv4 per AZ node), 2–4 instances, NAT GW | Hourly | ≈ $2.60 | ≈ $75–80 |
| [04](lab-04-s3-cloudfront.md) | **KMS customer managed key**; S3 storage; CloudFront | $1/key-month (prorated) + requests; S3 per GB; CloudFront per GB/request (always-free allowance covers a lab) | < $0.10 | ≈ $1–2 |
| [05](lab-05-rds-ha.md) | **RDS Multi-AZ** (2× instance + 2× storage) + read replica + client EC2 + NAT GW | Hourly per DB instance, GB-month storage | ≈ $2.80 | ≈ $85 |
| [06](lab-06-serverless-decoupled.md) | EventBridge schedule invoking Lambda every 5 min; CloudWatch Logs | Per request / per GB | ≈ $0–0.05 | < $1 |
| [07](lab-07-data-lake.md) | Glue crawler runs; Athena scans; Firehose; **Amazon Quick subscription** (if you sign up) | Per DPU-hour (minimum per run), per TB scanned, per GB; Quick per user-month | ≈ $0–0.20 (excluding Quick) | Quick subscription: check pricing |
| [08](lab-08-iac-backup-dr.md) | AWS Backup recovery points (both Regions), DynamoDB PITR, Route 53 hosted zone + health check | GB-month, cross-Region transfer per GB, $0.50/zone-month, about $0.50/health-check-month | < $0.10 | ≈ $1–3, growing while the plan runs |
| Any | **Public IPv4 address** (Elastic IP, attached or idle; auto-assigned public IP) | ≈ $0.005 per hour each | ≈ $0.12 each | ≈ $3.60 each |

The pattern to remember: **hourly resources hurt**: NAT GW, ALB/NLB, RDS/Aurora, interface endpoints, public IPv4, OpenSearch, Redshift, EKS control plane. **Per-request services barely register** at lab scale: Lambda, SQS, SNS, DynamoDB on-demand, API Gateway, S3. The exam's cost domain (D4) uses the same split.

---

## 7. Universal cleanup checklist

Each lab has its own checklist. Use this one at the end of each study week, or whenever a bill looks wrong. Check **both** `REGION` and `DR_REGION`.

- [ ] **EC2 instances** stopped or running (stopped instances still bill for EBS)
- [ ] **Auto Scaling groups** (delete these first, or they relaunch the instances you terminate)
- [ ] **EBS volumes** in `available` state (detached volumes still bill) and **EBS snapshots**
- [ ] **Elastic IPs**, allocated or idle (all public IPv4 bills hourly)
- [ ] **NAT gateways** (then release their EIPs)
- [ ] **Load balancers** (ALB/NLB) and target groups
- [ ] **RDS instances**, read replicas, **manual snapshots**, RDS Proxy
- [ ] **S3 buckets**: empty versioned buckets with `empty_bucket`, then delete them
- [ ] **CloudFront distributions**: disable, wait until *Deployed*, then delete; also delete OACs
- [ ] **KMS customer managed keys**: schedule deletion (7–30 day waiting period) and delete their aliases
- [ ] **CloudFormation stacks**, and resources a stack *retained* (`DeletionPolicy: Retain`)
- [ ] **CloudWatch log groups** (`/aws/lambda/*`, `/saa-lab/*`) and alarms
- [ ] **VPC endpoints** (interface endpoints bill hourly; gateway endpoints are free)
- [ ] **AWS Backup**: backup plans, **recovery points** in every vault (both Regions), then the vaults
- [ ] Lambda functions, API Gateway APIs, SQS queues, SNS topics, EventBridge rules, DynamoDB tables
- [ ] Glue crawlers and databases, Athena workgroups, Firehose streams, Amazon Quick subscription
- [ ] Route 53 hosted zones and health checks
- [ ] IAM roles and policies named `saa-lab-*` (free, but clutter makes later audits harder)

**Leftover finder.** Paste this into CloudShell. It prints only non-empty results.

```bash
source ~/saa-lab.env
chk() { local label=$1; shift; local out; out=$("$@" 2>/dev/null | tr '\t' ' ');
        [ -n "$out" ] && [ "$out" != "None" ] && printf '  %-18s %s\n' "$label" "$out"; }
for r in $REGION $DR_REGION; do
  echo "=== $r"
  chk instances   aws ec2 describe-instances --region $r --filters Name=instance-state-name,Values=pending,running,stopping,stopped --query 'Reservations[].Instances[].InstanceId' --output text
  chk asgs        aws autoscaling describe-auto-scaling-groups --region $r --query 'AutoScalingGroups[].AutoScalingGroupName' --output text
  chk nat-gws     aws ec2 describe-nat-gateways --region $r --filter Name=state,Values=pending,available --query 'NatGateways[].NatGatewayId' --output text
  chk eips        aws ec2 describe-addresses --region $r --query 'Addresses[].PublicIp' --output text
  chk volumes     aws ec2 describe-volumes --region $r --query 'Volumes[].[VolumeId,State]' --output text
  chk snapshots   aws ec2 describe-snapshots --region $r --owner-ids self --query 'Snapshots[].SnapshotId' --output text
  chk endpoints   aws ec2 describe-vpc-endpoints --region $r --query 'VpcEndpoints[].[VpcEndpointId,VpcEndpointType]' --output text
  chk load-bals   aws elbv2 describe-load-balancers --region $r --query 'LoadBalancers[].LoadBalancerName' --output text
  chk rds         aws rds describe-db-instances --region $r --query 'DBInstances[].DBInstanceIdentifier' --output text
  chk rds-snaps   aws rds describe-db-snapshots --region $r --snapshot-type manual --query 'DBSnapshots[].DBSnapshotIdentifier' --output text
  chk stacks      aws cloudformation list-stacks --region $r --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE ROLLBACK_COMPLETE UPDATE_ROLLBACK_COMPLETE --query 'StackSummaries[].StackName' --output text
  chk kms-aliases aws kms list-aliases --region $r --query "Aliases[?starts_with(AliasName,'alias/saa-lab')].AliasName" --output text
  chk vaults      aws backup list-backup-vaults --region $r --query 'BackupVaultList[?NumberOfRecoveryPoints>`0`].[BackupVaultName,NumberOfRecoveryPoints]' --output text
  chk lambdas     aws lambda list-functions --region $r --query 'Functions[].FunctionName' --output text
  chk tables      aws dynamodb list-tables --region $r --query 'TableNames' --output text
  chk log-groups  aws logs describe-log-groups --region $r --query 'logGroups[].logGroupName' --output text
done
echo "=== global"
chk buckets     aws s3api list-buckets --query 'Buckets[].Name' --output text
chk cloudfront  aws cloudfront list-distributions --query 'DistributionList.Items[].[Id,Status]' --output text
chk hosted-zones aws route53 list-hosted-zones --query 'HostedZones[].Name' --output text
```

Then confirm in **Billing → Bills** the next day that each service you used shows no new charges.

---

## 8. Lab index

Do the labs in order. Labs 03 and 05 reuse Lab 02's VPC. Each lab links to its notes module; read the notes first, then do the lab the same week.

| Lab | What you build | Time | Task statements | Notes |
|---|---|---|---|---|
| [01 Account & IAM](lab-01-account-iam.md) | Identity Center admin, roles + trust policies, permissions boundary, optional SCP + cross-account role | 2 h (+1 h optional) | 1.1 | [01 IAM & multi-account](../notes/01-iam-multi-account.md) |
| [02 Custom VPC](lab-02-custom-vpc.md) | 2-AZ VPC by hand, NAT GW, Session Manager, S3 gateway endpoint, SG vs NACL, Flow Logs | 3 h | 1.2, 2.2, 3.4, 4.4 | [02 Networking & VPC](../notes/02-networking-vpc.md) |
| [03 ALB + Auto Scaling](lab-03-alb-asg.md) | Launch template (IMDSv2), ALB, ASG across AZs, target tracking, self-healing, ELB vs EC2 health checks | 2 h | 2.1, 2.2, 3.2, 4.2 | [03 Compute & containers](../notes/03-compute-containers.md) |
| [04 S3 + CloudFront](lab-04-s3-cloudfront.md) | Versioning, lifecycle, presigned URLs, TLS-only policy, CRR, SSE-KMS + Bucket Key, CloudFront + OAC | 2.5 h | 1.3, 2.2, 3.1, 3.4, 4.1 | [04 Storage](../notes/04-storage.md), [07 Data protection](../notes/07-security-data-protection.md), [08 HA/DR/edge](../notes/08-ha-dr-edge-ops.md) |
| [05 RDS HA](lab-05-rds-ha.md) | RDS PostgreSQL with managed secret, Multi-AZ failover timing, read replica, snapshot restore | 2.5 h (one sitting) | 1.3, 2.2, 3.3, 4.3 | [05 Databases](../notes/05-databases.md), [07 Data protection](../notes/07-security-data-protection.md) |
| [06 Serverless & decoupling](lab-06-serverless-decoupled.md) | HTTP API → Lambda → SQS (+DLQ) → Lambda → DynamoDB, SNS fan-out with filters, EventBridge schedule, X-Ray | 2.5 h | 1.2, 2.1, 3.2 | [06 Decoupling & serverless](../notes/06-decoupling-serverless.md) |
| [07 Data lake](lab-07-data-lake.md) | S3 raw/curated, Glue crawler, Athena, CTAS to partitioned Parquet, optional Firehose / Lake Formation / Quick | 2 h | 1.3, 3.5, 4.1 | [09 Data, analytics & migration](../notes/09-data-analytics-migration.md) |
| [08 IaC, Backup & DR](lab-08-iac-backup-dr.md) | CloudFormation stack, change sets, drift, DeletionPolicy, AWS Backup cross-Region copy + restore, optional Route 53 failover | 2.5 h | 2.2, 4.1 | [08 HA/DR/edge](../notes/08-ha-dr-edge-ops.md), [10 Cost optimization](../notes/10-cost-optimization.md) |

Next: [Lab 01: Account foundations and IAM](lab-01-account-iam.md).
