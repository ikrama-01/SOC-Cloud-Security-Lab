# 04 - SentinelOne EDR Integration

## 1. Objective

This phase adds an endpoint detection and response (EDR) telemetry source to the SOC lab.

The goal is to place SentinelOne alongside the existing Windows Security telemetry and ingest SentinelOne activity into Microsoft Sentinel.

The integration establishes the foundation for future multi-source investigations and correlation.

## 2. Integration Architecture

The Windows endpoint now produces two primary telemetry streams:

```text
                    VM-SOC-WIN01
                    Windows 11 Pro
                         │
             ┌───────────┴───────────┐
             │                       │
             ▼                       ▼
     Windows Security Log      SentinelOne Agent
             │                       │
             ▼                       ▼
            AMA                SentinelOne Console
             │                       │
             ▼                       ▼
            DCR                 SentinelOne API
             │                       │
             ▼                       ▼
      LAW-SOC-LAB            SentinelOne V2 Connector
             │                       │
             ▼                       ▼
       SecurityEvent          SentinelOneActivities_CL
             │                       │
             └───────────┬───────────┘
                         ▼
                 Microsoft Sentinel
```

See the dedicated [SentinelOne integration diagram](diagrams/sentinelone-integration.md).

## 3. SentinelOne Lab Group

A dedicated SentinelOne group was created for the lab:

```text
SOC-Lab-Ikrama
```

The group was intentionally named generically enough to support additional endpoint operating systems in future phases.

The group is currently used only for SOC lab endpoints.

A dedicated group provides a clean boundary for:

- Lab endpoint assignment.
- Lab-specific policy testing.
- Future Windows and Linux endpoint additions.
- EDR configuration changes without affecting unrelated endpoints.

## 4. Endpoint Deployment

The SentinelOne agent was installed on:

```text
VM-SOC-WIN01
```

The endpoint successfully appeared in the SentinelOne management console.

Validated endpoint state included:

```text
Health status:        Healthy
Console connectivity: Online
Network status:       Connected
Agent version:        26.1.2.177
OS:                   Windows 11 Pro (64 bit)
```

The endpoint was then moved into:

```text
SOC-Lab-Ikrama
```

The endpoint was initially assigned the available default policy for the lab group.

No additional aggressive policy tuning was performed during this phase.

## 5. Initial EDR Console Review

The endpoint details were reviewed to understand the information exposed by the EDR console, including:

- Endpoint identity.
- Operating system.
- Agent version.
- Health state.
- Console connectivity.
- Network state.
- Last logged-in user.
- IP information.
- CPU and memory information.
- Scan status.
- Endpoint group association.

No threat or incident was expected at this stage because the endpoint had just been onboarded and no malicious activity had been generated.

The purpose of this step was to establish endpoint visibility before introducing controlled detections.

## 6. Microsoft Sentinel Connector

The SentinelOne solution was installed through the Microsoft Sentinel Content Hub.

The connector used for this lab is:

**SentinelOne V2 (via Codeless Connector Framework)**

The connector is designed to ingest SentinelOne data through the SentinelOne REST API and Unified Alert Management GraphQL API.

The configured connection uses:

- SentinelOne Management URL.
- SentinelOne API token/service-user authentication.

The connector exposes multiple SentinelOne data types, including activity, agent, group and threat information, as well as SentinelOne alerts.

## 7. Connector Validation

After configuring the SentinelOne API connection, the connector page showed the configured SentinelOne Management URL and data types.

The following data types were visible in the connector configuration:

- Activities
- Agents (new)
- Agents (updated)
- Alerts (V2)
- Groups
- Threats (new)

The connector was then validated from Microsoft Sentinel Logs.

## 8. SentinelOne Activity Table

SentinelOne activity telemetry was observed in:

```text
SentinelOneActivities_CL
```

A basic table-discovery query was used:

```kusto
SentinelOneActivities_CL
| getschema
```

The schema output showed **26 columns**.

The visible schema fields included:

```text
TimeGenerated
AgentUpdatedVersion
UserId
ThreatId
PrimaryDescription
SecondaryDescription
Id
```

Additional fields are available in the table and can be explored as the integration is used for more scenarios.

## 9. Activity Ingestion Validation

The following query was used to validate recent SentinelOne activity:

```kusto
SentinelOneActivities_CL
| where TimeGenerated > ago(1h)
| order by TimeGenerated desc
```

The query returned three recent activity records during validation.

The observed records included activity associated with the SentinelOne management user and endpoint/user activity.

This confirms that:

```text
SentinelOne
     ↓
SentinelOne API
     ↓
SentinelOne V2 Codeless Connector
     ↓
Microsoft Sentinel
     ↓
SentinelOneActivities_CL
```

is operational.

## 10. Current Integration State

| Component | Status |
|---|---|
| SentinelOne agent on `VM-SOC-WIN01` | Healthy |
| SentinelOne console connectivity | Online |
| SentinelOne lab group | `SOC-Lab-Ikrama` |
| SentinelOne policy | Default lab group policy |
| Microsoft Sentinel solution | Installed |
| Connector | SentinelOne V2 via Codeless Connector Framework |
| API connection | Configured |
| Activity ingestion | Validated |
| Table | `SentinelOneActivities_CL` |
| S1-specific analytics rules | Not yet configured |
| S1 threat simulation | Not yet performed |
| Cross-source correlation | Not yet configured |
| Sentinel automation | Not yet configured |

## 11. Why This Integration Matters

The lab previously had a single primary telemetry source:

```text
Windows Security Events
```

It now has an endpoint-security telemetry source as well:

```text
Windows Security Events + SentinelOne EDR
```

This is an important SOC architecture step because a real investigation often requires multiple sources.

For example, a future scenario could correlate:

```text
Windows authentication event
        +
SentinelOne process activity
        +
SentinelOne threat information
        +
Network/source information
        ↓
Higher-confidence investigation
```

## 12. What Was Not Done Yet

The following were intentionally left for later phases:

- SentinelOne-specific analytics rules.
- SentinelOne threat generation.
- Malware/ransomware simulation.
- Process-based hunting using S1 telemetry.
- Cross-source correlation between `SecurityEvent` and SentinelOne tables.
- Automated response/playbooks.
- Advanced S1 policy tuning.

This keeps the lab progression structured instead of introducing multiple new concepts at the same time.

## Screenshots

### SentinelOne endpoint

![SentinelOne endpoint](screenshots/sentinelone/s1-console-endpoint.png)

### SentinelOne schema

![SentinelOne schema](screenshots/sentinelone/s1-schema.png)

### SentinelOne connector configuration

![SentinelOne connector](screenshots/sentinelone/s1-sentinel-integration.png)

### SentinelOne activity ingestion

![SentinelOne activity ingestion](screenshots/sentinelone/s1-sentinel-integration1.png)

## Lessons Learned

- EDR provides a complementary telemetry source to operating-system security logs.
- A healthy endpoint does not automatically mean that the SIEM is receiving EDR telemetry; ingestion must be validated separately.
- The SentinelOne V2 Codeless Connector provides a direct integration path into Microsoft Sentinel.
- Schema inspection with `getschema` is useful before writing detections against a new data source.
- Multi-source telemetry creates the foundation for stronger investigation and correlation.
