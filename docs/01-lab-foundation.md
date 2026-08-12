# 01 — Lab Foundation

## 1. Purpose

The first phase establishes the Azure foundation for the SOC learning environment.

The goal is not to build a complete enterprise SOC in one step. Instead, the environment is being built incrementally so that each Azure service and security control can be understood, validated, and documented before the next layer is introduced.

## 2. Initial Design

The initial environment consists of:

- A dedicated Azure resource group.
- A Log Analytics workspace.
- Microsoft Sentinel connected to the workspace.
- A dedicated virtual network and subnet.
- A Windows endpoint VM.
- A Network Security Group controlling inbound access.
- Azure Monitor Agent and a Data Collection Rule for Windows Security telemetry.

### Resource inventory

| Resource | Name | Purpose |
|---|---|---|
| Resource Group | `RG-SOC-LAB` | Logical container for lab resources |
| Log Analytics Workspace | `LAW-SOC-LAB` | Central telemetry/log storage |
| Microsoft Sentinel | Connected to `LAW-SOC-LAB` | SIEM/SOC platform |
| Virtual Network | `VNET-SOC-LAB` | Private network boundary for the lab |
| Subnet | `SNET-SOC-LAB` | Initial workload subnet |
| Windows VM | `VM-SOC-WIN01` | Monitored Windows endpoint |
| Network Security Group | `NSG-SOC-WIN01` | Controls network access to the VM |
| Public IP | `VM-SOC-WIN01-ip` | Required for the initial RDP management path |
| Data Collection Rule | `DCR-SOC-WIN01-SecurityEvents` | Defines Windows Security event collection |

## 3. Resource Group

A dedicated resource group, `RG-SOC-LAB`, was created to keep the learning environment isolated from unrelated Azure resources.

This makes the lab easier to:

- Identify.
- Manage.
- Monitor.
- Clean up.
- Rebuild.

The project also uses consistent resource naming and tags.

## 4. Log Analytics Workspace

`LAW-SOC-LAB` was created as the central Log Analytics workspace.

Microsoft Sentinel was connected to this workspace.

The workspace is the central destination for the telemetry collected from the lab endpoints.

## 5. Microsoft Sentinel

Microsoft Sentinel was enabled for `LAW-SOC-LAB`.

The Sentinel workspace was also connected to the Microsoft Defender portal as part of the current Microsoft security operations experience.

At this stage, Sentinel is intentionally being used first as the SIEM/telemetry analysis layer. Detection rules, incidents, automation, and more advanced SOC capabilities will be introduced progressively.

## 6. Network Design

The initial virtual network was created specifically for the SOC lab.

```text
VNET-SOC-LAB
10.10.0.0/16
│
└── SNET-SOC-LAB
    10.10.1.0/24
    │
    └── VM-SOC-WIN01
```

The larger `/16` VNet address space leaves room for additional subnets as the lab expands.

Future phases may introduce separate subnets for attacker simulation, Linux systems, servers, or other security infrastructure.

## 7. Network Security Group

`NSG-SOC-WIN01` was created for the Windows endpoint.

Initially, Azure offered a default `Any → RDP` rule. That rule was removed.

The final custom management rule allows:

```text
Source:       Authorized public IP /32
Protocol:     TCP
Destination:  RDP / 3389
Action:       Allow
Priority:     100
```

This means the RDP management path is restricted to one public source address rather than being exposed to the entire Internet.

All other unsolicited inbound traffic remains denied by the NSG's default deny behavior.

### Security reasoning

RDP is required for the initial management workflow, but exposing TCP/3389 to all Internet sources would create unnecessary attack surface.

The lab therefore starts with:

```text
Internet
   │
   │ TCP/3389
   ▼
NSG-SOC-WIN01
   │
   ├── Authorized public IP → ALLOW
   │
   └── Other sources → DENY
   │
   ▼
VM-SOC-WIN01
```

## 8. Windows Endpoint

The first monitored endpoint is:

- Name: `VM-SOC-WIN01`
- OS: Windows 11 Pro, version 25H2
- Size: Standard `B2als_v2`
- Architecture: x64
- Security type: Trusted Launch
- Secure Boot: Enabled
- vTPM: Enabled
- OS disk: Standard SSD LRS
- Managed identity: Enabled
- Entra ID VM login: Disabled initially
- Backup: Disabled
- Site Recovery: Disabled
- Boot diagnostics: Enabled
- Auto-shutdown: Enabled

The VM was deliberately kept small and cost-conscious because it is a learning endpoint rather than a production workload.

The Azure portal showed an estimated compute price of `$0.0246/hour` at deployment time. This is not the complete lab cost because storage, public IP and other billable resources can have separate charges.

## 9. Cost Controls

The lab uses several cost-control measures:

- Small burstable VM size.
- Standard SSD rather than Premium SSD.
- Auto-shutdown.
- No Backup.
- No Site Recovery.
- No load balancer.
- No additional data disk.
- Selective telemetry collection rather than collecting every Windows event.
- Planned cleanup of unused resources.

Cost should be checked regularly in Azure Cost Management rather than assumed from the VM's compute price alone.

## 10. Initial Validation

After deployment, the following were validated:

- RDP access to the Windows VM.
- PowerShell functionality.
- Internet connectivity.
- Windows Event Viewer.
- Windows Security event generation.

A successful logon event (`4624`) was observed locally in the Windows Security log.

## 11. Result

The Azure foundation is ready for telemetry ingestion.

The next phase connected the Windows Security log to Log Analytics and Sentinel using Azure Monitor Agent and a Data Collection Rule.

## Screenshots

Suggested screenshots:

- `screenshots/azure/resource-group.png`
- `screenshots/azure/sentinel-workspace.png`
- `screenshots/azure/vm-review.png`
- `screenshots/azure/nsg-rdp-restricted.png`
- `screenshots/windows/security-event-viewer.png`

## Lessons Learned

- A resource group provides logical isolation for a lab.
- A VNet provides the network boundary, while the subnet provides the initial workload segment.
- An NSG is a network access control layer and should not be treated as an application security control.
- Management ports should not be unnecessarily exposed to all Internet sources.
- Cloud lab design must account for resource cost in addition to technical functionality.
