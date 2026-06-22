1: Your EC2 has a public IP and the port is open in the security group, but it's unreachable. Why?

Ans: Check the subnet’s NACL. If inbound or outbound rules are blocking traffic, the security group won’t help. NACLs silently drop traffic with no message.

I would troubleshoot layer by layer:

1. Check Security Group
    * Verify inbound rule allows the required port (e.g., 80, 443, 22).
    * Verify source CIDR is correct (e.g., 0.0.0.0/0 or specific IP range).
2. Check Network ACL (NACL)
    * NACLs are stateless, so both inbound and outbound rules must allow traffic.
    * A deny rule or missing ephemeral port range can block communication.
    * Traffic is dropped silently without logs.
3. Verify Route Table
    * Ensure the subnet has a route to the Internet Gateway (IGW):
      0.0.0.0/0 → Internet Gateway

          * Without this route, the instance cannot communicate with the internet.
4. Check Internet Gateway
    * Confirm the VPC has an Internet Gateway attached.
5. Verify the Application is Running
    * Check if the service is actually listening
      sudo netstat -tulpn
      Confirm the application is bound to 0.0.0.0 and not only 127.0.0.1.
      
“Even if the EC2 has a public IP and the Security Group allows the port, I would verify the NACL rules, route table, Internet Gateway attachment, OS firewall, and whether the application is actually listening on the expected port and bound to 0.0.0.0. In many cases, the issue is either a NACL block, missing route, or the application not listening externally.”

Cross Question 1

Interviewer:
Security Group allows traffic. Why should I check NACL?

Answer:
Security Groups are stateful, but NACLs are stateless. Even if inbound traffic is allowed by the Security Group, the NACL can still block inbound or outbound traffic. Since NACLs evaluate rules in order, a deny rule can silently drop packets.

⸻

Cross Question 2

Interviewer:
What do you mean by stateful and stateless?

Answer:

Security Group (Stateful)

If inbound traffic is allowed, the return traffic is automatically allowed.

Example:

* Allow inbound TCP 80.
* User accesses website.
* Response traffic automatically returns.

NACL (Stateless)

You must explicitly allow both directions.

Example:

* Allow inbound TCP 80.
* Also allow outbound ephemeral ports (1024-65535).
* Otherwise communication fails.

⸻

Cross Question 3

Interviewer:
How will you identify whether traffic is blocked by NACL?

Answer:
I would:

* Review NACL inbound and outbound rules.
* Enable VPC Flow Logs.
* Check whether traffic is marked as REJECT.
* Verify ephemeral ports are allowed.

⸻

Cross Question 4

Interviewer:
The NACL is fine. What’s next?

Answer:
I would verify:

* Route table has:
0.0.0.0/0 → Internet Gateway
    Internet Gateway is attached to the VPC.
* The subnet is public. 


  =====================

2) You shared an AMI with another AWS account, but they still can’t launch an instance from it. What’s usually missed?

Ans: Sharing the AMI isn’t enough. You also need to share the associated EBS snapshot. Without that, the AMI looks valid but fails at launch.

Cross Question 1

Interviewer:

Why does AWS need the snapshot if the AMI is already shared?

Answer:

An AMI contains:

* OS configuration
* Launch settings
* Block device mappings
* References to EBS snapshots

The actual disk data resides in the EBS snapshots.

During launch, AWS creates EBS volumes from those snapshots. If snapshot permissions are missing, AWS cannot create the volume.

⸻

Cross Question 2

Interviewer:

How do you share an EBS snapshot?

Answer:

1. Open EC2 Console.
2. Go to Snapshots.
3. Select the snapshot.
4. Actions → Modify Permissions.
5. Add the target AWS Account ID.

Cross Question 3

Interviewer:

The AMI and snapshot are shared, but the launch still fails. What else could be wrong?

Answer:

The snapshot may be encrypted using a KMS key.

The target account must also have permission to use that KMS key.

This is a very common issue in enterprise environments.

⸻

Cross Question 4

Interviewer:

How do you share an encrypted AMI?

Answer:

Three things must be shared:

1. AMI
2. EBS Snapshot
3. KMS Key

The KMS key policy must allow the target account.

Without KMS permissions, instance launch fails because AWS cannot decrypt the snapshot.
Interviewer:

5: In a production environment, what is your preferred method—sharing or copying?

Answer:

For production, I prefer copying the AMI into the destination account.

Reasons:

* Independent ownership.
* No dependency on source account.
* Better disaster recovery.
* Easier lifecycle management.
* Avoids accidental deletion by source account.

⸻


“When an AMI is shared but instance launch fails, the most common issue is that the associated EBS snapshot wasn’t shared. If the snapshot is encrypted, the KMS key permissions must also be shared. I would verify AMI permissions, snapshot permissions, KMS access, and review EC2 launch error messages to identify the exact failure point.”

=============================

3: You restored an RDS snapshot for staging, but some queries behave differently than production. 

Ans: When you restore from a snapshot, RDS assigns the default parameter group by default. Custom parameter groups from production are not restored automatically. 
If not manually reassigned, staging may run with different settings, leading to changes in query behavior or performance.

Your Answer:

One possible reason is that the restored RDS instance is using a different parameter group.

When an RDS snapshot is restored, AWS may associate the instance with the default parameter group unless the original custom parameter group is manually attached.

Differences in parameters such as memory allocation, query cache settings, optimizer behavior, timeouts, or logging can cause query execution plans and performance to differ from production.

⸻

Cross Question 1

Interviewer:

What is a Parameter Group?

Answer:

A Parameter Group is a collection of database engine configuration settings.

It’s similar to:

* MySQL → my.cnf
* PostgreSQL → postgresql.conf

  Interviewer:

Q How would you ensure every restored environment matches production?

Answer:

I would automate provisioning using Terraform.

The RDS definition would explicitly include:

* Parameter Group
* Option Group
* Engine Version
* Storage Type
* Monitoring Configuration

This prevents manual mistakes during snapshot restores.

⸻

Real Production Scenario

Interviewer:

Have you seen this issue in production?

Answer:

Yes. A staging environment restored from production snapshots showed significantly slower queries.

Investigation revealed:

* Production used a custom parameter group.
* Staging was attached to the default parameter group.
* innodb_buffer_pool_size and connection settings differed.

After attaching the correct parameter group and rebooting the instance, query performance matched production.

⸻

“When an RDS snapshot is restored, I first verify that the same Parameter Group, Option Group, instance class, storage type, and engine version are being used. A common issue is that the restored database gets associated with a default Parameter Group instead of the production custom Parameter Group, which can change optimizer behavior and query performance.”

==================






















































































