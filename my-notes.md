# AWS SAA notes + resources

# Fundamentals
- **regions** - (ex. 'us-east-1', the largest), cluster of data centers, most services region-scoped, each one has 2+ availability zones (AZs)
- **AZs** - 1+ discrete data centers with redundant power, networking, and connectivity, separate from each other = isolated from disasters, high bandwidth and ultra low latency networking
- **edge locations** - delivering content at high speeds, owned by trusted partners of AWS, used with CloudFront and R53, connects to AWS network, requests to CF and R53 are auto-routed to nearest EL.


## Identity and Access Management (IAM)
- **users** - user/person with least privileges, assign users to groups 
- **groups** - group of users, assign permission to groups 
- **roles** - use for giving permissions, 1 role per app
- **root** account - never share it, only login for intital setup and admin changes, use regular acct for day-to-day 
- policies written in JSON
- create password policy, use + enforce MFA
- use access keys for programmatic access (CLI, SDK)
- audit sec/perms with **Credentials Report** (account-level, displays simple UI to d/l CSV file, lists all IAM users and their status, active AK, if MFA enabled, etc) / **Access Advisor** (user-lvl - shows perms granted to users and svs accessed, use to revise pols)
- never share IAM users and AKs
- use orgs, tags, config resources, CW logs (send svc and access log here), CloudTrail to record API calls, Trusted Advisor for insights
- **Federation** - SAML, AD, login to AWS with company creds


## Elastic Compute Cloud (EC2)
- virtual server/launch server in the cloud, can start/stop whenever, can end on demand, host various types of workloads on svr / highly-configurable svr - pick CPU, mem, ntwk, OS
- **Instance Connect** - connect your EC2i w/in browser, no need for key file, temp key is uploaded by AWS, works only with Amazon Linux 2, leave port 22 open
- **security groups** - firewalls on EC2i, regulates access to ports, authorized IP ranges, controls how traffic is allowed in/out of EC2 machines, know this skill for troubleshooting ntwkg issues, SGs can be attached to many ints, locked down to region/VPC, lives outside EC2 (EC2i wont see if traffic blocked), maintain one sep SG for SSH access, app not accessible -> SG issue, connection refused -> app error or not launched, all inbound traffic is blocked by def, outbound autho by default
- IPv4 -  most common, 3 bil public addresses, format  `[0-255].[0-255].[0-255].[0-255]`
- **public IP** means machine can be ID'd on web, unique, easily geo-located
- **private IP** is ID'd only on private ntwk, unique across private ntwk, 2 diff PNs (2 companies) can have same IPs, connect to web using NAT and internet gw (proxy), specified range of IPs can be used as priv IP
- **elastic ip** - when you stop and then start an EC2i it changes its public IP, if you need to have fixed public ip for inst need eip. eip is public IPv4 you own long as you don't del, can attach it to one inst at a time, can mask failure of inst or software by remapping add. to another inst in your acct. can oly have 5 eips per acct (ask aws for increase) / avoid using eip - poor architectural decisions, use random public ip and register a DNS name, or use load balancer and not public IP
- EC2 user data - script launched at 1st start of an instance, runs with root user, can bootstrap/customize an inst using EC2 User data script. bootstrapping - launch cmds when machine start. can automate boot tasks like install updates and software, d/l common files from www, etc
- **launch types**  

launch types | about
--- | ---
**on demand** | short-term and uninterrupted w/l, predictable pricing, pay per use, least commitment, highest cost but no upfront pmt
**reserved instances** | min 1yr, long w/l, 72% disc vs OD, 1-3 year terms, longer term = greater savings, steady state, predictable usage (for ex. DBs), convertible RIs can change instance type, 54% disc, long w/l with flexible inst, scheduled RIs can launch w/in time window reserved (day, wk, month), 1 yr only / upfront pmt - all, partial, none. can resell unused RIs
**spot instances**  | short w/l, cheap less reliable up to 90% disc lose inst anytime if max price less than current spot price, handles interruptions, no charge if aws cnx and only charged when you cnx, used for batch jobs, data analysis, not critical jobs or DBs, can only term SI requests that are open/active/disabled, cnx a spot req doesnt term inst, cnx sr then term associated SI. spot fleets = set of Spot Instances + (optional) On-Demand Instances allows you to auto-request Spot Instances with the lowest price
**dedicated instances** | hosts - book entire servers, 3yr reserved, for compliance reqs licenses and BYOL, control inst placement. ded inst - no other customers, no placement control, can move after start/stop
<br>

- **instance types** - desc of kind of hardware resources your inst will be using (t2.micro). vCPU - the more you get the better your inst perf. / R = RAM, in-mem cache, C = CPU, compute/dbs, M = balanced/medium, general/web app, I = local I/O instance storage, dbs, G= GPU, video rendering, ML / T2/T3 burstable instances (up to a capacity) - burst means OK CPU perf, machine bursts, it utilizes “burst credits”, credits are gone, CPU = BAD, machine stops = credits accumulated overtime / T2/T3 - unlimited: unlimited burst - pay extra if over credit balance but dont lose perf / use https://instances.vantage.sh 
  
- **Amazon Machine Images** - AMIs lives on S3 (charged for space used in s3), are private, locked for your acct/region. image is a software bundle built from a template definition and made avail w/in single region. region-specific, can copy across regions, 4 collections - quick start set, custom AMI you create, AWs marketplace, community AMIs / can share AMI with other AWS account. if shared, youre the owner of target AMI in your acct. process - start instance, customize it, stop instance (data integrity), build AMI - creates snapshots, launch instance from other AMI / snapshots - backup of EBS vol, detach volume to snapshot. can copy ss across AZ or region. / remove AMIs you dont use. inexpensive to store private AMIs. cant copy encrypted AMI thats shared with you from another acct, copy ss while re-encrypting it with a key you own. cant copy AMI with billingProduct code. to copt shared AMI with that code, launch EC2i in acct using shared AMI then create an AMI from inst
- **placement groups** - control placement of EC2i. ec2i can be started in placement grps. 3 strategies - cluster - clusters inst into low-latency group in single az, great network, rack fails = all inst fail at same time, for big data jobs and apps needing real low latency and high ntwk throughput / spread - spreads instances across difft physical hardware, limited to 7 inst per group per AZ, for apps needing max HA, critical apps where each inst must be isolated from each other / partition - spreads inst across difft partitions with AZ. scales to 100s of EC2i per group, 7 partitions per AZ, they dont share racks with inst in the the other partitions, part failure can affect many EC2 but not other partitions, EC2i get access to info as metadata, or Hadoop, Cassandra, Kafka
- **Elastic Network Interfaces** - ENI, logical component in a VPC that reps a virtual network card, attribs - primary private IPv4 and 1+ secondary IPv4, one EIP per private IPv4, one public IPv4, 1+ SGs, and a MAC address. can create ENI and move them on other ec2i for failover, bound to specific AZ 
- **EC2 Hibernate** - in-mem (RAM) state is preserved. instance boots faster, OS not stopped or restarted. RAM state is written to file in EBS vol, root EBS must be encrypted. for long-running processes, saving RAM state, services that take time to start. supported fams - C3-C5, M3-M5, R3-R5. instance RAM size must be <150GB. instance size not support for bare metal instances, AMI - Amazon Linux 2, Linux AMI, Ubuntu, Windows. root vol must be EBS encrypted, not instance store, large. avail for on-dem and RIs. instance cant be hibernated more than 60 days. 
- EC2 for Solutions Architect - ec2i billed by second, t2.micro = free tier. linux/mac uses SSH, WIndows use putty, SSH in on port 22, lock down the SG to your IP, timeout issues = SG issues. perm issues on SSH key - run `chmod 0400`. **SGs can reference other SGs instead of IP ranges**, know launch modes, instance types (R, C, M..). can create AMIs to pre-install software on ec2 for faster boot.   

# Fundamentals II
- **scalability** -  vertical - scale up / down - increases in size (from jr to sr). t2.micro -> t2.large. common for DBs. RDS, Elasticache scale vertically. theres a hardware limit. / horizontal - scale out / in - increase # of instances. for web apps and modern apps. can horiz scale with EC2, ASG, LB. 
- **high availability** - HA works with h.s. means running app in at least 2 AZs to survive data center loss. can be passive - run same app across multi-az (RDS/ASG/LB multi-az) can be active (for h.s.)
- **load balancing** - servers that fwd traffic to ec2i downstream. use LBs to spread load, expose single point of access (DNS) to your app. seamlessly handle failures, do reg health chks, prove SSL term for sites. HA across AZs. sep public traffic from private. 
- **health checks** - for lbs, enables lbs to know if inst it fwds traffic to are avail to reply to reqs. done on a port and a route. (/health). 200 response = OK = healthy
- **load balancer types** 
- classic - old gen, L4 and L7, for apps built within ec2 classic ntwk
- application lb | HTTP/s = L7 - layer 7 requests - http/s traffic. routing rules - more usability from one lb. can attach waf. HTTP/2 and WebSocket support. supports redirects from http/s. lb-ing to apps across machines (target grps). lb-ing apps on same machine (containers)  / ALB can route to multiple target groups. health chks are target group lvl. 
- network lb | TCP = L4 - layer 4 protocol data. for extreme performance, tcp, tls (secure tcp), udp traffic. handled mil reqs with low lat per sec. optimized sudden and volatile traffic using 1 static Ip per az. supports eip for whitelisting specific IP. not inc in free tier.
- lb stickiness - for clb and alb. cookie used for stickiness has exp. date you control. use case - make user no lose session data. brings ibalance to load over backend ec2i.
- **cross-zone lb** - each load balancer instance dist evenly across all reg'd instances in all AZ. or each lb dist reqs across reg'd ins in its AZ only. cant be disabled in alb. no charges for inter-az data. disabled by def in clb and nlb. nlb - charged for inter-az data if enabled. 
- troubleshooting - 4xx = client errors. 5xx = app errors. lb error 503 = at capacity or no registered targer. check Sgs if lb cant connect to app.
- monitoring - elb access logs will log all access requests so can debug per req. CW metrics give aggregate stats. 
- SSL - Secure Sockets Layer, used to encrypt connections. public ssl certs issued by Cert Authorities (CA). exp date you set and must be renewed. lb uses X.509 certificate. can manage certs using Amazon Certification Manager (ACM). can create own certs. / Server Name Indication (SNI) - solves loading SSL certs onto 1 web svr to serve many sites. queuires client to indicate hostname of target svr in initial handshake. svr finds correct cert or return to default. for alb and nlb, CFt. 
- TLS - Transport Layer Security, mainly used. 
- EC2 LB is a managed lb. evenly dist incoming traffic across multiple targets. always working (no downtime) and completely mgd by aws (upgrades, maint, HA). less to setup your own lb but extra effort. integrated with other aws services. regular health cks. 
- ELB - CLB - Connection Draining / Target Group - Deregistration Delay for ALB and NLB 


## Auto Scaling Groups (ASG)
- scale out/in = add/remove instances to match load. create instances on demand. create SG, create conditions for scaling processes.
- ASGs have a launch config - AMI and Instance type, EC2 User Data, EBS vols, SG, SSH key pair / min/max capacity / lb info / scaling policies
- AS alarms - possible to scale an Asg based on CW alarms. monitors metrics (avg cpu). based on alarm, can create scale in or scale out pols.
- Rules - can define better autoscaling rules directly mgd by ec2 - target avg cpu usage, # of requests on elb per instance. avg network in or out. theses rules are easier to setup.   
- Custom Metric - can autoscale based on custom metric or based on a schedule(ex. # of connected users)
-  instances under an ASG = ASG auto creates new ones as a replacement. will terminate instances marked as unhealthy by an LB. ASGs are free and pay for res launched.
-  uses launch configs or launch templates. to update an asg, provide new lc or lt.
-  IAM roles attached to asg get assigned to ec2i
-  **scaling policies** target tracking scaling - simple and easy setup - "i want the avg cpu to stay at around 40%". / simple/step scaling - CW alarm triggered then adds 2 units if cpu >70% or removes if cpu <30% / scheduled actions - based on usage patterns. "increase min capacity to 10 at 5pm frid"
- **scaling cooldowns** cooldown period helps to ensure that your Auto Scaling group doesn't launch or terminate additional instances before the previous scaling activity takes effect
- **Default Termination Policy** - find az with most instances, deletes ones with oldest launch configuration. ASG balances # of ins by default.
- **lifecycle hooks** - an instance launched in an ASG is in service. can perform extra steps/actions before inst goes in service (pending state) or terminated (terminating state)
- launch config (legacy) recreate every time. launch template (recommended) many versions. create params subsets, provision using on-demand and SIs or mix. can use T2 unlim burst. 

## Elastic Block Storage (EBS)
- ec2 block storage volumes. network/virtual hard drive in the cloud. a way to store your instance data. can attach ins while they run. locked by az. have a provisioned capacity (in GBs and IOPS)
- IOPS - input/output per seconds. how fast you can read and write to vol. high IOPS = faster r+w

EBS vol types | info 
--- | ---
GP2 - general purpose SSD | recommended, cheap. typical w/l, bursting - added boost for heavy spikes. system boot volumes. virtual desktops. dev and test env. max IOPS is 16K. 3 IOPS per GB = 5334GB (max IOPS)
IO1 - provisioned IOPS SSD | expensive. highest-perf ssd vol for mission-critical low latency or high thruput w/l, no bursting (for dbs)
ST1 - throughput optimized HDD | w/l needing more thruput, streaming w/l, fast thruput at low price (for big data, data warehouses, log processing), apache kafka. can't be a boot volume. can burst. max IOPS is 500
SC1 - cold HDD | archive storage, infrequently accessed data

<br>

- **EBS snapshots** - stored in s3. recommended to detach volume to do snapshot. max = 100K ss. can copy across az and regions. can make AMI from ss. ebs vols restored by ss need to be pre-warmed using fio or dd command to read entire vol. ss can be automated using **Data Lifecycle Manager**. 
- **EBS Migration** - vols are locked to 1 az. to migrate to difft az - ss vol, copy col to dfft region, create vol from ss in the az of your choice
- **EBS encryption** - has a minimal impact on latency. uses KMS (AES-256). copying unenc ss allows encryption. ss of enc vol are enc. to enc an unenc ebs vol - create ebs ss of vol, enc ebs ss, create new ebs vol from ss (vol also enc), then attach enc vol to original inst.
- **Instance Store** - instance store = ephemeral storage. provides faster r+w and more secure. a physically attached to the machine. better i/o perf, good for buffer/cache/temp content, data survives reboot. term inst = inst store lost. cant resize. backups operated by user. 
- **RAID options** - mount vols in parallel in RAID settings. long as OS supports it. RAID 0, 1, 5, 6. (5 and 6 not recommended) / RAID 0 to inc perf. RAID 1 to inc fault tolerance


## Elastic File System (EFS)
- scalable network file system (NFS) thats mounted to many ec2 at the same time. works with ec2i in multi-az in a vpc. highly avail, expensive (3x GP2), pay per use, autoscales. for linux based ami not windows / POSIX file system. petabyte. uses SG to control access to efs. enc at rest using kms 
- for content mgmt, web serving, data sharing 
- 

## Relational Database Service (RDS)
- managed DB svc. data with meaning (like excel sheets). allows to create/setup a RDB in aws. can scale anytime - vertically - scale up/down or horizontally - scale in/out. supports mysql, postgres, mariadb, oracle, SQl server.
- backups automated enabled. daily full b/u, logs backed by rds every 5min. can restore to any point in time. retention of back up is 7-35 days 
- enable HA of db by using multi-az ftr (auto-replicates primary db into another az) for disaster recovery.
- storage backed by ebs - gp2 or io1
- cant ssh into instances
- DB snapshots - triggered by user. retention of b/u is as long as you want. 
- **read replicas** - up to 5 RRs. w/in AZ, cross AZ, cross region. replication is async = reads are consistent. replicas can have own db. ex. for run a reporting app to run analytics. network cost when data goes from 1 az to another. can reduce costs by having RRs in same AZ.
- multi-az - disaster recovery - SYNC replication. increases avail. not used for scaling. RRs setup as multi-az for DR
- RDS encryption - at rest enc. - done only when you first create the DB instance / enc master and rrs with KMS - aes-256. enc defined at launch time. if master encry, then RRs are enc. / in-flight enc - ssl certs to enc data to RDS in flight. provide ssl options with cert when you connect to db. 
- operations - enc RDS b/u - ss of unenc rds db are unenc. ss of enc rds are enc. can copy ss into an enc one. to enc an unenc rds db - create ss of unenc db, copy ss and enable enc for ss, restore db from enc ss, migrate apps to new db and del old db
- network security - rds dbs are deployed w/in a private subnet not public. uses SGs. controls which ip/sg gets access. 
- access mgmt - IAM pols control who manages aws rds (thru api). iam-based authentication used to log into rds mysql and postgreSQL - dont need pw just auth token from IAM and RDS api calls. token lasts 15 mins. 
- you - check ports, ip, SG inbound rules. perms created or managed thru IAM. create db with or w/o public access. make sure param groups or db congifured to allow SSL connections only 

## Amazon Aurora
- fully, high performanced mgd relational db that runs on rds. mysql = 5x faster and postgresql = 3x faster. reads via columns
- costs more than RDS but more efficient. highly avail.
- can have 15 replicas whil mysql has 5. repl process is faster 
- HA and read scaling. 6 copies of data across 3 AZs. 4 copies for writes and 3 copies for reads. 
- supports cross region replication. has backup and recovery. backtrack to restore data without using backups
- security - enc at rest using kms - enc done only when you first create the instance. backups automated. ss and replicas are enc. enc in flight using SSL. youre resp for protecting instance with SGs. cant ssh.
- **Aurora Serverless** - runs when needed. can autoscale >100k transactions per sec so you pay per sec. for unpredictable w/l. cost effective. / 
- Global Aurora - cross region RR for dis rec. / Aurora Global DB is recommended. - 1 primary region (r/w), up to 5 secondary regions (read only), replication lag is a <1 sec. up to 16 RRs per secondary region, 

## ElastiCache
- managed redis and memcached DB. fully managed and scalable svc. caches are in-memory dbs with high performance and low latency. 
- for gaming and IoT apps
- reduces load off of DBs for read intensive w/l. 
- AWS takes care of OS maint, setup, configuration, backups


## Route 53



## S3 



## AWS CLI



## SDK



## IAM Roles & Policies



## Advanced Amazon S3



## Athena



## CloudFront



## Global Accelerator



## Storage Extras



## SQS



## SNS



## Kinesis



## Active MQ



## Serverless



## Databases



## CloudWatch



## CloudTrail & Config



## Advanced IAM



## KMS



## SSM Parameter Store



## CloudHSM



## Shield



## WAF



## VPC



## Disaster Recovery & Migrations



## More Solutions Architectures



---

### Resources

- [AWS FAQs](https://aws.amazon.com/faqs/)
- [AWS VPC Core Concepts in an Analogy and Guide](https://start.jcolemorrison.com/aws-vpc-core-concepts-analogy-guide/)
- [AWS Certified Solutions Architect – Associate SAA-C02 Exam Learning Path](https://jayendrapatil.com/aws-certified-solutions-architect-associate-saa-c02-exam-learning-path/)
- [AWS Cheat Sheets - Tutorials Dojo](https://tutorialsdojo.com/aws-cheat-sheets/)
