# Ubuntu Server Installation

## Purpose

SOC-UBU01 will host the Elastic Stack and serve as the central Security Information and Event Management (SIEM) server for the lab.

## Why Ubuntu Server?

- Stable Long-Term Support release
- Widely used in enterprise environments
- Minimal attack surface
- Efficient resource utilization
- Excellent compatibility with Elastic Stack

## Planned Components

- Elasticsearch
- Kibana
- Fleet Server

## Installation Decisions

### Language

English

**Reason:** Standard language used throughout enterprise environments and Elastic documentation.

---

### Keyboard

English (US)

**Reason:** Matches the physical keyboard and prevents issues when using Linux command-line tools and SSH.

---

### Installation Type

Ubuntu Server (Standard)

**Reason:** Includes common administration tools and provides a smoother learning experience while maintaining a server-focused environment.

---

# Installation Decisions (Part 2)

## Network Configuration

### Lab Decision

Configured both network interfaces to use DHCP during the installation.

### Reason

Using DHCP during the installation simplifies the deployment process and allows the operating system to install without requiring manual network configuration. Static IP addresses will be configured after installation using Linux networking tools.

### Enterprise Consideration

Production servers commonly use static IP addresses or DHCP reservations for infrastructure services. Configuring networking after installation provides greater control and helps administrators verify connectivity before making permanent network changes.

### SOC Relevance

Understanding network interface configuration is essential when troubleshooting log ingestion failures, Elastic Agent connectivity issues, or communication problems between servers.

---

## Proxy Configuration

### Lab Decision

No proxy server was configured.

### Reason

The home lab has direct Internet access through the NAT adapter, so an outbound proxy is unnecessary.

### Enterprise Consideration

Many organizations require outbound traffic to pass through authenticated proxy servers for logging, malware inspection, and policy enforcement. Security tools often require additional configuration to function correctly behind a proxy.

### SOC Relevance

SOC analysts frequently investigate connectivity issues caused by incorrect proxy settings. Understanding whether a proxy exists can significantly reduce troubleshooting time.

---

## Ubuntu Package Mirror

### Lab Decision

Used the default Ubuntu package mirror selected by the installer.

### Reason

Official Ubuntu mirrors provide reliable access to operating system updates and software packages required throughout the project.

### Enterprise Consideration

Large organizations often maintain internal package repositories or mirrors to reduce Internet usage, improve download speeds, and ensure all systems install approved software versions.

### SOC Relevance

Security engineers regularly install software, security updates, and forensic utilities using package repositories. Repository availability directly affects the ability to deploy or update security tooling.

---

## Storage Configuration

### Lab Decision

Selected the default storage layout using the entire virtual disk without enabling encryption.

### Reason

The default partition layout provides a straightforward installation suitable for learning Linux administration while avoiding unnecessary complexity early in the project.

### Enterprise Consideration

Production servers may implement Logical Volume Manager (LVM), RAID, encrypted disks, or dedicated partitions for directories such as `/var`, `/home`, and `/tmp` depending on operational and security requirements.

### SOC Relevance

SOC analysts frequently investigate storage-related issues such as full disks preventing log ingestion, failed Elasticsearch indexing, or services that stop because critical partitions have reached capacity.

---

## Hostname

### Lab Decision

Configured the hostname as **SOC-UBU01**.

### Reason

A standardized hostname clearly identifies the system's role within the lab and follows a consistent naming convention that can scale as additional systems are added.

### Enterprise Consideration

Organizations often implement standardized naming conventions that identify business units, geographic regions, operating systems, or system roles to simplify asset management and incident response.

### SOC Relevance

Hostnames appear throughout SIEM alerts, dashboards, detection rules, and incident investigations. Consistent naming significantly improves analyst efficiency.

---

## Administrative User

### Lab Decision

Created a standard administrative account named **austin** instead of using the root account for daily administration.

### Reason

Using a dedicated administrative account follows Linux security best practices and reduces the likelihood of accidental system-wide changes.

### Enterprise Consideration

Enterprise Linux environments commonly integrate centralized identity providers such as Active Directory, LDAP, or Entra ID while restricting direct root logins through SSH.

### SOC Relevance

Authentication logs record the username performing administrative actions. Individual user accounts provide accountability, improve auditing, and simplify incident investigations.

---

## Ubuntu Pro

### Lab Decision

Skipped Ubuntu Pro during installation.

### Reason

The standard Ubuntu LTS repositories provide all required packages for this project without introducing additional licensing or configuration complexity.

### Enterprise Consideration

Organizations requiring extended security maintenance, compliance features, or Livepatch capabilities may subscribe to Ubuntu Pro for production servers.

### SOC Relevance

Understanding operating system support lifecycles helps analysts evaluate vulnerability management, patching strategies, and compliance requirements.

---

## OpenSSH

### Lab Decision

Installed the OpenSSH Server package.

### Reason

SSH enables secure remote administration and reflects how Linux servers are typically managed in enterprise environments.

### Enterprise Consideration

Production SSH deployments often include key-based authentication, centralized identity management, multi-factor authentication, and hardened SSH configurations.

### SOC Relevance

SOC analysts and security engineers frequently use SSH to investigate incidents, review logs, verify system health, and perform administrative tasks on Linux infrastructure.

---

## Optional Packages

### Lab Decision

Did not install any optional server packages.

### Reason

Only required software will be installed throughout the project to minimize unnecessary services and maintain a controlled learning environment.

### Enterprise Consideration

Production servers generally follow a minimal installation philosophy to reduce maintenance requirements and the system's attack surface.

### SOC Relevance

Every installed service represents another potential source of vulnerabilities and telemetry. Analysts should understand which services are expected to exist on a server.


---

# Post-Installation Validation

## Hostname Verification

### Command

```bash
hostname
```

### Purpose

Verify that the server hostname matches the planned naming convention.

### Expected Result

`SOC-UBU01`

### SOC Relevance

Hostnames appear throughout SIEM dashboards, detection rules, alerts, and investigations. Consistent naming improves incident response efficiency.

---

## Operating System Verification

### Command

```bash
cat /etc/os-release
```

### Purpose

Verify the installed Ubuntu version.

### SOC Relevance

Knowing the operating system version is important for software compatibility, vulnerability management, and incident response.

---

## Kernel Verification

### Command

```bash
uname -r
```

### Purpose

Verify the installed Linux kernel.

### SOC Relevance

Kernel versions determine available security features, driver compatibility, and vulnerability exposure.

---

## User Verification

### Command

```bash
whoami
```

### Purpose

Confirm that administrative work is being performed from a standard user account.

### SOC Relevance

Using individual administrative accounts improves accountability and auditability during investigations.


---

## Check Disk Usage

### Command

```bash
df -h
```

### Purpose 

Confirms that disk usage is at normal operational numbers. 

### SOC Relevance

Elastic stores logs on disks and if disk usage reaches 100% then Elasticsearch stops indexing, alerts stop firing, and investigation becomes impossible. 

---

## Check Memory

### Command

```bash
free -h
```

### Purpose 

Ram is where programs live and Elasticsearch is very memory hungry. Knowing how much memory Linux sees is critical. 

### SOC Relevance

A server running out of RAM can cause Kibana crashes, Elasticsearch crashes, Agent failures, and search slowdowns.  

---

## Check CPU Information

### Command

```bash
lscpu
```

### Purpose 

This command reports CPU model, virtual CPUs, architecture, threads, and cores

### SOC Relevance

If Elasticsearch is performing poorly, CPU is one of the first resources to investigate. 

---

## Check IP Addresses

### Command

```bash
ip addr
```

### Purpose 

This command shows your network interfaces and related information to them such as IP adresses, and DHCP information. 

### SOC Relevance

Having a properly set up IP configuration is vital to the entire ecosystem.

---

## Verify Internet Connectivity

### Command

```bash
ping -c 4 8.8.8.8
```

```bash
ping -c google.com
```

### Purpose 

These commands test the connection to the network and DNS of the system.

### SOC Relevance

The first ping only tests network connectivity. The second ping tests both network connectivity and DNS. If the IP ping works but the hostname fails then you know the network is fine but DNS is broken.

---

### Command

```bash
ip route
```

### Result

This command displays the current routing table.

### Why It Matters

The server can have multiple interfaces; the routing table shows Linux which route to send internet traffic through. 

### SOC Relevance

Knowing how to check IP routing is a key troubleshooting step in any IT environment.

---

### Command

```bash
ls /etc/netplan
```

### Result

This command lists the contents of the netplan directory.

### Why It Matters

Netplan is like the blueprint that tells Ubuntu how networking should look when the machine starts. 

### SOC Relevance

Knowing where your configuration files live ahead of time can save crucial time in the troubleshooting process.

---

### Command

```bash
sudo cat /etc/netplan/*.yaml
```

### Result

This command shows you what's inside of the yaml file.

### Why It Matters

This file can show you what interfaces are connected and whether DHCP is enabled. 

### SOC Relevance

Being able to read and know what the configuration file means is just as important as knowing how to find it.

---

## Network Interface Discovery

### Observation

The intiial Netplan configuration only contained the NAT interface.

### Reason

Ubuntu automatically generated a configuration only for the interface that recieved a DHCP lease. 

### Why It Matters

Infrastructure changes should be based on verified system information. Before assigning static IP addresses, all interfaces should be identified and validated. 

### SOC Relevance

Connectivity issues can be caused by incorrect interface selections or assumptions about network configuration. Verifying interface names ahead of time reduces configuration errors. 

---

## Netplan Static IP Configuration

### Concept 

Netplan is how Ubuntu knows how networking should look when the machine starts. Linux uses configuration files for networking due to the ease of applying edits and linking to other services that may use them. Automation is another main reason because in enterprise environments you don't have just one server, you may have thousands of servers. Nobody opens a GUI 5,000 times when you can write a script to push it instantly.

### Lab Decision

Added a static IP address to enp0s8 while leaving enp0s3 on DHCP.

### Reason 

Preserve internet access while giving the internal enterprise network a predicatable address.

### Enterprise Consideration 

Infrastructure servers typically use static addresses or DHCP reservations to ensure consistent connectivity.

### SOC Relevance

Stable IP addressing simplifies SIEM configuration, log forwarding, agent enrollment, and incident response.

---

## Configuration Verification

### Command 

```bash
sudo netplan generate
```

### Purpose

Netplan generate validates the updated configuration file and generates the backend networking configuration without making any live changes.

### Expected Result

When running this command you should expect there to be no output unless there is a problem with your configuraration file, such as a syntax error.

### Result

There was no error message after running the command so our configuration file is working.

### SOC Relevance

Properly configuring your netplan configuration file will avoid throwing any errors when assigning a static IP to the interface.

---

### Command

```bash
sudo netplan apply
```

### Purpose

After generating your updated configuration file, the apply command confirms and applies the changes.

### Expected Result

This command should also produce no output if there are no errors with your configuration file.

### Result

Running this command returned no error messages so our configuration looks good to go.

### SOC Relevance

Being able to sucessfully configure the network configuration of your SIEM server is the first step in avoiding problems while trying to build the environment. 

---

### Command

```bash
ip addr
```

### Purpose

This command is then run to confirm that the changes we made actually took place.

### Expected Result

We should now expect the internal adapter to have the static IP address that we configured. 

### Result

Running this command confirmed that the internal interface now has the static IP address that we assigned it. 

### SOC Relevance

It is vital that you double check and confirm the changes instead of just assuming that you did it correctly in order to save time and avoid mistakes.

---

### Command

```bash
ip route
```

### Purpose

This command was ran to confirm that the default route is still the NAT.

### Expected Result

We expect that the default route is still the NAT because assigning a static IP does not automatically change the route.

### Result

Upon running the command we were able to confirm that the NAT is the default route as expected.

### SOC Relevance

Double checking your work instead of relying on the assumptions assures proper configuration and no unexpected errors occur.

---

### Command

```bash
ping -c 4 8.8.8.8
```

```bash
ping -c google.com
```

### Purpsose

Lastly, pinging allows us to confirm that the change works as it should. 

### Expected Result

Since we double checked all the changes we made, it is expected that we should have sucessful pings.

### Result

And just as expected, our ping was successful which confirms we changed the networking without breaking the server.

### SOC Relevance

Taking the time to go back and check the work made after each step is a valuable habit to create that can help to avoid a lot of headache when troubleshooting problems.

---

## Server Readiness Validation

### Command

```bash
sudo apt update
```

### Prediction

Before running this command I expected it to install the available updates.

### Purpose 

This command is actually used to update the package information.

### Result

Running this command told me that 39 packages can be upgraded.

### Why It Matters

It's important to be able to see the available updates before just automatically running the actual upgrade command as it gives you an idea of the changes that will be made. 

### SOC Relevance

This is important for security and IT engineers because one tiny change can bring down the whole system. Being sure of the changes you're making will save you a lot of work and headache in the long run.

---

### Command

```bash
sudo apt upgrade -y
```

### Prediction

After learning that the previous command is the one that checks for updates, I was quite certain before running this one that it would be the one to initiate the actual update.

### Purpose

As expected, this command is used to upgrade all the package that have available updates. The -y confirms you want to make that decison without having to answer another following prompt.

### Result

Running this command successfully updated all 39 packages that we had available updates for.

### Why It Matters

This matters because we want a fully updated system before we start introducing Elasticsearch into the environment.

### SOC Relevance

This is an important step for engineers because there's a good chance of breaking things if you start introducing new tools into an outdated system, especially if those tools require certain versions of packages.

---

### Command

```bash
timedatectl status
```

### Prediction

I predicted that this command would be used to give more info on the time configuration of the system.

### Purpose

This command is used to ensure that your systems time is synchronized.

### Result

The output of this command showed that the date and time were synchronized it UTC format and that NTP service is running.

### Why It Matters

Having times synchronized across all systems is a vital part of incident response. Logs are only useful if their timestamps are trustworthy.

### SOC Relevance

Imagine you're investigating an attack and one log says 14:01, another log says 13:58, another is 14:05. Was the attacker moving through the system or are the clocks just simply different? 

---

### Command

```bash
systemctl --failed
```

### Prediction

My prediction for this command was that it would be checking for any failures in system programs or services.

### Purpose

This command is used to show if you have any failed services.

### Result

The command confirmed that this system has 0 failed services. 

### Why It Matters

Before installing anything you want to know if the operating system already has existing problems.

### SOC Relevance

You don't want to install new software or services if your machine already has problems as you could possibly be adding to them. Knowing the health status of your system before and after changes are being made is key when it's time to troublshoot problems.

---

### Command

```bash 
uptime
```

### Prediction

My prediction for this command was that it's to display how long the system has been running.

### Purpose

This command can be used to show how long the system has been up, how many users are logged in, and how much load is on the system.

### Result

Running the command showed us that the system has just started up after a reboot, has one user signed in, and a small load.

### Why It Matters

This is important information because it allows you to get a feel for the overall status of your system.

### SOC Relevance

This allowed us to establish a baseline for the system before we start installing more tools and services such as Elasticsearch.

---

### Command

### Prediction

### Result

### Why It Matters

### SOC Relevance

---