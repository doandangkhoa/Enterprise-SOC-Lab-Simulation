```xml
<group name="web,brute_force,">

  <rule id="100400" level="5">
    <if_sid>31101</if_sid>
    <options>no_log</options>
    <description>Log collector for 404 errors (Hidden)</description>
  </rule>

  <rule id="100401" level="12" frequency="20" timeframe="30">
    <if_matched_sid>100400</if_matched_sid>
    <same_source_ip />
    <description>SOC CRITICAL: Phat hien tan cong Brute Force vao Web Server!</>
    <mitre>
      <id>T1110</id>
    </mitre>
  </rule>

</group>

```