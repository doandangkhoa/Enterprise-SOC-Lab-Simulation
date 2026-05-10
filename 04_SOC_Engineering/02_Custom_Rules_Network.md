# TỔNG THỂ KIẾN TRÚC DETECTION

```txt
Internet Attacker
      ↓
[MẢNG 1] Network Edge (pfSense Firewall)
      ↓
[MẢNG 2] Network Payload (Suricata IDS)
      ↓
[MẢNG 3] Endpoint (Sysmon / Authentication / Windows)
      ↓
Wazuh Correlation Engine
      ↓
SOC Dashboard / Alert / Incident Response
```

**Kiến trúc triển khai:**

Để tối ưu hóa hiệu suất Engine phân tích và dễ dàng bảo trì theo chuẩn Enterprise, toàn bộ các luật Network dưới đây thay vì viết chung vào `local_rules.xml` đã được thiết kế tách biệt thành các file cấu hình độc lập trên thư mục `/var/ossec/etc/rules/` của máy chủ Wazuh.

# MẢNG 1 — NETWORK EDGE RULES (pfSense Firewall)

**Mục tiêu:** Phát hiện giai đoạn đầu của tấn công (Recon + Initial probing):

- Port scanning
    
- Thăm dò dịch vụ
    
- Thử truy cập cổng quản trị
    
- SSH brute force từ Internet
    

**File cấu hình:** `100-pfsense-rules.xml`

```xml
<group name="local,pfsense,firewall,recon">

  <!-- 🔵 100100 — Port Scan Detection -->
  <rule id="100100" level="10" frequency="10" timeframe="60">
    <if_matched_sid>87701</if_matched_sid>
    <same_source_ip />
    <description>Firewall: Mass Port Scanning detected from $(srcip)</description>
    <group>recon,scan,network_discovery</group>
    <mitre>
      <id>T1046</id>
    </mitre>
  </rule>

  <!-- 🔵 100101 — Admin Port Access Attempt -->
  <rule id="100101" level="8">
    <if_sid>87701</if_sid>
    <dstport>22,3389,5985,5986</dstport>
    <description>Firewall: Admin port access attempt $(dstport) from $(srcip)</description>
    <group>recon,targeted_attack,initial_access_attempt</group>
    <mitre>
      <id>T1046</id>
      <id>T1133</id>
    </mitre>
  </rule>

  <!-- 🔵 100102 — SSH Brute Force (Edge) -->
  <rule id="100102" level="12" frequency="10" timeframe="60">
    <if_matched_sid>87701</if_matched_sid>
    <dstport>22</dstport>
    <same_source_ip />
    <description>Firewall: Possible SSH brute force from $(srcip)</description>
    <group>brute_force,credential_access,ssh</group>
    <mitre>
      <id>T1110</id>
    </mitre>
  </rule>

</group>
```

# MẢNG 2 — SURICATA IDS (NETWORK PAYLOAD)

**Mục tiêu:** Phát hiện hành vi tấn công thực sự lẩn khuất trong payload của traffic mạng:

- Exploit web application (SQLi, XSS)
    
- Malware communication (C2 Server)
    
- Data exfiltration (Đánh cắp dữ liệu)
    
- DNS tunneling
    

**File cấu hình:** `101-suricata-rules.xml`

```XML
<group name="local,suricata,ids,attack">

  <!-- 🟡 100201 — Critical Exploit Detection -->
  <rule id="100201" level="12">
    <if_sid>86601</if_sid>
    <field name="alert.severity" type="pcre2">^1$|^2$</field>
    <description>Suricata CRITICAL: $(alert.signature) from $(src_ip) to $(dest_ip)</description>
    <group>exploit,malware</group>
    <mitre>
      <id>T1190</id>
    </mitre>
  </rule>

  <!-- 🟡 100202 — Internal C2 Communication -->
  <rule id="100202" level="14">
    <if_sid>100201</if_sid> 
    <field name="src_ip" type="pcre2">^(10\.|172\.(1[6-9]|2[0-9]|3[0-1])\.|192\.168\.)</field>
    <field name="alert.category">A Network Trojan was detected</field>
    <description>IDS: Internal host $(src_ip) communicating with C2 server $(dest_ip)</description>
    <group>c2,backdoor,compromise</group>
    <mitre>
      <id>T1071</id>
      <id>T1105</id>
    </mitre>
  </rule>

  <!-- 🟡 100203 — SQL Injection Detection -->
  <rule id="100203" level="12">
    <if_sid>86601</if_sid>
    <field name="alert.signature" type="pcre2">(?i)(sql injection|union select|or 1=1|information_schema)</field>
    <description>IDS: Possible SQL Injection attempt from $(src_ip)</description>
    <group>web_attack,exploit</group>
    <mitre>
      <id>T1190</id>
    </mitre>
  </rule>

  <!-- 🟡 100204 — DNS Tunneling / Exfiltration -->
  <rule id="100204" level="14">
    <if_sid>86601</if_sid>
    <field name="alert.signature" type="pcre2">(?i)(dns tunnel|large dns query|exfiltration|base64 dns)</field>
    <description>IDS: Possible DNS tunneling detected from $(src_ip)</description>
    <group>exfiltration,c2,dns_abuse</group>
    <mitre>
      <id>T1048</id>
    </mitre>
  </rule>

</group>
```