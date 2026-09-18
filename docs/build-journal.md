# Enterprise SOC Lab Build Journal

## Entry 001 - Repository Initialization

### Date

6/26/2026

### Objective

Create the project repository and establish the initial folder structure.

### Work Completed

- Created public GitHub repository.
- Created documentation folders.
- Created screenshot folders.
- Performed initial commit.

### Challenges

None.

### Lessons Learned

Version control should be established before infrastructure is deployed so every change is tracked from the beginning.

---

## Entry 002 - Architecture Planning

### Date

6/26/2026

### Objective

Plan the lab architecture before deployment.

### Work Completed

- Selected virtual machines.
- Designed IP addressing.
- Planned network topology.
- Documented architecture.

### Challenges

None.

### Lessons Learned

Planning infrastructure first prevents inconsistent configurations later in the project.

---

## Entry 003 - Virtual Machine Creation

### Date

6/26/2026

### Objective

Create the virtual machines and networking configuration.

### Work Completed

- Created four VirtualBox VMs.
- Configured NAT networking.
- Configured Internal Network.
- Assigned hardware resources.

### Challenges

None.

### Lessons Learned

Separating Internet access from internal enterprise communication closely mirrors real-world network architecture.

## Fleet and Agent Management

- Studied the roles of Fleet, Fleet Server, and Elastic Agent and distinguished the control plane from the telemetry data plane.
- Verified Elasticsearch was reachable on the isolated SOC network at `10.10.10.10:9200`.
- Encountered a Fleet initialization failure caused by a missing Kibana encrypted saved objects key.
- Generated and configured a persistent encryption key, restarted Kibana, and confirmed Fleet initialized successfully.
- Discovered the default Fleet Elasticsearch output incorrectly referenced the VM's NAT address (`10.0.2.15`).
- Determined the output was locked because it was preconfigured through `xpack.fleet.outputs` in `kibana.yml`.
- Changed the agent Elasticsearch output to the SOC-LAB-NET address (`10.10.10.10:9200`) while retaining Kibana's existing Elasticsearch connection.
- Inspected the Elasticsearch HTTP certificate and confirmed `10.10.10.10` was included in its SAN entries.
- Verified CA trust and IP identity with OpenSSL.
- Caught an ARM64 package selection before Fleet Server installation and corrected it to Linux x86_64.
- Installed Fleet Server on `SOC-UBU01`.
- Verified Elastic Agent health, TCP port 8220, Fleet Server registration, and Healthy status in Kibana.

**Result:** The Elastic management and data-plane infrastructure is ready for endpoint onboarding.