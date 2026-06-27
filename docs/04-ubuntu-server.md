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


