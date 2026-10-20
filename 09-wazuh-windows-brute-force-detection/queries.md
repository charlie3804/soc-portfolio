# Searches and commands — Project 09

## Wazuh dashboard (Threat Hunting → Events, DQL)

| Purpose | Query |
|---|---|
| Medium and higher alerts (initial triage) | `rule.level >= 7` |
| All failed logons from the Windows endpoint | `agent.name:WIN-CHARLIE and data.win.system.eventID:4625` |
| Brute-force correlation alert | `rule.id:60204` |
| Every attempt against the guessed account | `data.win.eventdata.targetUserName:hacker_test` |
| Network logons only | `data.win.system.eventID:4625 and data.win.eventdata.logonType:3` |
| Unknown-user failures (account guessing) | `data.win.eventdata.subStatus:0xc0000064` |
| Did the attacker get in? (successful logons) | `agent.name:WIN-CHARLIE and data.win.system.eventID:4624` |

Key fields: `rule.id`, `rule.level`, `rule.mitre.id`, `rule.frequency`, `previous_output`, `data.win.system.eventID`, `data.win.eventdata.targetUserName`, `data.win.eventdata.logonType`, `data.win.eventdata.subStatus`, `data.win.eventdata.ipAddress`, `data.win.eventdata.logonProcessName`.

## Wazuh server (Ubuntu / WSL2)

```bash
# Prerequisite: systemd must be PID 1
ps -p 1 -o comm=

# All-in-one install
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a

# Health check of the three components
systemctl is-active wazuh-indexer wazuh-manager wazuh-dashboard

# Test the admin credentials directly against the indexer
curl -k -u admin https://localhost:9200

# Reset the admin password
sudo bash /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh -u admin -p '<new-password>'
```

## Windows endpoint (PowerShell)

```powershell
# Install the agent (elevated PowerShell)
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.9-1.msi -OutFile $env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='127.0.0.1' WAZUH_AGENT_NAME='WIN-CHARLIE'
NET START Wazuh

# Verify
Get-Service WazuhSvc
Test-NetConnection 127.0.0.1 -Port 1515
wsl -l -v

# Single failed logon (interactive, Logon Type 2)
runas /user:$env:USERNAME cmd

# Simulated brute force over the network (Logon Type 3), non-existent account
1..10 | ForEach-Object { net use \\127.0.0.1\IPC$ /user:hacker_test Clave$_ 2>$null }
```
