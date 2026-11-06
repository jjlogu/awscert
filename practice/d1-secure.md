# Domain 1 practice: Design Secure Architectures (15 questions)
> [Home](../README.md) · [Practice index](README.md) · Notes: [IAM & multi-account](../notes/01-iam-multi-account.md) · [Networking & VPC](../notes/02-networking-vpc.md) · [Security & data protection](../notes/07-security-data-protection.md)

Untimed. Read each explanation, including why the wrong options are wrong. Log misses in your error log.

---

### Q1. SCP and identity policy evaluation
A company uses AWS Organizations with all features enabled. The security team detached the default `FullAWSAccess` SCP from the Sandbox OU and attached an SCP that allows only `s3:*`, `ec2:*`, and `cloudwatch:*`. The organization root and the Sandbox member accounts still have `FullAWSAccess` attached. In a Sandbox account, a developer assumes an IAM role whose only identity-based policy allows `dynamodb:*` and `s3:GetObject` on all resources. No permissions boundaries or session policies are in use.

Which API actions can the developer perform successfully?

- A. `s3:GetObject` and all `dynamodb:*` actions, because SCPs restrict only IAM users and the root user, not IAM roles.
- B. All `s3:*`, `ec2:*`, and `cloudwatch:*` actions, because the SCP grants those permissions to every principal in the account.
- C. Only `s3:GetObject`.
- D. None, because an allow-list SCP takes effect only when each role also has a permissions boundary that allows the same actions.

<details>
<summary>Show answer</summary>

**Answer: C**

**Why C:** A request is allowed only if **every** applicable policy type allows it and nothing explicitly denies it. SCPs set the **maximum** permissions for member accounts; they never grant anything. With an allow-list strategy, an action must be allowed by an SCP at **every level** from the root down to the account. Here the OU's SCP allows S3, EC2, and CloudWatch, and the identity policy allows DynamoDB and `s3:GetObject`. The only action in both is `s3:GetObject`.

**Why not the others:**
- **A**: SCPs apply to every IAM user and role in a member account, including that account's root user. They don't apply to the management account or to service-linked roles. DynamoDB isn't allowed by the OU's SCP, so it's denied even though the identity policy allows it.
- **B**: ⚠️ Trap: SCPs are guardrails, not grants. `s3:PutObject` or `ec2:RunInstances` would still need an allow in an identity-based or resource-based policy, and this role has none.
- **D**: Permissions boundaries are optional and independent of SCPs. Without one, the role's effective permissions are simply the intersection of the SCPs and its identity policy.

**Task statement:** 1.1 · **Review:** [IAM & multi-account: policy evaluation and SCPs](../notes/01-iam-multi-account.md)
</details>

---

### Q2. Third-party access to your account
A company is onboarding a SaaS cost-monitoring vendor. The vendor's platform runs in the vendor's own AWS account and needs read-only access to the company's Amazon CloudWatch metrics and AWS Cost Explorer data. The vendor serves thousands of customers from the same platform. The security team wants to avoid long-term credentials and to prevent the vendor's platform from being tricked into using the company's access on behalf of a different customer.

Which solution meets these requirements?

- A. Create an IAM role with a read-only permissions policy. In the role's trust policy, allow the vendor's AWS account as the principal and require an `sts:ExternalId` condition that matches the unique ID the vendor assigned to the company.
- B. Create an IAM user with a read-only permissions policy, generate an access key, and share the key with the vendor through AWS Secrets Manager with automatic rotation every 30 days.
- C. Create an IAM role with a read-only permissions policy and a trust policy that allows the vendor's AWS account as the principal. Send the role ARN to the vendor.
- D. Invite the vendor's AWS account into the company's organization in AWS Organizations, and attach an SCP that allows only read-only actions to that account.

<details>
<summary>Show answer</summary>

**Answer: A**

**Why A:** A **cross-account role** gives the vendor temporary credentials through `sts:AssumeRole`, so no long-term keys leave the company. The **external ID** condition solves the **confused deputy** problem. The vendor assigns each customer a unique ID and always passes the ID of the customer it is acting for. If another customer registers your role ARN with the vendor, the vendor passes *that* customer's ID, the condition fails, and the assume-role call is denied.

**Why not the others:**
- **B**: An access key is a long-term credential that sits in the vendor's systems. Rotating it doesn't change that, and the vendor would still need cross-account access to the secret.
- **C**: This works functionally but is open to the confused deputy problem: anyone who learns the role ARN can ask the vendor to use it. Adding the external ID condition is the fix.
- **D**: Joining your organization doesn't give the vendor access to resources in your accounts, and SCPs never grant permissions. It would also put a third party inside your governance boundary.

**Task statement:** 1.1 · **Review:** [IAM & multi-account: cross-account roles and external ID](../notes/01-iam-multi-account.md)
</details>

---

### Q3. Workforce SSO across many accounts
A company has 60 AWS accounts in AWS Organizations and adds about five accounts each month. Employee identities and group memberships live in a self-managed Microsoft Active Directory in the company's data center, which connects to AWS over AWS Direct Connect. Employees must sign in once with their AD credentials and reach only the accounts and permissions approved for their AD groups. The solution must require the LEAST ongoing operational effort.

Which solution meets these requirements?

- A. Write a script that runs nightly to create an IAM user in every account for each AD user, and add the IAM users to IAM groups that match their AD groups.
- B. Configure SAML 2.0 federation between AD FS and IAM in each account, and create IAM roles in each account that AD groups map to through SAML attributes.
- C. Create an Amazon Cognito user pool federated with AD FS, and grant the user pool's users access to the AWS Management Console of each account.
- D. Enable AWS IAM Identity Center for the organization, connect it to the on-premises AD through AD Connector in AWS Directory Service, create permission sets, and assign AD groups to permission sets in the required accounts.

<details>
<summary>Show answer</summary>

**Answer: D**

**Why D:** **IAM Identity Center** is the managed answer for **workforce** access across an organization. It can use Active Directory as its identity source, either self-managed AD through **AD Connector** (as here) or AWS Managed Microsoft AD. A **permission set** is defined once, and Identity Center creates and maintains the matching IAM role in every account you assign it to. New accounts only need group assignments, not a new IdP setup. AWS recommends running Identity Center administration from a **delegated administrator** account rather than the management account.

**Why not the others:**
- **A**: This creates long-term credentials in 60+ accounts and a sync script you must maintain. Users also get no single sign-on.
- **B**: Per-account SAML federation works, and it was the classic pattern before Identity Center. However, every new account needs its own IdP trust and roles, which is exactly the ongoing effort the company wants to avoid.
- **C**: Cognito is for customer-facing application users (CIAM), not for workforce access to AWS accounts.

**Task statement:** 1.1 · **Review:** [IAM & multi-account: IAM Identity Center and Directory Service](../notes/01-iam-multi-account.md)
</details>

---

### Q4. Governed multi-account landing zone
A startup is moving from one AWS account to a multi-account environment. It needs a central log archive account, a security audit account, single sign-on for employees, preventive and detective guardrails (for example, disallowing changes to logging configuration and detecting unencrypted EBS volumes), and a standard way to create new accounts that follow these rules. The platform team is small and wants the LEAST operational overhead.

Which solution meets these requirements?

- A. Create the accounts with AWS Organizations, write SCPs for the preventive rules, deploy AWS Config rules with AWS CloudFormation StackSets for the detective rules, and configure an organization trail in AWS CloudTrail manually.
- B. Set up an AWS Control Tower landing zone, enable the required preventive and detective controls on the OUs, and provision new accounts with Account Factory.
- C. Publish an account template in AWS Service Catalog and let each team launch new accounts from the catalog.
- D. Deploy AWS Config conformance packs and a CloudTrail trail in each existing account, and give the team a written checklist for configuring new accounts.

<details>
<summary>Show answer</summary>

**Answer: B**

**Why B:** **Control Tower** builds and maintains a **landing zone** for you. It sets up Organizations, the shared **Log Archive** and **Audit** accounts, centralized CloudTrail and Config logging, and IAM Identity Center. It then applies **controls** (formerly called guardrails): **preventive** controls are policy-based (such as SCPs), **detective** controls are Config rules, and proactive controls check CloudFormation resources before they're provisioned. **Account Factory** creates new accounts that are compliant from the start.

**Why not the others:**
- **A**: Every piece works, but you would design, build, and maintain the whole landing zone yourself. That is what Control Tower automates.
- **C**: Account Factory uses Service Catalog under the hood, but Service Catalog alone gives you no log archive, audit account, SSO, or controls.
- **D**: This is per-account and manual, and it drifts as accounts are added. A checklist isn't a guardrail.

**Task statement:** 1.1 · **Review:** [IAM & multi-account: Organizations and Control Tower](../notes/01-iam-multi-account.md)
</details>

---

### Q5. Mobile users uploading directly to S3
A company is building a mobile app. Users must be able to sign up and sign in with an email address and password or with their Google account. After signing in, each user must upload photos directly from the device to their own prefix in an Amazon S3 bucket, without sending the files through the company's backend. The app must not contain any long-term AWS credentials.

Which combination of steps meets these requirements? **(Choose two.)**

- A. Create an IAM user for each app user during sign-up, and store that user's access key in the device's secure storage.
- B. Create an Amazon Cognito user pool for sign-up and sign-in, and add Google as a social identity provider.
- C. Enable AWS IAM Identity Center and add the app's users to a permission set that allows `s3:PutObject` on the bucket.
- D. Create an Amazon Cognito identity pool that accepts the user pool's tokens and issues temporary AWS credentials for an IAM role. Scope the role's policy to each user's prefix with the `${cognito-identity.amazonaws.com:sub}` policy variable.
- E. Embed one IAM user's access key in the app, and have the app generate a presigned URL on the device for each upload.

<details>
<summary>Show answer</summary>

**Answer: B, D**

**Why B and D:** The two Cognito components do different jobs. A **user pool** handles **authentication**: it is the user directory, handles sign-up and sign-in, federates social and SAML/OIDC providers such as Google, and issues JWTs. An **identity pool** handles **authorization to AWS**: it exchanges a trusted token for **temporary STS credentials** for an IAM role. The policy variable restricts each user to `bucket/<their-identity-id>/*`, so users can't overwrite each other's photos.

**Why not the others:**
- **A**: These are long-term keys on millions of devices, and IAM has a per-account user quota. IAM users are for people and systems that administer AWS, not for app end users.
- **C**: IAM Identity Center is for **workforce** access to AWS accounts and business apps, not for consumer app users.
- **E**: ⚠️ Trap: Anyone can extract a key embedded in an app. Presigned URLs are fine when a **backend** signs them, but the requirement rules out both a backend in the path and long-term credentials in the app.

**Task statement:** 1.2 · **Review:** [IAM & multi-account: Amazon Cognito](../notes/01-iam-multi-account.md)
</details>

---

### Q6. Blocking a malicious IP range
A company runs a custom TCP service on port 7000 on EC2 instances in a public subnet. The instances have Elastic IP addresses, and their security group allows port 7000 from `0.0.0.0/0` because customers connect from anywhere. The security team has identified a CIDR block from which an attacker is scanning and flooding the service. The team must block that CIDR block for every instance in the subnet immediately, without affecting other clients.

Which solution meets these requirements?

- A. Add an inbound rule to the subnet's network ACL that denies all traffic from the CIDR block, with a lower rule number than the rule that allows port 7000.
- B. Add an inbound rule to the instances' security group that denies all traffic from the CIDR block.
- C. Remove the `0.0.0.0/0` rule from the security group and add an allow rule for each known customer CIDR block.
- D. Create an AWS WAF IP set that contains the CIDR block, and associate a web ACL that blocks the IP set with the EC2 instances.

<details>
<summary>Show answer</summary>

**Answer: A**

**Why A:** **Network ACLs** support both **allow and deny** rules, apply to the whole subnet, and are evaluated **in rule-number order, lowest first, until the first match**. A deny rule for the attacker's CIDR at, say, rule 90 drops those packets before the allow rule at 100 is evaluated. NACLs are stateless, but here that doesn't matter: the attacker's inbound packets are dropped on arrival.

**Why not the others:**
- **B**: ⚠️ Trap: Security groups have **allow rules only**. You can't write a deny rule.
- **C**: The customers connect from anywhere, so there's no complete allow list to write. Legitimate clients would be locked out.
- **D**: AWS WAF can't be associated with EC2 instances. It attaches to resources such as CloudFront, ALB, API Gateway, and AppSync, and it inspects HTTP(S), not a custom TCP protocol. For stateful L3/L4 filtering at scale, AWS Network Firewall is the managed option.

**Task statement:** 1.2 · **Review:** [Networking & VPC: security groups vs network ACLs](../notes/02-networking-vpc.md)
</details>

---

### Q7. Private access to S3 from private subnets
EC2 instances in private subnets read and write large volumes of data in Amazon S3 buckets in the same AWS Region. Today the traffic flows through a NAT gateway. A new security requirement states that the instances must have no route to the internet at all. The company also wants to keep costs as low as possible.

Which solution meets these requirements?

- A. Keep the NAT gateway, and add a bucket policy that allows requests only from the NAT gateway's Elastic IP address.
- B. Create an interface VPC endpoint for Amazon S3 in each private subnet's Availability Zone, enable private DNS, and remove the route to the NAT gateway.
- C. Create a gateway VPC endpoint for Amazon S3, associate it with the private subnets' route tables, and remove the route to the NAT gateway.
- D. Move the instances to public subnets, and restrict their security groups' outbound rules to the S3 managed prefix list.

<details>
<summary>Show answer</summary>

**Answer: C**

**Why C:** A **gateway endpoint** adds a route for the S3 prefix list to the route tables you choose. Traffic to S3 stays on the AWS network, and the endpoint has **no hourly or per-GB charge**. With the NAT route removed, the instances have no internet path. You can tighten it further with an endpoint policy and with an `aws:SourceVpce` condition in the bucket policy.

**Why not the others:**
- **A**: The NAT gateway is still there, so the requirement of no internet route is broken.
- **B**: An interface endpoint also keeps traffic private, but it is billed per hour per AZ and per GB processed. Choose it when you need S3 access from on premises, from peered VPCs, or from another Region, because gateway endpoints don't support those paths. Here the gateway endpoint is cheaper.
- **D**: A public subnet has a route to an internet gateway, which is the opposite of the requirement. Public IPv4 addresses are also billed hourly.

**Task statement:** 1.2 · **Review:** [Networking & VPC: VPC endpoints](../notes/02-networking-vpc.md)
</details>

---

### Q8. Locking a bucket to one VPC endpoint
A financial services company stores sensitive reports in an Amazon S3 bucket. Only an application on EC2 instances in one VPC may read or write the objects. The VPC has an S3 gateway endpoint. Compliance requires that any request that doesn't arrive through that endpoint is rejected, even if the caller has valid IAM credentials with S3 permissions.

Which solution meets this requirement?

- A. Attach an endpoint policy to the gateway endpoint that allows `s3:GetObject` and `s3:PutObject` only on the reports bucket.
- B. Add a bucket policy statement that allows access only when the `aws:SourceIp` condition key matches the VPC's private CIDR range.
- C. Turn on S3 Block Public Access for the bucket, and configure the instances' security groups to allow outbound traffic only to the S3 managed prefix list.
- D. Add a bucket policy statement that denies all `s3:*` actions on the bucket and its objects when the `aws:SourceVpce` condition key doesn't equal the gateway endpoint's ID.

<details>
<summary>Show answer</summary>

**Answer: D**

**Why D:** The bucket policy is evaluated for every request to the bucket, wherever it comes from. An **explicit Deny** with `StringNotEquals: aws:SourceVpce` overrides any Allow in any identity policy, so requests from the internet, the console, or other VPCs fail. ⚠️ Trap: The same statement also locks out administrators working in the console. Real policies usually add a narrow exception, such as an `aws:PrincipalArn` condition for a break-glass role.

**Why not the others:**
- **A**: An endpoint policy limits what can be reached **through the endpoint**. It does nothing about a caller who reaches the bucket from somewhere else.
- **B**: Requests through a VPC endpoint carry private source IPs, and `aws:SourceIp` doesn't match those (that is what `aws:VpcSourceIp` is for). Also, for a principal in the same account, an Allow in *either* the identity policy or the bucket policy is enough. An Allow-only bucket policy therefore can't restrict such a principal; only a Deny can.
- **C**: Block Public Access stops anonymous and public-policy access, not authenticated IAM principals. Security group rules restrict the instances, not other callers.

**Task statement:** 1.2 · **Review:** [Networking & VPC: endpoint policies and `aws:SourceVpce`](../notes/02-networking-vpc.md)
</details>

---

### Q9. Cross-account reads of SSE-KMS objects
Account A owns an Amazon S3 bucket whose objects are encrypted with SSE-KMS using a customer managed key in account A. An analytics team in account B reads the objects through an IAM role. The bucket policy already grants that role `s3:GetObject`, and the role's identity policy allows `s3:GetObject` on the bucket. Reads still fail with an `AccessDenied` error. Security policy requires that the objects stay encrypted with the existing customer managed key.

What should a solutions architect do to allow the reads?

- A. Convert the key to a multi-Region key and replicate it into account B, so that the role can use a local copy of the key.
- B. Add `kms:Decrypt` to the bucket policy statement that grants account B's role access to the bucket.
- C. Update the key policy in account A to allow account B to use the key for `kms:Decrypt`, and add a statement to the role's identity policy in account B that allows `kms:Decrypt` on the key's ARN.
- D. Add a statement to the role's identity policy in account B that allows `kms:*` on the key's ARN, and make no changes in account A.

<details>
<summary>Show answer</summary>

**Answer: C**

**Why C:** Reading an SSE-KMS object requires S3 permission **and** permission to call `kms:Decrypt` on the key. Using a KMS key across accounts takes **two** grants. The **key policy** in the owning account must allow the external account or role, and an **IAM policy** in the external account must allow the role to use the key. Also note that you can't share objects encrypted with the AWS managed key `aws/s3` this way, because you can't edit its key policy. Cross-account sharing needs a customer managed key.

**Why not the others:**
- **A**: Multi-Region keys are copies of a key in **other Regions of the same account**. They don't move a key into another account.
- **B**: A bucket policy controls S3 actions on that bucket. KMS permissions belong in the key policy, IAM policies, or grants.
- **D**: An IAM policy in account B can only pass on access that the key policy in account A already allows. Without the key policy change, the call is still denied.

**Task statement:** 1.3 · **Review:** [Security & data protection: cross-account KMS key use](../notes/07-security-data-protection.md)
</details>

---

### Q10. Rotating database credentials
An application on EC2 instances connects to an Amazon RDS for PostgreSQL database with a username and password stored in a configuration file on each instance. A new security standard requires that the password is rotated automatically every 30 days and that no credentials are stored on the instances. The DBA team requires the application to keep using a native PostgreSQL database user with a password.

Which solution meets these requirements with the LEAST operational overhead?

- A. Store the credentials in AWS Secrets Manager and turn on automatic rotation every 30 days with the AWS-provided rotation function for RDS for PostgreSQL. Change the application to retrieve the secret at runtime by using its instance profile role.
- B. Store the credentials as a SecureString parameter in AWS Systems Manager Parameter Store, and turn on the parameter's built-in automatic rotation every 30 days.
- C. Turn on IAM database authentication for the DB instance, and change the application to generate an authentication token for each connection.
- D. Encrypt the password with a customer managed AWS KMS key, store the ciphertext in the configuration file, and turn on automatic rotation for the KMS key.

<details>
<summary>Show answer</summary>

**Answer: A**

**Why A:** **Secrets Manager** has built-in rotation for RDS. The rotation function changes the password in the database and updates the secret, with staged versions (`AWSCURRENT`/`AWSPENDING`) so running clients aren't cut off. The application uses its **instance profile** to call `GetSecretValue`, ideally through a caching client, so nothing sensitive is stored on disk. For an RDS master user, RDS can also manage the password in Secrets Manager for you.

**Why not the others:**
- **B**: ⚠️ Trap: Parameter Store has **no built-in rotation**. You would have to write and schedule your own rotation Lambda function.
- **C**: IAM database authentication is a good way to avoid passwords, but the DBA requirement rules it out: the application must keep using a native database user with a password.
- **D**: KMS key rotation changes the key material, not the database password. The ciphertext also stays on the instances.

**Task statement:** 1.2 · **Review:** [Security & data protection: Secrets Manager vs Parameter Store](../notes/07-security-data-protection.md)
</details>

---

### Q11. Blocking SQL injection
A company runs a public web application on EC2 instances behind an Application Load Balancer. Application logs show repeated SQL injection attempts in query strings and form fields. The development team needs several weeks to fix the input validation in the code. The company wants to block these requests before they reach the instances, starting this week, with the LEAST operational overhead.

Which solution meets these requirements?

- A. Add inbound rules to the network ACLs of the ALB's subnets that deny requests containing SQL keywords.
- B. Enable Amazon GuardDuty and configure it to block the source IP addresses of SQL injection attempts.
- C. Subscribe to AWS Shield Advanced and add the ALB as a protected resource, so that SQL injection attempts are blocked automatically.
- D. Create an AWS WAF web ACL that includes the AWS managed rule group for SQL injection (the SQL database rule group), and associate the web ACL with the ALB.

<details>
<summary>Show answer</summary>

**Answer: D**

**Why D:** **AWS WAF** inspects Layer 7 HTTP(S) requests, including query strings, bodies, headers, and cookies. **AWS managed rule groups** give you maintained SQLi signatures without writing rules yourself. A web ACL can be associated directly with an ALB. A good practice is to deploy new rules in **Count** mode first to check for false positives, then switch them to **Block**.

**Why not the others:**
- **A**: NACLs work at Layers 3 and 4 (IP addresses, ports, protocols). They can't inspect request content.
- **B**: GuardDuty **detects** threats from logs such as CloudTrail, VPC Flow Logs, and DNS logs. It doesn't block traffic and doesn't inspect HTTP payloads for SQL injection.
- **C**: Shield Advanced is DDoS protection. It includes AWS WAF at no extra charge for protected resources, but you still have to configure SQLi rules. It doesn't add them automatically.

**Task statement:** 1.2 · **Review:** [Security & data protection: AWS WAF](../notes/07-security-data-protection.md)
</details>

---

### Q12. DDoS response team and cost protection
A media company serves a popular website through Amazon CloudFront with an Application Load Balancer origin, and uses Amazon Route 53 for DNS. A recent DDoS attack made the Auto Scaling group behind the ALB scale out sharply, which produced an unexpected bill. Leadership now requires 24/7 access to AWS DDoS experts during attacks, financial protection against scaling charges caused by DDoS attacks, and detailed attack diagnostics.

Which solution meets these requirements?

- A. Rely on AWS Shield Standard, which is already enabled for CloudFront and Route 53, and open a support case during the next attack.
- B. Subscribe to AWS Shield Advanced, and add the CloudFront distribution, the ALB, and the Route 53 hosted zone as protected resources.
- C. Use AWS Firewall Manager to deploy AWS WAF rate-based rules to the CloudFront distribution and the ALB in all accounts.
- D. Enable Amazon GuardDuty and Amazon Detective to detect DDoS attacks and investigate where they come from.

<details>
<summary>Show answer</summary>

**Answer: B**

**Why B:** **Shield Advanced** adds what Shield Standard doesn't have: access to the **Shield Response Team (SRT)** (with an eligible AWS Support plan), optional **proactive engagement**, **DDoS cost protection** (credits for scaling charges that a DDoS attack causes on protected resources), detailed attack metrics and diagnostics, and AWS WAF at no extra cost for protected resources. Protected resource types include CloudFront distributions, Route 53 hosted zones, Global Accelerator accelerators, load balancers, and Elastic IPs. It has a monthly fee and a 1-year commitment.

**Why not the others:**
- **A**: Shield Standard is free and automatic, and it protects against common L3/L4 attacks. It includes no SRT access and no cost protection.
- **C**: Rate-based rules help against HTTP floods and are a good addition, but they provide neither the SRT nor cost protection. Firewall Manager is how you manage WAF (and Shield Advanced) policies centrally, not a replacement for either.
- **D**: GuardDuty detects threats and Detective helps investigate them. Neither one mitigates a DDoS attack or covers its costs.

**Task statement:** 1.2 · **Review:** [Security & data protection: AWS Shield](../notes/07-security-data-protection.md)
</details>

---

### Q13. Finding PII in S3 at scale
A healthcare company has more than 400 Amazon S3 buckets across 30 AWS accounts in AWS Organizations. An auditor asks the company to identify which buckets contain personally identifiable information (PII), such as names, addresses, and passport numbers, and to keep doing so as new data arrives. The compliance team wants the results in one place and doesn't want to build or maintain any scanning code.

Which solution meets these requirements?

- A. Enable Amazon GuardDuty S3 Protection in every account, and review the findings in the GuardDuty administrator account.
- B. Enable Amazon Macie for the organization with a delegated administrator account, turn on automated sensitive data discovery, and send the findings to AWS Security Hub.
- C. Use AWS Glue crawlers to catalog every bucket, and run Amazon Athena queries with regular expressions that match PII patterns.
- D. Enable Amazon Inspector in every account, and configure it to scan the S3 buckets for sensitive data.

<details>
<summary>Show answer</summary>

**Answer: B**

**Why B:** **Macie** discovers sensitive data in S3 by using managed data identifiers (and custom ones if you need them) together with machine learning. With a **delegated administrator** in Organizations, it covers all member accounts centrally. **Automated sensitive data discovery** keeps sampling buckets as data changes. Macie also reports bucket posture, such as public access or missing encryption. Findings go to EventBridge and Security Hub.

**Why not the others:**
- **A**: GuardDuty S3 Protection watches S3 **access activity** for threats, such as unusual API calls or data exfiltration patterns. It doesn't classify object contents.
- **C**: This could work for some structured files, but it is custom code and regular expressions that you have to maintain, plus scanning costs. It's the opposite of "no scanning code".
- **D**: Inspector scans EC2 instances, container images in ECR, and Lambda functions for vulnerabilities. It doesn't scan S3 contents.

**Task statement:** 1.3 · **Review:** [Security & data protection: threat detection and data classification](../notes/07-security-data-protection.md)
</details>

---

### Q14. WORM retention for regulated records
A broker-dealer must keep trade confirmation records in Amazon S3 for 7 years in a non-rewritable, non-erasable format. During the retention period, no user, including administrators and the AWS account root user, may delete or overwrite the protected records or shorten the retention period.

Which solution meets these requirements?

- A. Enable S3 Object Lock on the bucket, and configure a default retention period of 7 years in compliance mode.
- B. Enable S3 Object Lock on the bucket, and configure a default retention period of 7 years in governance mode.
- C. Enable S3 Versioning and MFA Delete, and add a bucket policy that denies `s3:DeleteObject` for all principals.
- D. Add a lifecycle rule that transitions objects to S3 Glacier Deep Archive immediately, because objects in that storage class can't be deleted before the minimum storage duration ends.

<details>
<summary>Show answer</summary>

**Answer: A**

**Why A:** In **compliance mode**, no one, **including the root user**, can delete or overwrite a protected object version or shorten its retention until the retention period expires. Object Lock works only on versioned buckets (on an existing bucket, enable versioning first), and it can be enabled on new or existing buckets. A plain DELETE request still succeeds by adding a delete marker, but the locked version underneath stays intact. A **legal hold** is a separate flag with no expiry date.

**Why not the others:**
- **B**: In governance mode, principals with `s3:BypassGovernanceRetention` can delete versions or shorten retention. Use it to protect against accidents, not to meet regulatory WORM requirements.
- **C**: An administrator or the root user can edit the bucket policy, and the root user can disable MFA Delete. Neither control is immutable.
- **D**: ⚠️ Trap: You **can** delete objects in Deep Archive at any time. The 180-day minimum only means you pay a prorated early-deletion charge.

**Task statement:** 1.3 · **Review:** [Security & data protection: immutability (WORM)](../notes/07-security-data-protection.md)
</details>

---

### Q15. Certificates for CloudFront and an ALB origin
A company serves `www.example.com` through an Amazon CloudFront distribution. The origin is an Application Load Balancer in the eu-west-1 Region, reached at `origin.example.com`. The company requires HTTPS between viewers and CloudFront and between CloudFront and the ALB. The certificates must be publicly trusted and renew automatically. DNS is hosted in Amazon Route 53.

Which solution meets these requirements with the LEAST operational overhead?

- A. Request one public certificate in AWS Certificate Manager (ACM) in eu-west-1 that covers both names, and associate it with both the CloudFront distribution and the ALB.
- B. Issue certificates for both names from AWS Private CA, and associate them with the CloudFront distribution and the ALB.
- C. Request a public ACM certificate for `www.example.com` in us-east-1 for the CloudFront distribution, and a public ACM certificate for `origin.example.com` in eu-west-1 for the ALB. Use DNS validation with records in Route 53.
- D. Request one public ACM certificate in us-east-1 that covers both names, and associate it with both the CloudFront distribution and the ALB.

<details>
<summary>Show answer</summary>

**Answer: C**

**Why C:** ACM certificates are **Regional resources**. **CloudFront only uses ACM certificates from us-east-1**, and an ALB only uses certificates from **its own Region**. So you need one certificate in each Region. With **DNS validation**, ACM renews the certificates automatically for as long as the validation CNAME records stay in place.

**Why not the others:**
- **A**: CloudFront can't use a certificate from eu-west-1.
- **B**: Private CA certificates aren't trusted by public browsers. They're meant for internal services and mTLS, and Private CA also has its own cost.
- **D**: An ALB in eu-west-1 can't use a certificate from us-east-1.

**Task statement:** 1.3 · **Review:** [Security & data protection: ACM and certificates](../notes/07-security-data-protection.md)
</details>

---

## Answer summary

<details>
<summary>Show all answers</summary>

| Q | Answer | Topic | Task |
|---|---|---|---|
| 1 | C | SCP and identity policy evaluation | 1.1 |
| 2 | A | Cross-account role with external ID | 1.1 |
| 3 | D | IAM Identity Center with on-premises AD | 1.1 |
| 4 | B | Control Tower landing zone and controls | 1.1 |
| 5 | B, D | Cognito user pool + identity pool | 1.2 |
| 6 | A | NACL deny for an IP range | 1.2 |
| 7 | C | S3 gateway endpoint, no internet route | 1.2 |
| 8 | D | Bucket policy with `aws:SourceVpce` | 1.2 |
| 9 | C | KMS key policy for cross-account decryption | 1.3 |
| 10 | A | Secrets Manager rotation for RDS | 1.2 |
| 11 | D | AWS WAF managed SQLi rules on an ALB | 1.2 |
| 12 | B | Shield Advanced (SRT, cost protection) | 1.2 |
| 13 | B | Macie for PII discovery | 1.3 |
| 14 | A | S3 Object Lock compliance mode | 1.3 |
| 15 | C | ACM in us-east-1 for CloudFront, Regional for ALB | 1.3 |

</details>

**Scoring:** 13+/15 strong · 10–12 review misses · <10 re-read the module
