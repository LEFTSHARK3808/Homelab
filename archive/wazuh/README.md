# Wazuh SIEM (archived)

My first SIEM build, before the detection lab moved to Splunk. The VM is kept stopped as a reference; the host doesn't have the RAM to run both.

## What was built

| | |
|---|---|
| **Version** | Wazuh 4.7.5, all-in-one (indexer + manager + dashboard) |
| **OS** | Ubuntu 22.04 VM |
| **Agents** | Proxmox host, Docker LXC |
| **Active response** | SSH brute force (rule 5763) → `firewall-drop` for 600 s |

## Why I moved to Splunk

- Splunk is what most SOC job postings ask for, and SPL is a transferable skill.
- ESCU gives a free, ATT&CK-mapped baseline to measure my own detections against.
- 15 GB of host RAM can't hold two SIEMs.

## Planned but not built

- Valheim connection monitoring (custom decoders + correlation rules for connection floods and password brute force)
- Upgrade to 4.14 (one-way: the indexer's Lucene version can't be downgraded after 4.12)

## Example custom rules

```xml
<group name="atomic,linux,">
  <rule id="100300" level="12">
    <if_sid>80700</if_sid>
    <field name="audit.key">identity</field>
    <match>shadow</match>
    <description>Possible credential access: /etc/shadow touched (T1003.008)</description>
    <mitre><id>T1003.008</id></mitre>
  </rule>

  <rule id="100301" level="10">
    <if_sid>80700</if_sid>
    <field name="audit.key">persistence</field>
    <description>Cron persistence: scheduled task modified (T1053.003)</description>
    <mitre><id>T1053.003</id></mitre>
  </rule>
</group>
```
