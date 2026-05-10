# MẢNG 3 — ENDPOINT (SYSMON / WINDOWS / AUTH)

### Mục tiêu

Phát hiện **khi attacker đã vượt qua firewall + IDS**:

- Execution
- Credential dumping
- Persistence
- Lateral movement

**File cấu hình:** `102-endpoint-rules.xml`

```xml
<group name="local,windows,sysmon,endpoint_threats">

  <!-- 🟢 100301 — Web Server Shell Spawn -->
  <rule id="100301" level="12">
    <if_group>sysmon_event1</if_group>
    <field name="win.eventdata.parentImage" type="pcre2">(?i)(w3wp\.exe|httpd\.exe|nginx\.exe|php-cgi\.exe|tomcat\.exe)$</field>
    <field name="win.eventdata.image" type="pcre2">(?i)(cmd\.exe|powershell(\.exe)?|pwsh\.exe|rundll32\.exe|mshta\.exe|cscript\.exe|wscript\.exe)$</field>
    <description>Endpoint CRITICAL: Web Server spawned shell (Possible Webshell Activity)</description>
    <group>webshell,rce,execution</group>
    <mitre>
      <id>T1505.003</id>
      <id>T1059</id>
    </mitre>
  </rule>

  <!-- 🟢 100302 — Credential Dumping Tools -->
  <rule id="100302" level="10">
    <if_group>sysmon_event1</if_group>
    <field name="win.eventdata.originalFileName" type="pcre2">(?i)(mimikatz|psexec|procdump|vssadmin|lsass)</field>
    <description>Endpoint WARNING: Credential dumping tool executed [$(win.eventdata.originalFileName)]</description>
    <group>credential_access,lateral_movement</group>
    <mitre>
      <id>T1003</id>
    </mitre>
  </rule>

  <!-- 🟢 100303 — Suspicious PowerShell -->
  <rule id="100303" level="12">
    <if_group>sysmon_event1</if_group>
    <field name="win.eventdata.image" type="pcre2">(?i)powershell(\.exe)?|pwsh\.exe</field>
    <field name="win.eventdata.commandLine" type="pcre2">(?i)(-enc|-encodedcommand|invoke-webrequest|downloadstring|iwr)</field>
    <description>Endpoint CRITICAL: Suspicious PowerShell execution detected</description>
    <group>execution,script,malware</group>
    <mitre>
      <id>T1059.001</id>
      <id>T1105</id>
    </mitre>
  </rule>

  <!-- 🟢 100304 — Scheduled Task Creation -->
  <rule id="100304" level="10">
    <if_group>sysmon_event1</if_group>
    <field name="win.eventdata.image" type="pcre2">(?i)schtasks\.exe</field>
    <field name="win.eventdata.commandLine" type="pcre2">(?i)/create|/change</field>
    <description>Endpoint WARNING: Suspicious Scheduled task creation/modification detected</description>
    <group>persistence,execution</group>
    <mitre>
      <id>T1053</id>
    </mitre>
  </rule>

  <!-- 🟢 100305 — LSASS Access -->
  <rule id="100305" level="14">
    <if_sid>61612</if_sid>
    <match>lsass.exe</match>
    <description>Endpoint CRITICAL: LSASS memory access detected (Possible credential theft)</description>
    <group>credential_access,privilege_escalation</group>
  </rule>

  <!-- 🟢 100306 — Event Logs Cleared -->
  <rule id="100306" level="12">
    <if_sid>63103, 63101, 63104</if_sid>
    <field name="win.system.eventID">^1102$|^104$</field>
    <description>Endpoint CRITICAL: Windows Event Logs have been cleared (Defense Evasion)</description>
    <group>defense_evasion,windows</group>
    <mitre>
      <id>T1070.001</id>
    </mitre>
  </rule>

  <!-- 🟢 100307 — Remote Desktop Logon -->
  <rule id="100307" level="10">
    <if_sid>60103</if_sid>
    <field name="win.eventdata.logonType">^10$</field>
    <description>Endpoint WARNING: Lateral Movement - Successful Remote Desktop (RDP) logon detected</description>
    <group>lateral_movement,authentication,windows</group>
    <mitre>
      <id>T1021.001</id>
    </mitre>
  </rule>

   <rule id="100400" level="5">
    <if_group>sysmon_event1</if_group>
    <field name="win.eventdata.image" type="pcre2">(?i)(\\\\Public\\\\|\\\\Downloads\\\\|\\\\Temp\\\\)</field>
    <description>Endpoint WARNING: Process spawned in suspicious directory [$(win.eventdata.image)]>
    <group>vt_scan_target</group>
  </rule>

  <rule id="100401" level="0">
    <if_sid>100400</if_sid> <field name="win.eventdata.image" type="pcre2">(?i)(chrome_updater\.exe|zalo_setup\.exe|msteams\.exe)</field>
    <description>SOC Tuning: Whitelisted safe processes in Temp/Downloads</description>
  </rule>
</group>
```

1.  Ấn `Ctrl+O`, `Enter` để lưu file.
    
2.  Ấn `Ctrl+X` để thoát.
    
3.  Chạy lệnh cấp lại quyền cho file (để chắc chắn Wazuh có quyền đọc file mới này):
    
    ```
    chown wazuh:wazuh 102-endpoint-rules.xml
    ```
    
4.  Kiểm tra xem file có lỗi cú pháp không:
    
    ```
    /var/ossec/bin/wazuh-control checkconf
    ```
    
5.  Nếu không có báo lỗi `ERROR`, tiến hành khởi động lại Manager:
    
    ```
    systemctl restart wazuh-manager
    ```