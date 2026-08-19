# SOC Lab Architecture

## Architecture Evolution

The SOC Cloud Security Lab is intentionally documented as an evolving architecture rather than only as a final-state diagram.

| Version | Milestone | Focus | Status |
|---|---|---|---|
| V1 | Azure SOC Foundation | Azure network, Windows endpoint and initial telemetry path | Historical |
| V2 | Detection Engineering | Windows telemetry, KQL detection, alert and incident workflow | Historical |
| V3 | SentinelOne EDR Integration | Multi-source telemetry with SentinelOne EDR + Microsoft Sentinel | Current |

### Version history

- [Architecture V1 — Azure SOC Foundation](architecture-v1-foundation.md)
- [Architecture V2 — Detection Engineering](architecture-v2-detection.md)
- [Architecture V3 — Multi-Source SOC](architecture-v3-multi-source.md)

The historical versions are intentionally retained to show how the lab evolved.

---

## Current Architecture — V3

```mermaid
flowchart TB
    Analyst["Analyst / Administrator"]

    subgraph AZ["Azure - RG-SOC-LAB"]
        subgraph NET["VNET-SOC-LAB - 10.10.0.0/16"]
            SUB["SNET-SOC-LAB - 10.10.1.0/24"]
            VM["VM-SOC-WIN01<br/>Windows 11 Pro"]
            NSG["NSG-SOC-WIN01<br/>RDP restricted to authorized public IP"]
            PIP["VM-SOC-WIN01-ip"]

            SUB --> VM
            PIP --> NSG
            NSG --> VM
        end

        AMA["Azure Monitor Agent"]
        DCR["DCR-SOC-WIN01-SecurityEvents"]
        LAW["LAW-SOC-LAB"]
        SE["SecurityEvent"]
        SENT["Microsoft Sentinel"]
        RULE["SOC-LAB - Multiple Failed Logons Across Accounts"]
        INC["Alert / Incident<br/>Investigation & Closure"]
    end

    subgraph S1["SentinelOne"]
        AGENT["SentinelOne Agent<br/>VM-SOC-WIN01"]
        CONSOLE["SentinelOne Management Console<br/>SOC-Lab-Ikrama"]
        API["SentinelOne API"]
        CONN["SentinelOne V2<br/>Codeless Connector"]
        ACT["SentinelOneActivities_CL"]
    end

    Analyst -->|"Restricted RDP"| PIP

    VM --> AMA --> DCR --> LAW --> SE --> SENT
    SENT --> RULE --> INC

    VM --> AGENT
    AGENT --> CONSOLE
    CONSOLE --> API
    API --> CONN --> ACT --> SENT
```

## Supporting Flows

- [Windows Security Telemetry Ingestion](telemetry-ingestion.md)
- [SentinelOne EDR Integration](sentinelone-integration.md)
- [Detection & Incident Response](detection-and-response.md)
- [Network & Access Flow](network-security.md)
