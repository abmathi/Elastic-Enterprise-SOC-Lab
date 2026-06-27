| VM Name      | OS Type                                    |     RAM | CPUs |                                Disk |
| ------------ | ------------------------------------------ | ------: | ---: | ----------------------------------: |
| `SOC-UBU01`  | Ubuntu (64-bit)                            | 8192 MB |    4 | 150 GB (VDI, Dynamically Allocated) |
| `SOC-DC01`   | Windows Server 2022 (64-bit)               | 8192 MB |    4 | 100 GB (VDI, Dynamically Allocated) |
| `SOC-WS01`   | Windows 11 (64-bit)                        | 8192 MB |    4 |  80 GB (VDI, Dynamically Allocated) |
| `SOC-KALI01` | Debian/Linux (64-bit) or Kali if available | 4096 MB |    2 |  60 GB (VDI, Dynamically Allocated) |


ubu01 is used for the elastic and kibana server
dc01 is used for the domain controller machine
ws01 is used for the workstation machine
kali01 is used for kali machine

using nat and internal network to us Internet access for package downloads while keeping all lab communication on an isolated network.