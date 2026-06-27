# Network Design

## Planned Subnet

10.10.10.0/24

## Planned Hosts

| Host | Planned IP |
|------|------------|
| DC01 | 10.10.10.5 |
| Ubuntu | 10.10.10.10 |
| WS01 | 10.10.10.100 |
| Kali | 10.10.10.200 |

## Design Rationale

A dedicated RFC1918 subnet provides a clean, isolated environment that closely resembles enterprise network segmentation. Static IP addressing simplifies server configuration, log collection, and troubleshooting throughout the project.