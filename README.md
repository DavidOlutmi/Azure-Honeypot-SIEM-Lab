# Azure Honeypot & SIEM Lab

I deployed a deliberately exposed Azure VM to attract real-world attackers,
then used Microsoft Sentinel to detect, geolocate, and visualize where
the attacks were coming from — in real time on a live world map.

## Architecture
```text
Public Internet (Real Attackers)
           |
           | Live Attack Traffic
           v
+----------+----------+
|   Network Security  |
|   Group (NSG)       |
|   [Firewall OFF]    |
+----------+----------+
           |
           v
+----------+----------+
|   Azure VM          |
|   (Honeypot)        |
|   Windows Firewall  |
|   Disabled          |
+----------+----------+
           |
           | Logs forwarded via Data Collection Rule
           v
+----------+----------+
|   Log Analytics     |
|   Workspace (LAW)   |
|   (Log Conduit)     |
+----------+----------+
           |
           | Log data + Geo IP enrichment
           v
+----------+----------+
|   Microsoft         |
|   Sentinel (SIEM)   |
|                     |
|   - KQL Queries     |
|   - Geo IP Mapping  |
|   - Alert Triage    |
|   - Attack Map      |
+---------------------+
```


## Azure Resources Deployed

| Resource | Type |
|---|---|
| CORPORATE-NETWORK | Virtual Machine (Honeypot) |
| CORPORATE-NETWORK-ip | Public IP Address |
| CORPORATE-NETWORK-nsg | Network Security Group |
| LogAnalyticWorkspaceSocLab | Log Analytics Workspace |
| SecurityInsights | Microsoft Sentinel (SIEM) |
| VnetworkSocLab | Virtual Network |
| DataCollectionRule | Data Collection Rule |
| WindowsVmAttackMap | Azure Workbook (Attack Map) |

## How It Works

1. **Honeypot setup** — Created an Azure VM, disabled Windows Firewall to
   expose it fully to the internet, and confirmed visibility with a ping test
2. **Log pipeline** — Connected the VM to a Log Analytics Workspace, which
   acts as a conduit feeding raw log data into Microsoft Sentinel
3. **Geo IP enrichment** — Fed Sentinel a dataset of IP ranges mapped to
   countries, cities, and coordinates so it could cross-reference the attacker
   IPs and pinpoint their locations
4. **KQL detection** — Wrote KQL queries to filter for failed RDP login
   attempts (Event ID 4625) and surface attacker IP, timestamp, and
   geolocation data
5. **Attack map** — Built a Sentinel Workbook using an open-source JSON
   script to plot attacks on a real-time world map, with circle size
   representing the volume of attempts per city

## Attack Map Results

Attackers were detected from multiple countries within hours of the VM
going live:

| Location | Failed Attempts |
|---|---|
| Rome, Italy | 2,720 |
| Tilburg, Netherlands | 1,040 |
| Mandurah, Australia | 31 |
| Copenhagen, Denmark | 2 |

## Key Takeaways

Exposing a VM to the internet, even briefly, results in immediate,
automated brute force attempts from across the world. The majority of
traffic came from Europe within the first hours. The geo IP enrichment
pipeline in Sentinel made it possible to move from raw logs to a
visual, analyst-ready dashboard without any manual IP lookups.
