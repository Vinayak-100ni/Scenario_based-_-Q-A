# Junior Support Engineer – L1 Infrastructure Monitoring
## Interview Questions & Speak-Directly Answers

> **How to use this document:** The answers are written in natural interview language. Do not memorize every word mechanically. Understand the flow and speak confidently. Replace tool/company names with the exact ones you have actually used.

---

# 1. HR & Introduction

## 1. Tell me about yourself.

**Answer:**

"Hi, I’m Vinayak. I have around 3+ years of experience in DevOps and cloud infrastructure. I have worked with AWS and Azure, and I have hands-on experience with Linux, Docker, Kubernetes, Terraform, CI/CD, monitoring, and cloud services.

In my current work, I’m involved in infrastructure troubleshooting, deployments, monitoring, and resolving issues related to servers and applications. I’m comfortable working with Linux commands, checking logs, CPU, memory, disk, networking, and application health.

I’m now looking for an opportunity where I can use my technical troubleshooting skills in a structured support environment, learn from experienced engineers, and grow further in infrastructure and cloud operations."

---

## 2. Walk me through your technical background.

**Answer:**

"My background is mainly in cloud and DevOps. I have worked with Linux servers, AWS services such as EC2, IAM, S3 and CloudWatch, and also with Azure infrastructure.

I have experience with Terraform for infrastructure automation, Docker and Kubernetes for containers, and CI/CD tools for application deployments. I have also worked with monitoring tools and troubleshooting production-related issues.

Because of this experience, I’m comfortable starting troubleshooting from the infrastructure layer and moving toward networking, services, logs, and application health."

---

## 3. Why do you want to join the company?

**Answer:**

"I’m interested because the role gives me an opportunity to work in infrastructure monitoring and production support. I like troubleshooting technical issues and working with Linux, cloud, monitoring, and networking.

I also see this role as a good opportunity to work in a structured 24×7 support environment, understand incident management and SLAs better, and strengthen my operational skills while contributing to the team."

---

## 4. What do you understand about the Junior Support Engineer role?

**Answer:**

"My understanding is that an L1 Support Engineer is responsible for monitoring infrastructure, responding to alerts and tickets, performing initial troubleshooting, following SOPs and runbooks, documenting actions, communicating with users or internal teams, and escalating issues to L2 or L3 when required.

The main responsibility is to detect issues quickly, validate whether the alert is genuine, perform the troubleshooting allowed at L1, minimize impact, and make sure the issue is properly tracked until resolution or handover."

---

## 5. Why are you interested in an L1 Support Engineer position?

**Answer:**

"I already have infrastructure and DevOps experience, and I want to strengthen the operational side of my career. L1 support gives me strong exposure to real-time monitoring, incident handling, troubleshooting, SLAs, and production environments.

I believe my existing cloud and Linux knowledge will help me contribute from the beginning, while the support environment will help me become stronger in incident management and systematic troubleshooting."

---

## 6. Why should we hire you?

**Answer:**

"I have a combination of hands-on infrastructure knowledge and a troubleshooting mindset. I’m comfortable with Linux, AWS, networking basics, Docker, Kubernetes, Terraform, and monitoring.

I also understand that in support, technical knowledge alone is not enough. It is important to follow procedures, prioritize incidents based on impact, communicate clearly, document everything, and escalate at the right time.

I believe I can bring both the technical skills and the discipline required for an L1 support role."

---

## 7. What are your strengths?

**Answer:**

"My main strengths are troubleshooting, willingness to learn, staying calm during incidents, and being systematic.

When I face an issue, I normally break it down into layers such as connectivity, server resources, service status, application health, and logs instead of randomly changing things."

---

## 8. What is your biggest weakness?

**Answer:**

"Earlier, I sometimes spent too much time trying to solve an issue independently before escalating. I’ve learned that in production support, escalation is not a failure. If an issue is outside my access or responsibility, or if the SLA or business impact requires it, I should escalate early with complete troubleshooting information.

Now I focus on balancing troubleshooting with timely escalation."

---

## 9. Where do you see yourself in the next 2–3 years?

**Answer:**

"In the next two to three years, I want to become a strong infrastructure or cloud engineer with solid production support experience. I want to deepen my knowledge of AWS, Kubernetes, networking, monitoring, automation, and incident management.

My goal is to grow from L1 responsibilities into higher-level infrastructure or cloud operations responsibilities while continuing to improve my troubleshooting skills."

---

## 10. Are you comfortable working in a 24×7 rotational shift, including night shifts?

**Answer:**

"Yes, I’m comfortable with rotational shifts and night shifts. I understand that infrastructure monitoring is a 24×7 responsibility and incidents can happen at any time.

During a night shift, I would follow the defined SOPs and escalation matrix, handle the issues within my responsibility, communicate clearly, and escalate critical issues when required."

---

# 2. Linux Fundamentals

## 11. What is Linux?

**Answer:**

"Linux is an open-source operating system based on the Linux kernel. It is widely used for servers, cloud infrastructure, containers, networking, and application hosting.

I’m comfortable using Linux from the command line for tasks such as checking system resources, processes, services, logs, files, permissions, and network connectivity."

---

## 12. What is the difference between Linux and Windows Server?

**Answer:**

"Linux is generally command-line oriented, open source, and widely used for web servers, cloud infrastructure, containers, and DevOps environments. Windows Server is a Microsoft operating system and commonly uses GUI-based administration along with PowerShell.

Linux commonly uses tools such as systemctl, journalctl, grep, ps, top and SSH, while Windows environments commonly use PowerShell, Services, Event Viewer, and RDP."

---

## 13. What are commonly used Linux commands?

**Answer:**

"Some commands I commonly use are:

- `ls` for listing files
- `cd` for changing directories
- `pwd` for the current directory
- `cp` and `mv` for copying and moving files
- `rm` for removing files
- `cat` and `less` for viewing files
- `grep` for searching text
- `tail` for viewing the latest log entries
- `df` for filesystem usage
- `du` for directory/file usage
- `free` for memory
- `top` for processes and CPU
- `ps` for process information
- `systemctl` for services
- `journalctl` for systemd logs
- `ss` for network sockets
- `curl` for testing HTTP endpoints
- `ping` for basic connectivity."

---

## 14. How do you check CPU utilization in Linux?

**Answer:**

"I would start with `top` or `htop` to see overall CPU utilization and which processes are consuming CPU.

For example:

```bash
top
```

I can also use:

```bash
ps aux --sort=-%cpu | head
```

This helps identify the processes consuming the most CPU."

---

## 15. How do you check memory utilization?

**Answer:**

"I use `free -h` for an overall memory view.

```bash
free -h
```

I can also use `top` or `ps` to identify which processes are consuming the most memory."

---

## 16. How do you check disk space?

**Answer:**

"I use:

```bash
df -h
```

This shows filesystem-level disk utilization in a human-readable format.

If I need to find which directories are consuming space, I use `du`, for example:

```bash
du -sh /var/*
```

I would also check inode usage if disk space appears normal but applications are still reporting filesystem problems."

---

## 17. What is the difference between df and du?

**Answer:**

"`df` shows filesystem-level disk usage and available space.

`du` shows how much space individual files or directories are consuming.

For example, if `/var` is 95% full, I would use `du` to identify which directory is responsible."

---

## 18. How do you check running processes?

**Answer:**

"I can use:

```bash
ps aux
```

or:

```bash
top
```

I can also search for a particular process using:

```bash
ps aux | grep nginx
```

For production troubleshooting, I usually combine process information with CPU, memory, service status, and logs."

---

## 19. What is the difference between ps and top?

**Answer:**

"`ps` gives me a snapshot of currently running processes.

`top` provides a continuously updating view of processes and system resource usage such as CPU and memory.

So I would use `ps` for a quick process query and `top` when I want to observe system activity in real time."

---

## 20. How do you kill a process in Linux?

**Answer:**

"First I identify the process ID using `ps` or `top`.

Then I can gracefully terminate it using:

```bash
kill <PID>
```

If the process does not stop and I have authorization to force it, I can use:

```bash
kill -9 <PID>
```

I prefer graceful termination first because `kill -9` does not allow the process to clean up normally."

---

## 21. How do you check whether a service is running?

**Answer:**

"I use systemctl:

```bash
systemctl status nginx
```

This tells me whether the service is active, inactive, failed, and often shows recent service information."

---

## 22. What is systemctl?

**Answer:**

"`systemctl` is a command used to manage systemd services on Linux.

I can use it to check, start, stop, restart, enable, or disable services.

For example:

```bash
systemctl status nginx
systemctl restart nginx
systemctl enable nginx
```"

---

## 23. How do you restart a service?

**Answer:**

"I use:

```bash
sudo systemctl restart <service-name>
```

Before restarting a production service, I would first check the reason for failure, review the logs, follow the runbook, and make sure I’m authorized to perform the restart."

---

## 24. How do you check Linux system logs?

**Answer:**

"For systemd-based Linux systems, I use `journalctl`.

For example:

```bash
journalctl -u nginx
```

or:

```bash
journalctl -xe
```

I also check application logs under locations such as `/var/log`, depending on the application.

For real incidents, I usually check the logs around the exact time the problem started."

---

## 25. What is the difference between grep, awk, and sed?

**Answer:**

"`grep` is mainly used to search for matching text.

`awk` is useful for processing structured text and columns.

`sed` is commonly used for stream editing, such as replacing or modifying text.

For example:

```bash
grep "ERROR" app.log
```

would find error lines in a log file."

---

# 3. AWS Fundamentals

## 26. What is AWS?

**Answer:**

"AWS is Amazon Web Services, a cloud platform that provides services for compute, storage, networking, databases, security, monitoring, and many other workloads.

I have worked with AWS concepts such as EC2, S3, IAM, CloudWatch, networking, and infrastructure automation."

---

## 27. What major AWS services have you worked with?

**Answer:**

"I have worked with services and concepts including EC2, S3, IAM, CloudWatch, VPC and networking. I have also used infrastructure-as-code tools such as Terraform to provision and manage cloud infrastructure.

The exact services depend on the project, but I’m comfortable understanding the relationship between compute, networking, IAM, storage, and monitoring."

---

## 28. What is EC2?

**Answer:**

"Amazon EC2 is a service that provides virtual servers in AWS. We can choose the instance type, operating system, storage, networking, and security configuration based on the workload.

For support, I would commonly monitor EC2 instances for availability, CPU, memory where available, disk, network connectivity, application health, and system logs."

---

## 29. What is an AMI?

**Answer:**

"AMI stands for Amazon Machine Image. It is a template used to launch EC2 instances.

It can contain the operating system, application configuration, and other required software or settings."

---

## 30. What is an EC2 Security Group?

**Answer:**

"A Security Group is a virtual firewall associated with an EC2 instance or supported network interface.

It controls inbound and outbound traffic using rules based on protocols, ports, and sources or destinations.

For example, I may allow SSH on port 22 only from an approved source rather than exposing it to the entire internet."

---

## 31. Security Groups vs NACLs?

**Answer:**

"Security Groups are stateful and are associated with instances or network interfaces. NACLs are associated with subnets and are stateless.

Security Groups generally control traffic at the instance level, while NACLs provide an additional subnet-level network control layer."

---

## 32. What is S3?

**Answer:**

"Amazon S3 is an object storage service. It is commonly used to store files, backups, logs, static content, artifacts, and other objects.

S3 organizes data into buckets and objects and provides features such as access control, versioning, lifecycle management, and encryption."

---

## 33. What is IAM?

**Answer:**

"IAM stands for Identity and Access Management. It controls who or what can access AWS resources and what actions they are allowed to perform.

IAM includes users, groups, roles, policies, and permissions."

---

## 34. What is an IAM user?

**Answer:**

"An IAM user represents an identity in AWS with credentials and permissions. In modern AWS environments, I would prefer temporary credentials and IAM roles where possible instead of long-lived access keys."

---

## 35. What is an IAM role?

**Answer:**

"An IAM role is an identity that can be assumed by trusted entities such as AWS services, applications, or users.

Roles are commonly preferred for workloads because they provide temporary credentials instead of requiring applications to store permanent access keys."

---

## 36. What is the principle of least privilege?

**Answer:**

"Least privilege means giving a user, service, or application only the permissions it actually needs to perform its task and no more.

For example, if an application only needs to read objects from a particular S3 bucket, I would avoid giving it full S3 administrator permissions."

---

## 37. What is CloudWatch?

**Answer:**

"Amazon CloudWatch is an AWS monitoring and observability service. It can collect metrics, logs, events, and alarms.

For example, we can monitor EC2 CPU utilization, create alarms, collect application logs, and trigger notifications or automated actions based on conditions."

---

## 38. What can you monitor using CloudWatch?

**Answer:**

"I can monitor metrics such as EC2 CPU utilization, network traffic, disk-related metrics when configured, application metrics, load balancer metrics, database metrics, and many AWS service metrics.

CloudWatch can also collect logs and create alarms based on metric thresholds."

---

## 39. What is an AWS Region?

**Answer:**

"An AWS Region is a separate geographic area containing multiple AWS Availability Zones.

We choose a region based on factors such as latency, compliance, service availability, and business requirements."

---

## 40. What is an Availability Zone?

**Answer:**

"An Availability Zone is an isolated location within an AWS Region.

A region normally contains multiple Availability Zones, which allows applications to be designed for higher availability by distributing resources across zones."

---

## 41. EC2 server has high CPU. How would you troubleshoot it?

**Answer:**

"First, I would verify the alert in CloudWatch and check whether the high CPU is sustained or just a temporary spike.

Then I would connect to the server if access is available and run:

```bash
top
```

or:

```bash
ps aux --sort=-%cpu | head
```

I would identify the process consuming CPU and check application and system logs.

I would also check whether there was a recent deployment, traffic increase, scheduled job, or abnormal process.

If the issue is within my L1 procedure, I would follow the runbook. If it requires application changes, scaling, or deeper investigation, I would escalate to the appropriate team with the evidence collected."

---

## 42. An EC2 instance is unreachable. What would you check?

**Answer:**

"I would troubleshoot layer by layer.

First, I would confirm the instance state in AWS and check whether it is running.

Then I would check monitoring data, network connectivity, security group rules, NACLs, route tables, subnet configuration, and whether the correct public or private connectivity path is being used.

If SSH is the issue, I would check port 22 connectivity and the SSH service if I can access the machine through another method.

I would also check recent changes and system health. If I cannot resolve it within L1 access, I would escalate with the checks already completed."

---

## 43. How would you check whether an application on EC2 is working?

**Answer:**

"I would check it from multiple levels.

First, I would test the endpoint using `curl`.

For example:

```bash
curl -I https://example.com
```

Then I would check whether the application process is running, whether the expected port is listening using `ss`, whether the service is active using `systemctl`, and whether application logs show errors.

I would also check CPU, memory, disk, DNS, security group rules, load balancer health, and recent deployments depending on the architecture."

---

## 44. What would you do if a CloudWatch alarm is triggered?

**Answer:**

"First, I would acknowledge and verify the alert.

I would check the metric, threshold, timestamp, duration, affected resource, and whether it is a genuine incident or a temporary spike.

Then I would follow the relevant runbook, perform permitted L1 troubleshooting, document my actions, and escalate if the issue is beyond my scope or has significant business impact.

I would continue monitoring after the recovery and update the ticket with the final status."

---

## 45. Difference between stopped, terminated, and rebooted EC2 instance?

**Answer:**

"Stopped means the instance is shut down but can generally be started again later.

Reboot means the operating system is restarted while the instance remains in service.

Terminated means the EC2 instance is permanently deleted and cannot normally be started again.

The impact on storage and other resources depends on the specific configuration, so I would always verify before taking an action."

---

# 4. Networking

## 46. What is an IP address?

**Answer:**

"An IP address is a logical address used to identify a device or network interface on a network.

IPv4 addresses are 32-bit addresses, while IPv6 uses 128-bit addresses."

---

## 47. Public vs private IP?

**Answer:**

"A public IP can be reachable over the public internet subject to routing and security controls.

A private IP is used for internal network communication and is not directly routable over the public internet.

In cloud environments, private IPs are commonly used for communication between internal services."

---

## 48. What is DNS?

**Answer:**

"DNS stands for Domain Name System. It translates human-readable domain names such as `example.com` into IP addresses or other DNS records.

When troubleshooting an application, I may check DNS resolution using tools such as `nslookup` or `dig`."

---

## 49. How does DNS work?

**Answer:**

"When a client requests a domain, it normally checks its local cache first. If the answer is not cached, the request is handled by a DNS resolver.

The resolver can query the DNS hierarchy to find the authoritative server for the domain and return the required record, such as an A or AAAA record.

The client then uses the returned address to connect to the service."

---

## 50. What is DHCP?

**Answer:**

"DHCP stands for Dynamic Host Configuration Protocol. It automatically provides network configuration such as IP address, subnet mask, gateway, and DNS server information to clients."

---

## 51. What is a subnet?

**Answer:**

"A subnet is a logical subdivision of an IP network. It defines a range of IP addresses and helps organize and control network traffic.

In cloud platforms such as AWS, resources are placed inside subnets within a VPC."

---

## 52. What is a default gateway?

**Answer:**

"A default gateway is the network device or route used by a host to send traffic to destinations outside its local network."

---

## 53. What is a port?

**Answer:**

"A port is a logical endpoint used by network applications to identify a particular service.

For example, HTTP commonly uses port 80, HTTPS uses 443, SSH uses 22, and DNS commonly uses 53."

---

## 54. TCP vs UDP?

**Answer:**

"TCP is connection-oriented and provides reliable, ordered delivery with mechanisms such as retransmission.

UDP is connectionless and has lower overhead but does not provide the same delivery guarantees.

TCP is commonly used for HTTP/HTTPS and SSH, while UDP is used for use cases such as DNS queries and streaming or real-time traffic depending on the application."

---

## 55. What is HTTP?

**Answer:**

"HTTP is a protocol used for communication between clients and web servers.

It uses methods such as GET, POST, PUT, and DELETE and commonly runs on port 80."

---

## 56. What is HTTPS?

**Answer:**

"HTTPS is HTTP over TLS. It encrypts communication between the client and server and provides confidentiality and server authentication through certificates.

It commonly runs on port 443."

---

## 57. What are ports 22, 53, 80, and 443 used for?

**Answer:**

"Port 22 is commonly used for SSH.

Port 53 is used by DNS.

Port 80 is commonly used for HTTP.

Port 443 is commonly used for HTTPS."

---

## 58. What is SSH?

**Answer:**

"SSH stands for Secure Shell. It is a secure protocol used to remotely access and administer Linux and Unix systems.

It commonly uses TCP port 22."

---

## 59. Difference between ping, traceroute, and curl?

**Answer:**

"`ping` checks basic network reachability using ICMP.

`traceroute` or `tracepath` helps identify the network path and where packets may be failing or experiencing delay.

`curl` tests application-layer communication, such as HTTP or HTTPS.

So if a website is down, `ping` alone is not enough. I would also check DNS, port connectivity, and the HTTP response using `curl`."

---

## 60. How would you troubleshoot if a server is not reachable?

**Answer:**

"I would troubleshoot systematically.

First, I would confirm the server status.

Then I would check DNS resolution, IP address, routing, security group or firewall rules, port connectivity, and whether the required service is listening.

For example:

```bash
nslookup example.com
```

```bash
ping <IP>
```

```bash
ss -lntp
```

and:

```bash
curl -I http://example.com
```

I would also check monitoring and logs and escalate if the issue is outside my scope."

---

# 5. Monitoring

## 61. What is infrastructure monitoring?

**Answer:**

"Infrastructure monitoring means continuously observing the health and performance of servers, networks, applications, databases, and cloud resources.

The goal is to detect failures or abnormal behavior early and take corrective action before the business impact becomes larger."

---

## 62. Why is monitoring important?

**Answer:**

"Monitoring helps us detect issues quickly, reduce downtime, understand system health, identify performance problems, and meet availability and SLA requirements.

It also provides historical information that can help with troubleshooting and capacity planning."

---

## 63. What is an alert?

**Answer:**

"An alert is a notification generated when a monitoring condition or threshold is met.

For example, a monitoring system may generate an alert when CPU remains above 90% for a specified period."

---

## 64. What is an alarm?

**Answer:**

"An alarm is a defined monitoring condition that indicates something requires attention.

For example, a CloudWatch alarm can move into an alarm state when an EC2 CPU metric exceeds a configured threshold."

---

## 65. Monitoring vs logging?

**Answer:**

"Monitoring mainly focuses on metrics and system health, such as CPU, memory, latency, availability, and error rates.

Logging records detailed events and messages generated by systems and applications.

During troubleshooting, I normally use monitoring to identify when and where a problem occurred, and logs to understand what happened."

---

## 66. What metrics do you normally monitor on a server?

**Answer:**

"I would normally monitor CPU utilization, memory utilization, disk usage, disk I/O, network traffic, system load, process health, service availability, and application-specific metrics.

For production systems, I would also pay attention to availability, latency, error rates, and capacity trends."

---

## 67. What is CPU utilization?

**Answer:**

"CPU utilization indicates how much processing capacity is being used.

High CPU can be caused by heavy application traffic, inefficient processes, scheduled jobs, loops, or other workloads.

I would identify the process responsible before deciding on an action."

---

## 68. What is memory utilization?

**Answer:**

"Memory utilization indicates how much RAM is being used.

High memory usage can be caused by application workloads, memory leaks, caching, or insufficient resources.

I would check `free -h`, `top`, processes, swap usage, and application logs."

---

## 69. What is disk utilization?

**Answer:**

"Disk utilization shows how much filesystem storage is being consumed.

If disk usage becomes too high, applications may fail to write logs or files and the operating system can become unstable.

I would use `df -h` and then `du` to identify the source."

---

## 70. What would you do after receiving a critical monitoring alert?

**Answer:**

"First, I would acknowledge and verify the alert.

Then I would identify the affected resource, severity, business impact, and start time.

I would follow the relevant runbook, perform the allowed L1 troubleshooting, document everything in the ticket, communicate with the relevant team, and escalate according to the escalation matrix if required.

After recovery, I would verify the service is stable and continue monitoring before closing the incident."

---

## 71. CPU utilization is above 90%. What do you do?

**Answer:**

"I would first confirm whether it is a sustained high CPU condition or a short spike.

Then I would identify the process using CPU with `top` or `ps`, check application logs, recent deployments, traffic levels, scheduled jobs, and monitoring history.

I would follow the approved runbook. If a restart or scaling action is permitted, I would perform it carefully. Otherwise, I would escalate with the CPU graphs, process information, logs, and timeline."

---

## 72. Disk space is 95% full. What do you check?

**Answer:**

"I would first identify the affected filesystem using:

```bash
df -h
```

Then I would identify large directories using:

```bash
du -sh /var/*
```

I would check application logs, temporary files, backups, core dumps, container images, and other known storage consumers.

I would not randomly delete files. I would follow the cleanup procedure or runbook and escalate if approval is required."

---

## 73. Server-down alert at 2 AM. What do you do?

**Answer:**

"First, I would verify whether the alert is genuine.

I would check the monitoring system, try connectivity, verify the instance or server state, and check whether other related services are also affected.

If I have approved access, I would perform the initial checks and follow the server-down runbook.

Because it is a critical production incident, I would communicate and escalate according to the defined process rather than spending excessive time troubleshooting alone.

I would keep the ticket updated and monitor the recovery."

---

## 74. Monitoring says server is down but application team says it works. What do you do?

**Answer:**

"I would not assume either side is wrong.

I would verify the monitoring check itself and test the application independently.

I would check DNS, endpoint availability, port connectivity, service status, and monitoring-agent health.

It could be a monitoring false positive, a monitoring agent issue, or a problem affecting only a particular network path.

I would collect evidence from both sides and escalate if needed."

---

# 6. Ticketing & Incident Management

## 75. What is a ticketing system?

**Answer:**

"A ticketing system is used to record, track, prioritize, assign, communicate, and close incidents, service requests, and other support activities.

It provides visibility into the issue, SLA, ownership, actions taken, and final resolution."

---

## 76. Have you worked with ServiceNow, Jira, or another ticketing tool?

**Answer:**

"I have experience working with ticket-based operational workflows and issue tracking. I understand that regardless of the specific tool, the important parts are accurate categorization, priority, timestamps, clear updates, troubleshooting notes, ownership, escalation, and proper closure.

If your organization uses a different ticketing platform, I’m comfortable learning it quickly."

---

## 77. What information should be included in a support ticket?

**Answer:**

"I would include the affected system, issue description, alert or error message, time of occurrence, impact, priority, troubleshooting steps performed, command or monitoring results where appropriate, communication with other teams, escalation details, and resolution.

The ticket should be clear enough that another engineer can understand the incident without starting the investigation from zero."

---

## 78. What is an incident?

**Answer:**

"An incident is an unplanned interruption or degradation of an IT service.

For example, if a production application becomes unavailable, that would be treated as an incident."

---

## 79. What is a problem?

**Answer:**

"A problem is the underlying cause or potential cause of one or more incidents.

Incident management focuses on restoring service quickly, while problem management focuses on identifying and addressing the underlying root cause to prevent recurrence."

---

## 80. What is a change request?

**Answer:**

"A change request is a formal request to modify an IT environment, such as changing configuration, deploying a new version, modifying infrastructure, or applying a patch.

Changes should follow the organization's approval, testing, scheduling, and rollback procedures."

---

## 81. What is an SLA?

**Answer:**

"SLA stands for Service Level Agreement. It defines the expected service levels and response or resolution targets between a service provider and customer or business.

In support, SLAs are important because they help us prioritize work and ensure incidents are handled within agreed timelines."

---

## 82. Why are SLAs important?

**Answer:**

"SLAs help ensure that incidents receive the appropriate level of attention within an agreed timeframe.

They also create accountability and help the support team prioritize incidents based on urgency, business impact, and contractual requirements."

---

## 83. What is incident priority?

**Answer:**

"Priority determines how urgently an incident should be handled.

It is usually based on factors such as business impact, urgency, number of users affected, criticality of the service, and SLA requirements."

---

## 84. How do you decide P1, P2, P3, or P4?

**Answer:**

"I would follow the organization's priority matrix rather than deciding only based on my personal judgment.

Generally, P1 is a critical issue with major business impact, such as a production-wide outage.

P2 could be a significant service degradation or outage affecting an important group of users.

P3 is usually a lower-impact issue with a workaround available.

P4 is generally a minor issue or request.

The exact definitions depend on the company's incident policy."

---

## 85. Critical alert arrives while another ticket is close to SLA. Which do you handle first?

**Answer:**

"I would prioritize based on business impact, severity, urgency, and the organization's priority and SLA rules.

A critical production outage affecting many users would normally take priority over a lower-impact ticket, even if the other ticket is approaching its SLA.

I would also make sure the lower-priority ticket is updated or reassigned so that it does not get forgotten."

---

# 7. Troubleshooting & L1 Support

## 86. What is your general troubleshooting approach?

**Answer:**

"My approach is to first understand and verify the problem, then identify the scope and impact.

I usually troubleshoot layer by layer:

1. Verify the alert or user report.
2. Check availability and connectivity.
3. Check CPU, memory, disk, and system health.
4. Check process and service status.
5. Check ports and application endpoints.
6. Review logs and recent changes.
7. Follow the relevant runbook.
8. Apply an approved fix if it is within my scope.
9. Verify recovery.
10. Document and escalate if required.

I try to avoid making changes without evidence."

---

## 87. Server suddenly becomes slow. How do you troubleshoot?

**Answer:**

"I would first check whether the slowness is system-wide or application-specific.

Then I would check CPU, memory, load, disk I/O, disk space, network usage, and running processes using commands such as `top`, `free`, `df`, `du`, and `ps`.

I would check application and system logs and look for recent deployments, traffic increases, scheduled jobs, or resource exhaustion.

If the issue is not within L1 scope, I would escalate with the evidence collected."

---

## 88. How do you troubleshoot high memory usage?

**Answer:**

"I would start with:

```bash
free -h
```

Then use:

```bash
top
```

or:

```bash
ps aux --sort=-%mem | head
```

to identify memory-consuming processes.

I would check whether the usage is expected, whether swap is being heavily used, and whether logs show application errors or memory-related issues.

I would also check historical monitoring data to see whether memory usage is continuously increasing, which could indicate a memory leak."

---

## 89. How do you troubleshoot high CPU usage?

**Answer:**

"I would confirm the condition using monitoring and `top`.

Then I would identify the process consuming CPU using:

```bash
ps aux --sort=-%cpu | head
```

I would check logs, traffic, recent deployments, scheduled jobs, and whether the process behavior is expected.

I would follow the runbook for any restart or scaling action and escalate if required."

---

## 90. How do you troubleshoot low disk space?

**Answer:**

"I would run:

```bash
df -h
```

to identify the affected filesystem.

Then I would use `du` to identify large directories and check logs, temporary files, backups, container data, and other storage consumers.

I would clean only according to an approved procedure and verify disk usage afterward."

---

## 91. How do you troubleshoot a failed service?

**Answer:**

"I would first check:

```bash
systemctl status <service>
```

Then I would check the service logs:

```bash
journalctl -u <service>
```

I would check configuration, ports, dependencies, disk space, permissions, recent changes, and whether another process is already using the required port.

If the runbook allows it, I may restart the service after understanding the failure. Otherwise, I would escalate with the error details."

---

## 92. How do you troubleshoot a website that is not opening?

**Answer:**

"I would troubleshoot from the outside in.

First, I would check DNS resolution.

Then I would test connectivity and the application endpoint using `curl`.

I would check whether the required port is reachable, whether the web server is running, whether the application is healthy, and whether firewall or security group rules are blocking traffic.

I would also check load balancer health checks, certificates for HTTPS, server resources, and application logs depending on the architecture."

---

## 93. When should an L1 engineer escalate an issue?

**Answer:**

"I would escalate when the issue is outside my access or responsibility, when the runbook does not provide a safe resolution, when the incident requires L2 or L3 expertise, when a production change requires approval, or when the business impact and SLA require immediate escalation.

I would not simply say 'I cannot solve it.' I would provide the troubleshooting already completed, evidence, timeline, impact, and what I believe needs to happen next."

---

## 94. What information should you provide while escalating to L2/L3?

**Answer:**

"I would provide:

- A clear problem statement
- Affected server or application
- Environment
- Start time
- Business impact
- Alert details
- Relevant metrics
- Error messages
- Logs
- Commands or checks performed
- Changes already made
- Current status
- What assistance is required

This makes the escalation actionable and avoids repeating the same initial checks."

---

## 95. What would you do if you don't know how to resolve an issue?

**Answer:**

"I would stay calm and avoid making an unsafe change.

First, I would check the available SOP, runbook, knowledge base, monitoring information, logs, and previous incidents.

If I still cannot resolve it, I would escalate according to the process with all the relevant information I have collected.

For me, good support is not about pretending to know everything. It is about troubleshooting safely, communicating clearly, and getting the right team involved at the right time."

---

# 8. Real Interview Scenarios

## 96. Production server goes down at 3 AM. What will you do?

**Answer:**

"First, I would stay calm and verify the alert to make sure it is a genuine outage.

Then I would identify the affected server, application, environment, and business impact.

I would check whether the server is running, test connectivity, check monitoring metrics, and review relevant logs.

I would follow the documented server-down runbook and perform only the actions permitted for L1.

Because it is a production outage, I would immediately follow the escalation matrix and notify the appropriate L2 or on-call team if required.

Throughout the incident, I would keep the ticket updated with the timeline and actions taken.

Once the service is restored, I would verify application health, continue monitoring for stability, document the resolution, and close or hand over the incident according to the process."

**Short version to remember:**

"Verify → Assess impact → Troubleshoot → Follow runbook → Escalate → Communicate → Monitor → Document."

---

## 97. You receive 10 alerts simultaneously. How will you prioritize them?

**Answer:**

"I would not simply handle them in the order they arrived.

I would first group and correlate the alerts to determine whether multiple alerts are caused by one underlying problem.

Then I would prioritize based on severity, business impact, number of users affected, production versus non-production, criticality of the service, SLA, and dependencies.

For example, a production-wide outage would take priority over a low-impact disk warning on a development server.

I would also acknowledge the other alerts and make sure they are tracked or assigned so none are missed."

---

## 98. You fixed an issue, but the same alert comes back after 30 minutes. What will you do?

**Answer:**

"I would not immediately close the ticket again.

I would investigate why the alert returned. I would compare the current condition with the previous incident, check monitoring history and logs, and verify whether the previous fix only addressed the symptom.

I would look for an underlying cause such as a recurring process, resource leak, scheduled job, traffic pattern, or configuration problem.

I would update the ticket with the recurrence and escalate if a deeper investigation is required."

---

## 99. You are working alone during a night shift and a critical production issue occurs. What will you do?

**Answer:**

"I would remain calm and follow the incident procedure.

First, I would verify the alert and assess the business impact. Then I would follow the relevant runbook and perform the L1 troubleshooting that I am authorized to perform.

For a critical production issue, I would use the escalation matrix and contact the on-call L2 or L3 engineer without waiting too long.

I would keep the incident ticket updated, communicate important status changes, continue monitoring the affected service, and provide a clear handover if the incident continues into the next shift.

My priority would be safe recovery, clear communication, and following the defined process."

---

## 100. What if a critical incident happens and you cannot reach the primary escalation contact?

**Answer:**

"I would follow the documented escalation matrix rather than waiting indefinitely.

If the primary contact does not respond within the defined time, I would contact the secondary or next-level escalation contact.

I would document the escalation attempts and timestamps in the ticket and continue the approved L1 troubleshooting and monitoring.

For a critical incident, I would make sure the escalation path continues until the appropriate team takes ownership."

---

# 9. Extra Questions They May Ask Because of Your DevOps Background

## 101. What is Docker?

**Answer:**

"Docker is a containerization platform that packages an application and its dependencies into a container image.

Containers provide a consistent runtime environment and are lightweight compared with traditional virtual machines because they share the host operating system kernel."

---

## 102. What is Kubernetes?

**Answer:**

"Kubernetes is a container orchestration platform used to deploy, manage, scale, and maintain containerized applications.

It provides features such as deployments, services, pods, scheduling, self-healing, scaling, and service discovery."

---

## 103. A Docker container is not running. What would you check?

**Answer:**

"I would first check the container status:

```bash
docker ps -a
```

Then I would check the logs:

```bash
docker logs <container>
```

I would check the container exit code, configuration, environment variables, mounted volumes, port mappings, and whether the required image is available.

I would avoid immediately restarting it without understanding why it stopped."

---

## 104. A Kubernetes pod is in CrashLoopBackOff. What would you do?

**Answer:**

"I would first check the pod status and events:

```bash
kubectl get pod <pod-name>
kubectl describe pod <pod-name>
```

Then I would check the logs:

```bash
kubectl logs <pod-name>
```

If the container restarted multiple times, I would also check previous logs:

```bash
kubectl logs <pod-name> --previous
```

I would investigate configuration, environment variables, secrets, probes, resource limits, dependencies, and application errors.

If it is outside my L1 scope, I would provide these findings when escalating."

---

## 105. What is Terraform?

**Answer:**

"Terraform is an infrastructure-as-code tool used to define and manage infrastructure using configuration files.

It allows infrastructure to be version-controlled, repeatable, and automated.

A typical workflow is `terraform init`, `terraform plan`, and `terraform apply`, with appropriate review and approval processes."

---

## 106. What is CloudWatch vs Prometheus?

**Answer:**

"CloudWatch is AWS's native monitoring and observability service and is tightly integrated with AWS resources.

Prometheus is an open-source monitoring system commonly used for collecting and querying time-series metrics, especially in Kubernetes and cloud-native environments.

The choice depends on the environment and monitoring architecture. They can also be used together."

---

# 10. Quick Command Cheat Sheet

## Linux Resource Checks

```bash
uptime
top
free -h
df -h
du -sh /var/*
```

## Process Checks

```bash
ps aux
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
pgrep nginx
```

## Service Checks

```bash
systemctl status nginx
systemctl restart nginx
systemctl is-active nginx
journalctl -u nginx
```

## Logs

```bash
tail -f /var/log/application.log
tail -n 100 /var/log/application.log
grep "ERROR" /var/log/application.log
journalctl -xe
```

## Networking

```bash
ip addr
ip route
ss -lntp
ping <host>
curl -I https://example.com
curl -v https://example.com
nslookup example.com
dig example.com
```

## Disk

```bash
df -h
df -i
du -sh /var/*
```

---

# 11. Interview Scenario Framework

When you get almost any troubleshooting question, use this structure:

### 1. Verify
"First, I would verify whether the alert or issue is genuine."

### 2. Assess Impact
"I would identify the affected service, environment, users, and business impact."

### 3. Check Basic Health
"I would check connectivity, CPU, memory, disk, processes, and service status."

### 4. Check Logs
"I would review system and application logs around the time the issue started."

### 5. Check Recent Changes
"I would check recent deployments, configuration changes, maintenance, or traffic changes."

### 6. Follow the Runbook
"I would follow the approved SOP/runbook and perform only the actions within my L1 scope."

### 7. Escalate
"If the issue is outside my scope or requires deeper investigation, I would escalate with complete evidence."

### 8. Verify Recovery
"After the fix, I would verify that the service is healthy and monitor it for stability."

### 9. Document
"I would update the ticket with the timeline, actions, findings, escalation, and resolution."

---

# 12. Important Interview Phrases

Use these naturally during the interview:

- "First, I would verify the alert."
- "I would assess the business impact."
- "I would follow the documented runbook."
- "I would avoid making an unapproved production change."
- "I would check the logs around the time of failure."
- "I would correlate the alert with monitoring metrics."
- "I would check whether there was a recent deployment or configuration change."
- "If it is outside my L1 scope, I would escalate with complete troubleshooting details."
- "I would keep the ticket updated throughout the incident."
- "After recovery, I would continue monitoring to make sure the issue does not recur."

---

# 13. Final 1-Day Revision Priority

If you have limited preparation time, revise in this order:

1. **Linux commands**
2. **CPU, memory, disk troubleshooting**
3. **systemctl and journalctl**
4. **AWS EC2**
5. **CloudWatch**
6. **Security Groups and networking**
7. **DNS, HTTP, HTTPS, SSH**
8. **Monitoring alerts**
9. **Incident priority and SLA**
10. **Production server-down scenario**
11. **Multiple alerts scenario**
12. **Night-shift critical incident**
13. **Ticket documentation and escalation**
14. **Docker/Kubernetes basics**
15. **HR introduction**

---

# 14. 30-Second Introduction to Memorize

"Hi, I’m Vinayak. I have around 3+ years of experience in DevOps and cloud infrastructure, with hands-on experience in Linux, AWS, Azure, Docker, Kubernetes, Terraform, CI/CD, and monitoring.

My day-to-day work involves infrastructure troubleshooting, deployments, monitoring, and resolving issues related to servers and applications. I’m comfortable checking system resources, services, logs, networking, and application health.

I’m interested in this role because I want to apply my technical troubleshooting experience in a structured 24×7 support environment, strengthen my incident management skills, and continue growing in cloud and infrastructure operations."

---

# 15. Final Interview Mindset

For an L1 Infrastructure Monitoring interview, the interviewer is usually not expecting you to solve every complex production problem yourself.

They want to see that you can:

- Detect and verify problems.
- Stay calm during incidents.
- Understand severity and business impact.
- Perform safe initial troubleshooting.
- Follow SOPs and runbooks.
- Understand Linux and cloud basics.
- Communicate clearly.
- Maintain accurate tickets.
- Respect SLAs.
- Escalate at the right time.
- Verify recovery.
- Monitor after recovery.
- Document what happened.

**The strongest L1 answer is usually not "I will fix everything myself."**

A stronger answer is:

> "I will verify the issue, assess the impact, perform the approved L1 checks, follow the runbook, escalate with complete evidence when required, keep the ticket updated, and verify the service after recovery."

