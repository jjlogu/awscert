# Lab 02: Build a 2-AZ VPC by hand

> [Home](../README.md) · [Labs index](README.md) · Notes: [Networking & VPC](../notes/02-networking-vpc.md) · Practice: [D1 Secure](../practice/d1-secure.md), [D4 Cost-optimized](../practice/d4-cost-optimized.md)

| Time | Cost risk | Task statements | Prereqs |
|---|---|---|---|
| 3 h | ⚠️ **NAT gateway**: hourly + per GB, plus its public IPv4. About $1.60/day if forgotten (2 tiny instances included) | 1.2, 2.2, 3.4, 4.4 | [Lab safety](README.md) setup; [Lab 01](lab-01-account-iam.md) admin in CloudShell |

## Objective

Build the classic exam VPC piece by piece instead of with the wizard, so you know what each piece does:

- public and private subnets in two AZs
- an internet gateway and a NAT gateway
- separate route tables

Then use it:

- reach a private instance with **Session Manager** (no bastion host, no SSH keys, no inbound ports)
- watch S3 traffic move from the NAT gateway to a free **gateway endpoint**
- break connectivity with a **NACL** to see stateless filtering and ephemeral ports
- find the evidence in **VPC Flow Logs**

Labs 03 and 05 reuse this VPC.

## Architecture

```
                                 Internet
                                    │
                             ┌──────┴──────┐
                             │     IGW     │
┌────────────────────────────┴─────────────┴──────────── VPC 10.0.0.0/16 ──┐
│            AZ1                                       AZ2                  │
│  ┌── public-a 10.0.0.0/24 ──┐              ┌── public-b 10.0.1.0/24 ──┐   │
│  │  NAT GW + Elastic IP     │              │  (2nd NAT GW for HA)     │   │
│  └────────────▲─────────────┘              └──────────────────────────┘   │
│     public RT: 0.0.0.0/0 → IGW (both public subnets)                      │
│               │                                                           │
│  ┌── private-a 10.0.10.0/24 ┐   HTTP/ICMP  ┌── private-b 10.0.11.0/24 ┐   │
│  │  saa-lab-client (SSM)    │ ───────────▶ │  saa-lab-web             │   │
│  │  SG: no inbound          │              │  SG: 80+ICMP from client │   │
│  └──────────────────────────┘              │  custom NACL (step 13)   │   │
│                                            └──────────────────────────┘   │
│     private RT: 0.0.0.0/0 → NAT GW  ·  S3 prefix list → gateway endpoint  │
│     Flow Logs (VPC, ALL) → CloudWatch Logs /saa-lab/vpc-flow              │
└───────────────────────────────────────────────────────────────────────────┘
```

## Steps

All CLI runs in CloudShell unless a block says **on the client instance**. Each step also names the console path so you can compare. The console's resource map (**VPC console → Your VPCs → select → Resource map**) is a good way to check your work as you go.

1. **Pick two AZs and create the VPC.** Console: **VPC console → Your VPCs → Create VPC → VPC only**, CIDR `10.0.0.0/16`. Then **Actions → Edit VPC settings → Enable DNS hostnames**.

   ```bash
   source ~/saa-lab.env
   cat >> ~/saa-lab.env <<'EOF'
   # ts RESOURCE_TYPE NAME -> tag specification with Name + project=saa-lab
   ts() { echo "ResourceType=$1,Tags=[{Key=Name,Value=$2},{Key=project,Value=saa-lab}]"; }
   EOF
   source ~/saa-lab.env

   # Two AZs that offer t4g.micro (not every AZ offers every instance type)
   read -r a1 a2 _ <<< "$(aws ec2 describe-instance-type-offerings --location-type availability-zone \
     --filters Name=instance-type,Values=t4g.micro --query 'sort(InstanceTypeOfferings[].Location)' --output text)"
   save AZ1 "$a1"; save AZ2 "$a2"; echo "$AZ1 $AZ2"

   save VPC_ID "$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 \
     --tag-specifications "$(ts vpc saa-lab-vpc)" --query Vpc.VpcId --output text)"
   aws ec2 modify-vpc-attribute --vpc-id "$VPC_ID" --enable-dns-hostnames '{"Value":true}'
   ```

2. **Create four subnets.** Console: **VPC console → Subnets → Create subnet**.

   ```bash
   mk_subnet() { aws ec2 create-subnet --vpc-id "$VPC_ID" --cidr-block "$2" --availability-zone "$3" \
     --tag-specifications "$(ts subnet "$1")" --query Subnet.SubnetId --output text; }
   save PUB_A  "$(mk_subnet saa-lab-public-a  10.0.0.0/24  "$AZ1")"
   save PUB_B  "$(mk_subnet saa-lab-public-b  10.0.1.0/24  "$AZ2")"
   save PRIV_A "$(mk_subnet saa-lab-private-a 10.0.10.0/24 "$AZ1")"
   save PRIV_B "$(mk_subnet saa-lab-private-b 10.0.11.0/24 "$AZ2")"
   aws ec2 describe-subnets --subnet-ids $PUB_A $PRIV_A \
     --query 'Subnets[].[Tags[?Key==`Name`]|[0].Value,CidrBlock,AvailableIpAddressCount]' --output table
   ```

   The subnets show **251** available addresses, not 256. AWS reserves 5 in every subnet: network address, VPC router, DNS, one reserved for future use, and broadcast. Remember this when sizing small subnets.

3. **Internet gateway and the public route table.** Console: **Internet gateways → Create → Actions → Attach to VPC**, then **Route tables → Create route table**, **Edit routes**, and **Subnet associations**.

   ```bash
   save IGW_ID "$(aws ec2 create-internet-gateway --tag-specifications "$(ts internet-gateway saa-lab-igw)" \
     --query InternetGateway.InternetGatewayId --output text)"
   aws ec2 attach-internet-gateway --internet-gateway-id "$IGW_ID" --vpc-id "$VPC_ID"

   save PUB_RT "$(aws ec2 create-route-table --vpc-id "$VPC_ID" \
     --tag-specifications "$(ts route-table saa-lab-public-rt)" --query RouteTable.RouteTableId --output text)"
   aws ec2 create-route --route-table-id "$PUB_RT" --destination-cidr-block 0.0.0.0/0 --gateway-id "$IGW_ID"
   aws ec2 associate-route-table --route-table-id "$PUB_RT" --subnet-id "$PUB_A" --output text
   aws ec2 associate-route-table --route-table-id "$PUB_RT" --subnet-id "$PUB_B" --output text
   ```

   A subnet is "public" **only** because its route table sends `0.0.0.0/0` to an IGW. An instance there also needs a public IPv4 address (auto-assigned or an Elastic IP) to be reachable from the internet. Both kinds of address bill hourly.

4. **Private route table and a NAT gateway.** ⚠️ The NAT gateway's billing clock starts here. Console: **NAT gateways → Create NAT gateway**: subnet `public-a`, connectivity *Public*, **Allocate Elastic IP**. Then add `0.0.0.0/0 → nat-...` to the private route table.

   ```bash
   save PRIV_RT "$(aws ec2 create-route-table --vpc-id "$VPC_ID" \
     --tag-specifications "$(ts route-table saa-lab-private-rt)" --query RouteTable.RouteTableId --output text)"
   aws ec2 associate-route-table --route-table-id "$PRIV_RT" --subnet-id "$PRIV_A" --output text
   aws ec2 associate-route-table --route-table-id "$PRIV_RT" --subnet-id "$PRIV_B" --output text

   # Reusable helpers: Labs 03 and 05 call nat_up / nat_down
   cat >> ~/saa-lab.env <<'EOF'
   nat_up() {
     save EIP_ALLOC "$(aws ec2 allocate-address --domain vpc --tag-specifications "$(ts elastic-ip saa-lab-nat-a)" \
       --query AllocationId --output text)"
     save NAT_ID "$(aws ec2 create-nat-gateway --subnet-id "$PUB_A" --allocation-id "$EIP_ALLOC" \
       --tag-specifications "$(ts natgateway saa-lab-nat-a)" --query NatGateway.NatGatewayId --output text)"
     echo "waiting for $NAT_ID ..."; aws ec2 wait nat-gateway-available --nat-gateway-ids "$NAT_ID"
     aws ec2 replace-route --route-table-id "$PRIV_RT" --destination-cidr-block 0.0.0.0/0 --nat-gateway-id "$NAT_ID" 2>/dev/null \
       || aws ec2 create-route --route-table-id "$PRIV_RT" --destination-cidr-block 0.0.0.0/0 --nat-gateway-id "$NAT_ID" >/dev/null
     echo "NAT $NAT_ID is up (billing started)"
   }
   nat_down() {
     aws ec2 delete-nat-gateway --nat-gateway-id "$NAT_ID" >/dev/null
     until [ "$(aws ec2 describe-nat-gateways --nat-gateway-ids "$NAT_ID" --query 'NatGateways[0].State' --output text)" = deleted ]; do sleep 15; done
     aws ec2 release-address --allocation-id "$EIP_ALLOC" && echo "NAT $NAT_ID deleted, EIP released"
   }
   EOF
   source ~/saa-lab.env
   nat_up      # 1-3 minutes
   ```

   One NAT gateway in AZ1 serves both private subnets. That is cheap, but it is a **single-AZ dependency**, and traffic from private-b crosses AZs (cross-AZ data charges). Step 15 shows the HA version.

5. **Instance role for Session Manager.** Console: **IAM → Roles → Create role → AWS service → EC2 → `AmazonSSMManagedInstanceCore`**. The console creates the instance profile for you; the CLI does not.

   ```bash
   cat > ec2-trust.json <<'EOF'
   {"Version": "2012-10-17", "Statement": [{"Effect": "Allow", "Principal": {"Service": "ec2.amazonaws.com"}, "Action": "sts:AssumeRole"}]}
   EOF
   aws iam create-role --role-name saa-lab-ec2-ssm --assume-role-policy-document file://ec2-trust.json \
     --tags Key=project,Value=saa-lab --query Role.Arn --output text
   aws iam attach-role-policy --role-name saa-lab-ec2-ssm \
     --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
   aws iam create-instance-profile --instance-profile-name saa-lab-ec2-ssm --query InstanceProfile.Arn --output text
   aws iam add-role-to-instance-profile --instance-profile-name saa-lab-ec2-ssm --role-name saa-lab-ec2-ssm
   ```

6. **Security groups.** The client has **no inbound rules at all**. The web server accepts HTTP and ICMP only from members of the client SG, so the rule references a security group, not a CIDR.

   ```bash
   save CLIENT_SG "$(aws ec2 create-security-group --vpc-id "$VPC_ID" --group-name saa-lab-client-sg \
     --description "client: no inbound" --tag-specifications "$(ts security-group saa-lab-client-sg)" \
     --query GroupId --output text)"
   save WEB_SG "$(aws ec2 create-security-group --vpc-id "$VPC_ID" --group-name saa-lab-web-sg \
     --description "web: HTTP and ICMP from client SG" --tag-specifications "$(ts security-group saa-lab-web-sg)" \
     --query GroupId --output text)"
   aws ec2 authorize-security-group-ingress --group-id "$WEB_SG" --protocol tcp  --port 80 --source-group "$CLIENT_SG" --output text
   aws ec2 authorize-security-group-ingress --group-id "$WEB_SG" --protocol icmp --port -1 --source-group "$CLIENT_SG" --output text
   ```

7. **Launch two private instances.** They have no key pair, no public IP and IMDSv2 only. The web server uses Python's built-in HTTP server, so it needs no packages.

   ```bash
   cat > web-userdata.sh <<'EOF'
   #!/bin/bash
   mkdir -p /srv/www && echo "hello from $(hostname)" > /srv/www/index.html
   cat > /etc/systemd/system/web.service <<'UNIT'
   [Unit]
   Description=Lab web server
   After=network-online.target
   [Service]
   ExecStart=/usr/bin/python3 -m http.server 80 --directory /srv/www
   Restart=always
   [Install]
   WantedBy=multi-user.target
   UNIT
   systemctl daemon-reload && systemctl enable --now web.service
   EOF

   AMI=resolve:ssm:/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-arm64
   launch() {  # name subnet sg [userdata-file]
     aws ec2 run-instances --image-id "$AMI" --instance-type t4g.micro \
       --subnet-id "$2" --security-group-ids "$3" --iam-instance-profile Name=saa-lab-ec2-ssm \
       --metadata-options HttpTokens=required,HttpEndpoint=enabled \
       ${4:+--user-data file://$4} \
       --tag-specifications "$(ts instance "$1")" "$(ts volume "$1")" \
       --query 'Instances[0].InstanceId' --output text
   }
   sleep 10   # let the new instance profile propagate
   save CLIENT_ID "$(launch saa-lab-client "$PRIV_A" "$CLIENT_SG")"
   save WEB_ID    "$(launch saa-lab-web    "$PRIV_B" "$WEB_SG" web-userdata.sh)"
   aws ec2 wait instance-running --instance-ids "$CLIENT_ID" "$WEB_ID"
   save WEB_IP "$(aws ec2 describe-instances --instance-ids "$WEB_ID" \
     --query 'Reservations[0].Instances[0].PrivateIpAddress' --output text)"; echo "web = $WEB_IP"
   ```

   If `run-instances` says the instance profile is invalid, wait 10 seconds and run it again. IAM changes take a few seconds to propagate.

8. **Connect with Session Manager and test outbound internet through the NAT.** Wait 2–3 minutes for the SSM agent to register:

   ```bash
   aws ssm describe-instance-information --query 'InstanceInformationList[].[InstanceId,PingStatus]' --output table
   aws ec2 describe-addresses --allocation-ids "$EIP_ALLOC" --query 'Addresses[0].PublicIp' --output text   # NAT's EIP
   ```

   **EC2 console → Instances → saa-lab-client → Connect → Session Manager → Connect.** In the browser shell that opens, type `bash`, then run these **on the client instance**:

   ```bash
   curl -s https://checkip.amazonaws.com          # prints the NAT gateway's Elastic IP, not the instance's
   TOKEN=$(curl -sX PUT http://169.254.169.254/latest/api/token -H 'X-aws-ec2-metadata-token-ttl-seconds: 300')
   curl -s -o /dev/null -w '%{http_code}\n' -H "X-aws-ec2-metadata-token: $TOKEN" \
     http://169.254.169.254/latest/meta-data/public-ipv4     # 404: the instance has no public IP
   cat /proc/sys/net/ipv4/ip_local_port_range              # 32768 60999: Linux's ephemeral ports (step 13)
   ```

   No inbound rule, no SSH key and no bastion were involved. The SSM agent made an **outbound** HTTPS connection to Systems Manager through the NAT gateway, and your session rides on that connection. Sessions are logged in CloudTrail and can also be logged to S3 or CloudWatch Logs.

9. **Turn on VPC Flow Logs** now, so the experiments below get recorded. Console: **Your VPCs → select → Flow logs → Create flow log**: filter *All*, interval *1 minute*, destination *CloudWatch Logs*, plus a log group and an IAM role.

   ```bash
   cat > flowlogs-trust.json <<'EOF'
   {"Version": "2012-10-17", "Statement": [{"Effect": "Allow", "Principal": {"Service": "vpc-flow-logs.amazonaws.com"}, "Action": "sts:AssumeRole"}]}
   EOF
   cat > flowlogs-perms.json <<'EOF'
   {"Version": "2012-10-17", "Statement": [{"Effect": "Allow",
     "Action": ["logs:CreateLogGroup", "logs:CreateLogStream", "logs:PutLogEvents", "logs:DescribeLogGroups", "logs:DescribeLogStreams"],
     "Resource": "*"}]}
   EOF
   save FLOW_ROLE_ARN "$(aws iam create-role --role-name saa-lab-flowlogs \
     --assume-role-policy-document file://flowlogs-trust.json --tags Key=project,Value=saa-lab --query Role.Arn --output text)"
   aws iam put-role-policy --role-name saa-lab-flowlogs --policy-name write-logs --policy-document file://flowlogs-perms.json
   aws logs create-log-group --log-group-name /saa-lab/vpc-flow --tags project=saa-lab
   aws logs put-retention-policy --log-group-name /saa-lab/vpc-flow --retention-in-days 1
   sleep 10
   save FLOW_LOG_ID "$(aws ec2 create-flow-logs --resource-type VPC --resource-ids "$VPC_ID" --traffic-type ALL \
     --log-destination-type cloud-watch-logs --log-group-name /saa-lab/vpc-flow \
     --deliver-logs-permission-arn "$FLOW_ROLE_ARN" --max-aggregation-interval 60 \
     --tag-specifications "$(ts vpc-flow-log saa-lab-flow)" --query 'FlowLogIds[0]' --output text)"
   ```

10. **Baseline: client → web works.** Run `echo $WEB_IP` in CloudShell, then **on the client instance**:

    ```bash
    WEB=10.0.11.x                      # paste WEB_IP here
    curl -s --max-time 5 http://$WEB   # hello from ip-10-0-11-x...
    ping -c 3 $WEB                     # replies
    ```

11. **Add an S3 gateway endpoint, then prove S3 traffic uses it instead of the NAT.** First a bucket, and an IAM policy that lets the instance role read it:

    ```bash
    save LAB02_BUCKET "saa-lab-vpce-$ACCOUNT_ID"
    aws s3 mb "s3://$LAB02_BUCKET"
    echo "hello via the gateway endpoint" > hello.txt && aws s3 cp hello.txt "s3://$LAB02_BUCKET/"
    cat > ec2-s3-read.json <<EOF
    {"Version": "2012-10-17", "Statement": [
      {"Effect": "Allow", "Action": "s3:ListAllMyBuckets", "Resource": "*"},
      {"Effect": "Allow", "Action": "s3:ListBucket", "Resource": "arn:aws:s3:::${LAB02_BUCKET}"},
      {"Effect": "Allow", "Action": "s3:GetObject", "Resource": "arn:aws:s3:::${LAB02_BUCKET}/*"}]}
    EOF
    aws iam put-role-policy --role-name saa-lab-ec2-ssm --policy-name lab02-s3-read --policy-document file://ec2-s3-read.json
    ```

    Create the endpoint. Console: **Endpoints → Create endpoint → AWS services → `com.amazonaws.<region>.s3`, type Gateway → select the private route table**.

    ```bash
    save S3_VPCE "$(aws ec2 create-vpc-endpoint --vpc-id "$VPC_ID" --vpc-endpoint-type Gateway \
      --service-name "com.amazonaws.$REGION.s3" --route-table-ids "$PRIV_RT" \
      --tag-specifications "$(ts vpc-endpoint saa-lab-s3-gw)" --query VpcEndpoint.VpcEndpointId --output text)"
    aws ec2 describe-route-tables --route-table-ids "$PRIV_RT" \
      --query 'RouteTables[0].Routes[].[DestinationCidrBlock||DestinationPrefixListId, GatewayId||NatGatewayId, State]' --output table
    ```

    The private route table now has a `pl-xxxx → vpce-xxxx` route: S3's IP ranges as a **prefix list**. It is more specific than `0.0.0.0/0`, so longest-prefix match sends S3 traffic to the endpoint and everything else to the NAT.

    To prove which path the traffic takes, pin the bucket to the endpoint:

    ```bash
    cat > bucket-vpce-only.json <<EOF
    {
      "Version": "2012-10-17",
      "Statement": [{
        "Sid": "DenyGetUnlessViaMyEndpoint",
        "Effect": "Deny",
        "Principal": "*",
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::${LAB02_BUCKET}/*",
        "Condition": {"StringNotEquals": {"aws:SourceVpce": "${S3_VPCE}"}}
      }]
    }
    EOF
    aws s3api put-bucket-policy --bucket "$LAB02_BUCKET" --policy file://bucket-vpce-only.json
    aws s3 cp "s3://$LAB02_BUCKET/hello.txt" -    # from CloudShell: AccessDenied (CloudShell is not in your VPC)
    echo "$LAB02_BUCKET"
    ```

    **On the client instance:**

    ```bash
    aws s3 cp s3://saa-lab-vpce-<account-id>/hello.txt - --region us-east-1   # use your bucket and Region: works via the endpoint
    ```

    **The cost lesson:** every byte to S3 that went through the NAT gateway was billed per GB processed. Through a gateway endpoint the same traffic costs nothing, and the endpoint has no hourly charge. Gateway endpoints exist **only for S3 and DynamoDB**.

12. **Security groups are stateful.** Remove *all* outbound rules from the web SG:

    ```bash
    aws ec2 revoke-security-group-egress --group-id "$WEB_SG" \
      --ip-permissions 'IpProtocol=-1,IpRanges=[{CidrIp=0.0.0.0/0}]' --output text
    ```

    On the client, run `curl -s --max-time 5 http://$WEB` again. **It still works.** The SG tracks the connection and lets the response out, even with no outbound rules at all.

13. **NACLs are stateless.** Create a custom NACL for private-b. ⚠️ Trap: a new *custom* NACL denies everything (only the final `*` DENY rules exist), while the VPC's *default* NACL allows everything. Associate it, then allow only *inbound* HTTP and ICMP:

    ```bash
    save NACL_ID "$(aws ec2 create-network-acl --vpc-id "$VPC_ID" \
      --tag-specifications "$(ts network-acl saa-lab-nacl-private-b)" --query NetworkAcl.NetworkAclId --output text)"
    nacl_assoc() { aws ec2 describe-network-acls --filters Name=association.subnet-id,Values="$1" \
      --query "NetworkAcls[0].Associations[?SubnetId=='$1'].NetworkAclAssociationId" --output text; }
    aws ec2 replace-network-acl-association --association-id "$(nacl_assoc "$PRIV_B")" --network-acl-id "$NACL_ID" --output text

    # protocol 6 = TCP, 1 = ICMP
    aws ec2 create-network-acl-entry --network-acl-id "$NACL_ID" --ingress --rule-number 100 \
      --protocol 6 --port-range From=80,To=80 --cidr-block 10.0.0.0/16 --rule-action allow
    aws ec2 create-network-acl-entry --network-acl-id "$NACL_ID" --ingress --rule-number 110 \
      --protocol 1 --icmp-type-code Type=-1,Code=-1 --cidr-block 10.0.0.0/16 --rule-action allow
    ```

    On the client, `curl -s --max-time 5 http://$WEB` now **times out**, and `ping -c 3 $WEB` gets no replies. The request reaches the web server, but the response leaves from port 80 to the client's **ephemeral port** (32768–60999), and the NACL's outbound side denies it. Fix the outbound side:

    ```bash
    aws ec2 create-network-acl-entry --network-acl-id "$NACL_ID" --egress --rule-number 100 \
      --protocol 6 --port-range From=1024,To=65535 --cidr-block 10.0.0.0/16 --rule-action allow
    aws ec2 create-network-acl-entry --network-acl-id "$NACL_ID" --egress --rule-number 110 \
      --protocol 1 --icmp-type-code Type=-1,Code=-1 --cidr-block 10.0.0.0/16 --rule-action allow
    ```

    `curl` and `ping` work again. AWS recommends allowing 1024–65535 for return traffic because clients differ: Linux uses 32768–60999, Windows 49152–65535, and NAT gateways and ELB 1024–65535.

14. **Read the REJECT in the flow logs.** Records arrive 1–10 minutes after the traffic.

    ```bash
    aws logs filter-log-events --log-group-name /saa-lab/vpc-flow --filter-pattern REJECT \
      --start-time $(( ($(date +%s) - 3600) * 1000 )) --max-items 10 --query 'events[].message' --output text | tr '\t' '\n'
    ```

    Look for a line like the one below. Each record follows the default format, `version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status`.

    ```
    2 111122223333 eni-0a1b2c3d 10.0.11.25 10.0.10.37 80 41234 6 3 180 1730800000 1730800060 REJECT OK
    ```

    That is the web server's *response*: source port 80, sent to the client's ephemeral port 41234, protocol 6 (TCP), rejected on the way out. You will not find this pattern in a SG-only VPC, because SGs never reject return traffic.

    The same search in **CloudWatch console → Logs Insights** (log group `/saa-lab/vpc-flow`):

    ```
    fields @timestamp, srcAddr, srcPort, dstAddr, dstPort, protocol, action
    | filter action = "REJECT"
    | sort @timestamp desc
    | limit 20
    ```

15. **(Read, or run and tear down right away) NAT gateway per AZ for HA.** If AZ1 fails, the single NAT gateway goes with it, and private-b loses internet access even though AZ2 is healthy. The exam's HA pattern: **one NAT gateway in each AZ's public subnet, and one private route table per AZ that points to its own AZ's NAT**. It also removes the cross-AZ data charge.

    ```bash
    # Optional: doubles NAT cost. Undo commands are at the end of this block.
    EIP_B=$(aws ec2 allocate-address --domain vpc --tag-specifications "$(ts elastic-ip saa-lab-nat-b)" --query AllocationId --output text)
    NAT_B=$(aws ec2 create-nat-gateway --subnet-id "$PUB_B" --allocation-id "$EIP_B" \
      --tag-specifications "$(ts natgateway saa-lab-nat-b)" --query NatGateway.NatGatewayId --output text)
    aws ec2 wait nat-gateway-available --nat-gateway-ids "$NAT_B"
    PRIV_RT_B=$(aws ec2 create-route-table --vpc-id "$VPC_ID" --tag-specifications "$(ts route-table saa-lab-private-rt-b)" \
      --query RouteTable.RouteTableId --output text)
    aws ec2 create-route --route-table-id "$PRIV_RT_B" --destination-cidr-block 0.0.0.0/0 --nat-gateway-id "$NAT_B"
    aws ec2 modify-vpc-endpoint --vpc-endpoint-id "$S3_VPCE" --add-route-table-ids "$PRIV_RT_B"   # easy to forget!
    ASSOC_B=$(aws ec2 describe-route-tables --route-table-ids "$PRIV_RT" \
      --query "RouteTables[0].Associations[?SubnetId=='$PRIV_B'].RouteTableAssociationId" --output text)
    aws ec2 replace-route-table-association --association-id "$ASSOC_B" --route-table-id "$PRIV_RT_B"
    # Undo: put private-b back on PRIV_RT, delete NAT_B, release EIP_B, delete PRIV_RT_B
    #   aws ec2 replace-route-table-association --association-id <new assoc id> --route-table-id "$PRIV_RT"
    #   aws ec2 delete-nat-gateway --nat-gateway-id "$NAT_B"   (wait until deleted)
    #   aws ec2 release-address --allocation-id "$EIP_B"
    #   aws ec2 modify-vpc-endpoint --vpc-endpoint-id "$S3_VPCE" --remove-route-table-ids "$PRIV_RT_B"
    #   aws ec2 delete-route-table --route-table-id "$PRIV_RT_B"
    ```

    **New since Nov 2025:** NAT gateways also offer a *regional* mode that spans AZs automatically and needs no public subnet. Know that it exists. Exam questions still frame the choice as "a single shared NAT gateway vs one NAT gateway per AZ", so learn the zonal pattern above.

16. **Now see what the wizard does.** **VPC console → Create VPC → VPC and more.** Set 2 AZs, 2 public + 2 private subnets, NAT gateways *In 1 AZ* or *1 per AZ*, and VPC endpoints *S3 Gateway*. The preview pane draws exactly what you just built. Do **not** create it; just compare. In practice you would use the wizard or IaC (Lab 08). Building by hand once is how you learn what the wizard is doing.

## Verify

- [ ] `checkip.amazonaws.com` from the client returns the NAT gateway's Elastic IP.
- [ ] The private route table shows `0.0.0.0/0 → nat-…` and `pl-… → vpce-…`.
- [ ] CloudShell gets AccessDenied on `hello.txt`; the client instance can read it.
- [ ] Removing the web SG's outbound rules did **not** break HTTP; the NACL's missing outbound rule **did**.
- [ ] You found a `REJECT` flow-log record whose source port is 80 and whose destination port is ephemeral.

## Break it

1. **NACL rule order.** Add a deny for the client's IP with a *lower* rule number than the allow:

   ```bash
   CLIENT_IP=$(aws ec2 describe-instances --instance-ids "$CLIENT_ID" --query 'Reservations[0].Instances[0].PrivateIpAddress' --output text)
   aws ec2 create-network-acl-entry --network-acl-id "$NACL_ID" --ingress --rule-number 90 \
     --protocol 6 --port-range From=80,To=80 --cidr-block "$CLIENT_IP/32" --rule-action deny
   ```

   `curl` from the client fails, although rule 100 allows the VPC CIDR: NACL rules are evaluated **lowest number first, first match wins**. This is also the only way to *block* a specific IP inside a VPC, because SGs cannot deny. Remove the rule afterwards: `aws ec2 delete-network-acl-entry --network-acl-id "$NACL_ID" --ingress --rule-number 90`.

2. **Endpoint policies.** Restrict the gateway endpoint itself to your lab bucket:

   ```bash
   cat > vpce-policy.json <<EOF
   {"Version": "2012-10-17", "Statement": [{"Effect": "Allow", "Principal": "*",
     "Action": ["s3:GetObject", "s3:ListBucket"],
     "Resource": ["arn:aws:s3:::${LAB02_BUCKET}", "arn:aws:s3:::${LAB02_BUCKET}/*"]}]}
   EOF
   aws ec2 modify-vpc-endpoint --vpc-endpoint-id "$S3_VPCE" --policy-document file://vpce-policy.json
   ```

   On the client, `aws s3 ls --region <region>` (ListAllMyBuckets) is now **denied**, even though the instance role allows it. `aws s3 ls s3://<lab-bucket>` still works. An endpoint policy is a third filter, alongside the identity policy and the bucket policy. It is a common data-exfiltration control. Reset it with `aws ec2 modify-vpc-endpoint --vpc-endpoint-id "$S3_VPCE" --reset-policy`.

3. **Which traffic needs the NAT?** Delete the default route: `aws ec2 delete-route --route-table-id "$PRIV_RT" --destination-cidr-block 0.0.0.0/0`.
   - Within about a minute your Session Manager session freezes. The agent can no longer reach the public Systems Manager endpoints.
   - S3 via the gateway endpoint would still work. To keep SSM without a NAT, you would add **interface endpoints** for `ssm`, `ssmmessages` and `ec2messages`, which bill hourly per AZ.

   Restore the route with `aws ec2 create-route --route-table-id "$PRIV_RT" --destination-cidr-block 0.0.0.0/0 --nat-gateway-id "$NAT_ID"` and reconnect.

## Exam takeaways

- **Public subnet = route table with `0.0.0.0/0 → IGW`.** A private subnet reaches the internet outbound through a **NAT gateway placed in a public subnet**. For HA, use **one NAT GW per AZ, each with its own private route table**. One shared NAT GW is cheaper but has an AZ-wide blast radius and cross-AZ charges.
- NAT GW bills **per hour + per GB processed**. "Reduce NAT costs for S3/DynamoDB traffic" → **gateway VPC endpoint** (free; route-table based; S3 and DynamoDB only). Interface endpoints (PrivateLink) put ENIs in your subnets, bill hourly per AZ, and are reachable from on-premises and peered networks; gateway endpoints are not.
- Lock a bucket to your network with `aws:SourceVpce` (or `aws:SourceVpc`) in the **bucket policy**. Restrict what the network can reach with an **endpoint policy**.
- **Security groups:** stateful, allow-only, attached to ENIs, can reference other SGs. **NACLs:** stateless, allow + deny, numbered rules (first match wins), apply per subnet, need **ephemeral ports 1024–65535** for return traffic. "Block one malicious IP" → NACL deny (or AWS WAF at the edge).
- **Session Manager** = no bastion, no inbound ports, no SSH keys, IAM-controlled, audited. It needs the SSM agent, an instance profile with `AmazonSSMManagedInstanceCore`, and a path to the SSM endpoints (NAT or interface endpoints). EC2 Instance Connect Endpoint is the other no-bastion option.
- **VPC Flow Logs** record metadata (5-tuple, bytes, ACCEPT/REJECT), never payloads. They can go to CloudWatch Logs, S3 or Firehose, and are the first tool for "is it the SG or the NACL?". They do not capture traffic to the Amazon DNS resolver or the instance metadata service.
- Every subnet loses **5 IPs** to AWS reservations. A VPC's primary CIDR can't be changed, but you can add secondary CIDRs.

## Cleanup ✅

Choose **Path A** if you will do Lab 03 or Lab 05 this week. Choose **Path B** for a full teardown. Both start with the hourly resources.

**Path A: keep the free skeleton** (VPC, subnets, route tables, IGW, S3 gateway endpoint, SGs, instance profile all cost $0):

- [ ] Terminate both instances: `aws ec2 terminate-instances --instance-ids "$CLIENT_ID" "$WEB_ID" --output text`
- [ ] Delete the NAT gateway and release its EIP: `nat_down`. The private route's target will show **blackhole** until Lab 03 runs `nat_up`.
- [ ] Delete the flow log and its log group:

  ```bash
  aws ec2 delete-flow-logs --flow-log-ids "$FLOW_LOG_ID"
  aws logs delete-log-group --log-group-name /saa-lab/vpc-flow
  ```

- [ ] Put private-b back on the default NACL and delete the custom one:

  ```bash
  DEFAULT_NACL=$(aws ec2 describe-network-acls --filters Name=vpc-id,Values="$VPC_ID" Name=default,Values=true \
    --query 'NetworkAcls[0].NetworkAclId' --output text)
  aws ec2 replace-network-acl-association --association-id "$(nacl_assoc "$PRIV_B")" --network-acl-id "$DEFAULT_NACL" --output text
  aws ec2 delete-network-acl --network-acl-id "$NACL_ID"
  ```

  If `nacl_assoc` is gone because your session restarted, re-define it from step 13.
- [ ] Reset the endpoint policy if you changed it (`--reset-policy`), delete the bucket and the instance role's S3 policy:

  ```bash
  aws s3 rb "s3://$LAB02_BUCKET" --force
  aws iam delete-role-policy --role-name saa-lab-ec2-ssm --policy-name lab02-s3-read
  ```

- [ ] If you ran step 15: run its undo commands.

**Path B: full teardown** (Path A first, then):

- [ ] Endpoint, SGs (web first, because it references the client SG), subnets, IGW, route tables, VPC:

  ```bash
  source ~/saa-lab.env
  aws ec2 wait instance-terminated --instance-ids "$CLIENT_ID" "$WEB_ID"
  aws ec2 delete-vpc-endpoints --vpc-endpoint-ids "$S3_VPCE" --output text
  aws ec2 delete-security-group --group-id "$WEB_SG"
  aws ec2 delete-security-group --group-id "$CLIENT_SG"
  for s in "$PUB_A" "$PUB_B" "$PRIV_A" "$PRIV_B"; do aws ec2 delete-subnet --subnet-id "$s"; done
  aws ec2 detach-internet-gateway --internet-gateway-id "$IGW_ID" --vpc-id "$VPC_ID"
  aws ec2 delete-internet-gateway --internet-gateway-id "$IGW_ID"
  aws ec2 delete-route-table --route-table-id "$PUB_RT"
  aws ec2 delete-route-table --route-table-id "$PRIV_RT"
  aws ec2 delete-vpc --vpc-id "$VPC_ID"
  ```

- [ ] IAM: the flow-logs role and the EC2 role/instance profile (if you are not doing Labs 03/05):

  ```bash
  aws iam delete-role-policy --role-name saa-lab-flowlogs --policy-name write-logs
  aws iam delete-role --role-name saa-lab-flowlogs
  aws iam remove-role-from-instance-profile --instance-profile-name saa-lab-ec2-ssm --role-name saa-lab-ec2-ssm
  aws iam delete-instance-profile --instance-profile-name saa-lab-ec2-ssm
  aws iam detach-role-policy --role-name saa-lab-ec2-ssm --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
  aws iam delete-role --role-name saa-lab-ec2-ssm
  ```

- [ ] Run the [leftover finder](README.md#7-universal-cleanup-checklist). In particular, confirm that no NAT gateway and no Elastic IP remain.
