# Windows Security Telemetry Ingestion Flow

## End-to-end data flow

```mermaid
flowchart LR
    A["Windows activity<br/>Logon / Process / Account change"]
    B["Windows Security Log"]
    C["Azure Monitor Agent<br/>AMA"]
    D["Data Collection Rule<br/>DCR-SOC-WIN01-SecurityEvents"]
    E["Log Analytics Workspace<br/>LAW-SOC-LAB"]
    F["SecurityEvent"]
    G["Microsoft Sentinel"]
    H["KQL"]

    A --> B
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G
    G --> H
```

## Current configuration

| Component | Current value |
|---|---|
| Endpoint | `VM-SOC-WIN01` |
| Agent | Azure Monitor Agent |
| DCR | `DCR-SOC-WIN01-SecurityEvents` |
| Event set | Common |
| Workspace | `LAW-SOC-LAB` |
| Table | `SecurityEvent` |
| Retention at configuration | 30 days |
| SIEM | Microsoft Sentinel |

## Validation example

A controlled logon generated Event ID `4624`.

The same event was then found centrally with:

```kusto
SecurityEvent
| where EventID == 4624
| sort by TimeGenerated desc
| take 10
```

This confirms the telemetry path is operational.
