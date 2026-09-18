# Fleet and Elastic Agent Architecture

## Elastic Agent

Agent that runs on an endpoint to collect telemetry like logs, metrics, and security data and sends it to Elastic. Can also add other integrations like security monitoring.

## Fleet

Allows for centralized management of Elastic Agents. Can configure and monitor your agents all from one place.

## Fleet Server

The management gateway between Fleet and Elastic Agents. Agents report their status and policies to Fleet Server.

## Agent Policy

A central configuration that defines what an agent should collect and how.

## Control Plane

Handles management traffic like policies and configurations.

## Data Plane 

Handles the actual telemetry data collected by agents like logs, metrics, and events. 

## Lab Architecture Decision

We’re hosting Fleet Server on SOC-UBU01 rather than creating another VM to keep the deployment simple and lightweight.

## Fleet Network Design

| Service | Lab address | Port | Purpose |
|---|---|---:|---|
| Elasticsearch | `10.10.10.10` | 9200 | Agent telemetry/data output |
| Fleet Server | `10.10.10.10` | 8220 | Agent management/control plane |
| Kibana | `192.168.56.10` | 5601 | Administrative UI from host |

# Fleet and Elastic Agent Architecture

## Overview

Elastic Fleet provides centralized management for Elastic Agents deployed throughout the SOC lab. Rather than configuring each endpoint individually, Fleet allows agent policies, integrations, enrollment, and health to be managed from Kibana.

The lab uses three related components:

- **Elastic Agent** collects telemetry from an endpoint and sends data to the configured Elasticsearch output.
- **Fleet** provides the centralized management interface within Kibana.
- **Fleet Server** provides the management communication path between Fleet and enrolled Elastic Agents.

This creates a separation between the management (control) plane and telemetry (data) plane.

## Control Plane vs. Data Plane

The **control plane** is responsible for agent management. Elastic Agents communicate with Fleet Server to receive policies and report their status.

The **data plane** carries the security telemetry collected by Elastic Agent to Elasticsearch.

In this lab:

| Plane | Destination | Port | Purpose |
|---|---|---:|---|
| Control | Fleet Server (`10.10.10.10`) | 8220 | Agent management and policy communication |
| Data | Elasticsearch (`10.10.10.10`) | 9200 | Security telemetry ingestion |

A healthy Fleet connection therefore does not necessarily prove that telemetry ingestion is working. An agent could successfully communicate with Fleet Server while experiencing a separate problem sending data to Elasticsearch.

## Lab Architecture Decision

Fleet Server is hosted on `SOC-UBU01` alongside Elasticsearch and Kibana.

This design was chosen because the lab contains only a small number of endpoints and does not require dedicated infrastructure for each Elastic component. A larger production environment could separate Elasticsearch, Kibana, and Fleet Server to support additional scalability, availability, security boundaries, and resource requirements.

The resulting architecture is:

                         SOC-UBU01
                        10.10.10.10
                             |
              +--------------+--------------+
              |                             |
       Fleet Server                    Elasticsearch
          :8220                           :9200
      CONTROL PLANE                    DATA PLANE
              ^                             ^
              |                             |
       +------+-------+              +------+-------+
       |              |              |              |
    SOC-DC01       SOC-WS01       SOC-DC01       SOC-WS01
 Elastic Agent   Elastic Agent    Telemetry      Telemetry


## Fleet Network Design

The Ubuntu SIEM server contains multiple network interfaces with different purposes:

| Service | Address | Port | Purpose |
|---|---|---:|---|
| Elasticsearch | `10.10.10.10` | 9200 | Elastic Agent telemetry output |
| Fleet Server | `10.10.10.10` | 8220 | Elastic Agent management |
| Kibana | `192.168.56.10` | 5601 | Administrative access from physical host |

The `10.10.10.10` address was selected for Fleet and Elasticsearch because it belongs to the isolated `SOC-LAB-NET` network that will be shared by the lab endpoints.

The Ubuntu NAT address (`10.0.2.15`) is used for Internet connectivity but is not the intended communication path between SOC systems. The Host-Only address (`192.168.56.10`) is reserved for administrative access from the physical Windows host.

## Secure Agent Connectivity

Three separate conditions must be satisfied for an Elastic Agent to establish a secure connection to Elasticsearch:

1. **Network reachability** — the client must be able to reach `10.10.10.10:9200`.
2. **Certificate trust** — the client must trust the certificate authority that issued the Elasticsearch HTTP certificate.
3. **Certificate identity** — the certificate must identify the destination the client intended to contact.

Elasticsearch was verified to listen on port 9200 across its interfaces:

    *:9200

Its configuration contains:

    http.host: 0.0.0.0

An authenticated HTTPS request to `https://10.10.10.10:9200` succeeded.

The Elasticsearch HTTP certificate was then inspected with OpenSSL. Its Subject Alternative Name (SAN) entries include `10.10.10.10`, confirming that the certificate identifies the SOC-LAB-NET address.

Full CA and IP identity verification was also performed with OpenSSL and returned:

    Verification: OK
    Verify return code: 0 (ok)

This demonstrated that the Elasticsearch data-plane destination was reachable and cryptographically valid before deploying Fleet Server.

## Troubleshooting: Fleet Unable to Initialize

When Fleet was first opened in Kibana, initialization failed with the following error:

    Unable to initialize Fleet
    Agent binary source needs encrypted saved object api key to be set

The issue was traced to Kibana not having a persistent encrypted saved objects key configured.

A strong encryption key was generated using Kibana's encryption-key utility and configured as:

    xpack.encryptedSavedObjects.encryptionKey

The actual key is intentionally excluded from this repository.

After restarting Kibana, Fleet initialized successfully and all Fleet management pages became available.

This demonstrated the distinction between several security mechanisms used by Elastic:

- **TLS certificates** protect communications in transit and establish server identity.
- **Authentication credentials and service tokens** establish the identity and permissions of users and services.
- **Kibana's encrypted saved objects key** protects sensitive properties stored by Kibana.

## Troubleshooting: Incorrect Default Elasticsearch Output

After Fleet initialized, its default Elasticsearch output was:

    https://10.0.2.15:9200

This was the Ubuntu VM's VirtualBox NAT address rather than its dedicated SOC-LAB-NET address.

The output could not be edited or deleted through Fleet.

Inspection of `/etc/kibana/kibana.yml` revealed that the output had been preconfigured using `xpack.fleet.outputs`. Because the output was managed through Kibana configuration, Fleet correctly displayed it as read-only.

The Fleet output was changed in the Kibana configuration from:

    https://10.0.2.15:9200

to:

    https://10.10.10.10:9200

Kibana's own `elasticsearch.hosts` setting was intentionally left unchanged because the existing Kibana-to-Elasticsearch connection was already functioning correctly.

After restarting Kibana, the default Fleet output displayed:

    https://10.10.10.10:9200

The output remained locked in Fleet because it continued to be managed through `kibana.yml`, which is expected behavior.

The main lesson was that a service can be healthy and reachable locally while still advertising an inappropriate address to its intended clients.

## Fleet Server Deployment

Fleet Server was deployed on `SOC-UBU01` using Elastic Agent 9.4.3.

Before installation, the generated Fleet command was reviewed rather than immediately executed. Kibana initially selected the Linux ARM64 package. Because `SOC-UBU01` runs on an x86-64 virtual machine, the platform selection was corrected to Linux x86_64 before installation.

The generated Fleet Server configuration used:

    Elasticsearch: https://10.10.10.10:9200
    Fleet Server:  https://10.10.10.10:8220

The generated Fleet Server service token and other sensitive enrollment material were not captured in screenshots or stored in this repository.

Fleet Server installation completed successfully.

The deployment was validated at multiple layers:

- `systemctl` confirmed the Elastic Agent service was running.
- `elastic-agent status` reported healthy components.
- `ss` confirmed a listener on TCP port 8220.
- Kibana Fleet reported the Fleet Server agent as Healthy.
- Fleet Server Hosts contained `https://10.10.10.10:8220`.

These checks confirmed both the local service state and Fleet's centralized view of the server.

## Current State

The central Elastic infrastructure is now operational:

    SOC-UBU01 (10.10.10.10)
    |
    +-- Elasticsearch :9200
    |     Security telemetry / data plane
    |
    +-- Fleet Server  :8220
    |     Agent management / control plane
    |
    +-- Kibana        :5601
          Analyst and administration interface

The next stage of the lab is to deploy Windows systems and begin onboarding endpoint telemetry through Elastic Agent.

