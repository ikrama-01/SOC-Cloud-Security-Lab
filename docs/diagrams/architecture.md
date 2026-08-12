# Architecture Diagram

## Day 1 - Azure SOC Foundation

```mermaid
flowchart TB
    Analyst["Analyst / Administrator"]

    subgraph RG["Resource Group: RG-SOC-LAB"]
        subgraph NET["VNET-SOC-LAB - 10.10.0.0/16"]
            SUB["SNET-SOC-LAB - 10.10.1.0/24"]
            VM["VM-SOC-WIN01<br/>Windows 11 Pro"]
            NSG["NSG-SOC-WIN01<br/>RDP restricted to authorized public IP"]
            PIP["VM-SOC-WIN01-ip"]
            SUB --> VM
            NSG --> VM
            PIP --> NSG
        end

        LAW["LAW-SOC-LAB<br/>Log Analytics"]
        SENT["Microsoft Sentinel"]
        DCR["DCR-SOC-WIN01-SecurityEvents"]
        AMA["Azure Monitor Agent"]
    end

    Analyst -->|"RDP / TCP 3389"| PIP
    VM --> AMA
    AMA --> DCR
    DCR --> LAW
    LAW --> SENT
```

## Architectural intent

The lab is intentionally separated into:

1. **Network layer** - VNet, subnet, NSG and public management path.
2. **Endpoint layer** - Windows endpoint producing security telemetry.
3. **Collection layer** - AMA and DCR.
4. **Data layer** - Log Analytics and `SecurityEvent`.
5. **SOC layer** - Microsoft Sentinel.

This separation makes it easier to understand where a failure occurs.

For example:

- No Windows event → endpoint problem.
- Event exists locally but not centrally → collection/agent/DCR problem.
- Event reaches Log Analytics but query fails → KQL/table/query problem.
- Event is visible but no alert → detection engineering problem.
