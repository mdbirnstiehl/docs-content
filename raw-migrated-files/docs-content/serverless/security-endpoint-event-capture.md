# Event capture and {{elastic-defend}} [security-endpoint-event-capture]

{{elastic-defend}} collects select data on system activity in order to detect and prevent as many threats as possible, while balancing storage and performance overhead. To that end, {{elastic-defend}} isn’t designed to capture all system events. Some event data that {{elastic-defend}} generates gets aggregated, truncated, or deduplicated as needed to optimize threat detection and prevention.

You can supplement {{elastic-defend}}'s protection capabilities with [Elastic integrations](https://docs.elastic.co/en/integrations) and tools that provide more visibility and historical data. Consult the following sections to expand data collection for specific types of system events.


## Network port creation and deletion [security-endpoint-event-capture-network-port-creation-and-deletion] 

{{elastic-defend}} tracks TCP connections. If a port is created but no traffic flows, no events are generated.

For complete capture of network port creation and deletion, consider capturing Windows event ID 5158 using the [Custom Windows Event Logs](https://docs.elastic.co/en/integrations/winlog) integration.


## Network in/out connections [security-endpoint-event-capture-network-inout-connections] 

{{elastic-defend}} tracks TCP connections, which don’t include network in/out connections.

For complete network capture, consider deploying {{packetbeat}} using the [Network Packet Capture](https://docs.elastic.co/en/integrations/network_traffic) integration.


## User behavior [security-endpoint-event-capture-user-behavior] 

{{elastic-defend}} only captures user security events required by its behavioral protection. This doesn’t include every user event such as logins and logouts, or every time a user account is created, deleted, or modified.

For complete capture of all or specific Windows security events, consider the [Custom Windows Event Logs](https://docs.elastic.co/en/integrations/winlog) integration.


## System service registration, deletion, and modification [security-endpoint-event-capture-system-service-registration-deletion-and-modification] 

{{elastic-defend}} only captures system service security events required by its behavioral protection engine. Service creation and modification can also be detected in registry activity, for which {{elastic-defend}} has internal rules such as [Registry or File Modification from Suspicious Memory](https://github.com/elastic/protections-artifacts/blob/6d54ae289b290b1d42a7717569483f6ce907200a/behavior/rules/persistence_registry_or_file_modification_from_suspicious_memory.toml).

For complete capture of all or specific Windows security events, consider the [Custom Windows Event Logs](https://docs.elastic.co/en/integrations/winlog) integration. In particular, capture events such as [Windows event ID 4697](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4697).


## Kernel driver registration, deletion, and queries [security-endpoint-event-capture-kernel-driver-registration-deletion-and-queries] 

{{elastic-defend}} scans every driver as it is loaded, but it doesn’t generate an event each time.

Drivers are registered in the system as system services. You can capture this with Windows event ID 4697 using the [Custom Windows Event Logs](https://docs.elastic.co/en/integrations/winlog) integration.

Also consider capturing Windows event ID 6 using {{winlogbeat}}'s [Sysmon module](https://www.elastic.co/guide/en/beats/winlogbeat/current/winlogbeat-module-sysmon.html).


## System configuration file creation, modification, and deletion [security-endpoint-event-capture-system-configuration-file-creation-modification-and-deletion] 

{{elastic-defend}} tracks creation, modification, and deletion of all files on the system. However, as mentioned above, the data might be aggregated, truncated, or deduplicated to provide only what’s required for threat detection and prevention.

