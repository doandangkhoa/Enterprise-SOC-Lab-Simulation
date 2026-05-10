### MẢNG CORRELATION (ATTACK CHAIN)

### 🔴 100400 — Scan → Brute Force → Access Chain

```
<rule id="100400" level="15" frequency="1" timeframe="300">

  <if_group>brute_force</if_matched_group>
  <if_matched_group>recon</if_matched_group>
  
  <same_source_ip />

  <description>
    Attack chain detected: Recon → Credential attack
  </description>

  <group>attack_chain,correlation</group>

  <mitre>
    <id>T1046</id>
    <id>T1110</id>
  </mitre>

</rule>
```