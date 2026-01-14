#  Stepped-On Silicon
## Dirty Consumer Devices Pass the Purity Test
### Exposé analysis of documented findings found post [Recovery Mode](https://support.apple.com/en-us/118106) (DFU Reset)
---

## Executive Summary


Consumer iPhones purchased new & used from different authorized U.S. retailers and factory reset at Apple Stores, emerged with persistent surveillance infrastructure fully intact. The findings demonstrate that standard factory reset procedures fail to achieve advertised data erasure, while monitoring capabilities remain active and completely invisible to both users and security validation tools.

**Independent verification:** All findings backed by publicly available sysdiagnose captures with SHA256 verification hashes provided in per-device breakdowns (`/devices/`).
```
┌─────────────────────────────────────────────────────────────┐
│  ✓ Pre-configured monitoring infrastructure on clean devices │
│  ✓ Historical data surviving factory reset (timestamped)     │
│  ✓ Cross-carrier coordination (Taiwan Mobile + T-Mobile USA) │
│  ✓ Automated enrollment without user knowledge               │
│  ✓ All traffic routed through hidden private infrastructure  │
│  ✓ Completely invisible to users and security tools          │
└─────────────────────────────────────────────────────────────┘
```

**The Problem:** A device can be forensically "dirty" while appearing completely "clean" - passing all standard security validation procedures. Maintaining persistent monitoring and enrollment capabilities.



---

## Quick Facts

| Question | Answer |
|----------|--------|
| **Devices Tested** | iPhone 12, 14 Pro Max, 15 Pro Max(preliminary)|
| **Device type** | Consumer (no enterprise management) |
| **When captured** | Immediately after DFU restore |
| **Security scan result** | ✅ Clean (falsely) |
| **Actual state** | 🔴 Monitoring active |

---

## 📊 What was Found

###  Finding #1: Cross-Carrier Infrastructure Coordination

**Two continents. Different carriers. Same backend.**

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                   │
│  🇹🇼 Taiwan Mobile          🇺🇸 T-Mobile USA                      │
│  iPhone 14 Pro Max         iPhone 12                             │
│         ↓                        ↓                                │
│   172.31.34.114            172.31.35.241                         │
│         ↓                        ↓                                │
│         └────────────┬───────────┘                                │
│                      ↓                                            │
│              172.31.32.1 (VPC Gateway)                           │
│                      ↓                                            │
│              AWS Private Subnet                                  │
│            172.31.0.0/16 (RFC 1918)                             │
│                                                                   │
│  Same Certificate Authority: FDRDC-UCRT-SUBCA                   │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

**Question:** Why do carriers on different continents route to identical private infrastructure?

---

###  Finding #2: Pre-Installed Monitoring Infrastructure

**Found on ALL consumer devices immediately post-factory reset:**

<table>
<tr><td width="50%">

**Verified Devices:**
```
✓ iPhone 12 (A14)
✓ iPhone 14 Pro Max (A16)
✓ iPhone 15 Pro Max (A17 Pro)
```
**No enterprise management**  
**All post-DFU**  
**All had monitoring configs**

</td><td width="50%">

**Monitoring Capabilities:**
```
👁️  Screen (VoiceOverTouch)
🎤 Audio (SoundDetection)  
📝 Text (SpeakSelection)
⚙️  Processes (RunningBoard)
🌐 Network (SYSTEM_PROXY)
```
**Active before user setup**

</td></tr>
</table>

#### 📂 Configuration Files Found:

| File | Size | Function | Capability |
|------|------|----------|------------|
| `VoiceOverTouch.plist` | 45KB | Screen reader | Monitor all on-screen content, UI, text input |
| `SoundDetection.plist` | 236B | Audio analysis | Detect environmental sounds |
| `HearingAids.plist` | 80B | Audio routing | Route/monitor audio streams |
| `SpeakSelection.plist` | 430B | Text tracking | Track selection, reading patterns |
| `RunningBoard_state.log` | 442KB | Process control | Control apps, resources, background execution |

**Critical:** Present whether or not users ever enabled accessibility features.

**🔗 Combined with network routing →** Infrastructure can monitor:
-  What user **sees** (screen)
-  What user **hears** (audio)  
-  What user **types** (text)
-  What **runs** (processes)
-  Where it **goes** (network)

---

### Finding #3: Mandatory Traffic Interception

**All network traffic forced through hidden infrastructure:**

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│   📱 User's iPhone                                               │
│   "Normal internet browsing"                                    │
│            ↓                                                     │
│   ╔═══════════════════════════════════════════════╗            │
│   ║ SYSTEM_PROXY (system-wide routing)            ║            │
│   ║ NEVPN (VPN tunnel establishment)              ║            │
│   ║ ne_dns_proxy_state_watch (DNS interception)   ║            │
│   ╚═══════════════════════════════════════════════╝            │
│            ↓                                                     │
│   🔒 Encrypted VPN Tunnel                                       │
│            ↓                                                     │
│   ☁️  172.31.x.x (Private AWS Infrastructure)                   │
│   • Not publicly routable                                       │
│   • Requires pre-configured proxy                               │
│   • TLS decryption capable                                      │
│            ↓                                                     │
│   ❓ Monitoring/Analysis Backend                                │
│   • Location: Unknown                                           │
│   • Operator: Unknown                                           │
│                                                                  │
│   👤 User sees: Normal internet ✅                              │
│   🔴 Reality: Complete interception                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

###  Finding #4: Invisible Automated Enrollment

**Timeline of automatic enrollment (no user interaction):**

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                  │
│  T-00:00  🔧 User initiates DFU restore                         │
│           └─ Device enters recovery mode                        │
│                                                                  │
│  T+00:15  📜 Certificate generation DURING DFU                  │
│           └─ Before OS boots                                    │
│           └─ iPhone 12: 18:52:14                               │
│                                                                  │
│  T+01:15  ✅ Device activated                                   │
│           └─ iPhone 12: 75 seconds post-DFU                    │
│           └─ All device IDs transmitted                         │
│                                                                  │
│  T+01:15  🔐 Permissions auto-granted                           │
│           └─ iPhone 14: Health/Photos at 14:47:24              │
│           └─ During DFU process                                 │
│                                                                  │
│  T+01:15  🌐 Network routing configured                         │
│           └─ VPN to 172.31.x.x established                     │
│           └─ DNS proxy activated                                │
│                                                                  │
│   User sees: Normal setup process                            │
│   Reality: Complete enrollment active                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Enrollment methods identified:**
-  Proximity beacons (BuddyDaemon)
-  Carrier integration (IMS/SIP)
-  Certificate-based (auto-issued)

---

###  Finding #5: Forensically Undetectable

**All indicators pass validation:**

<table>
<tr>
<th>Component</th>
<th>What Security Tools See</th>
<th>Actual State</th>
</tr>
<tr>
<td>🔄 Processes</td>
<td>✅ Legitimate Apple daemons<br/><code>mobileactivationd</code><br/><code>betaenrollmentd</code></td>
<td>🔴 Enrollment active</td>
</tr>
<tr>
<td>🔐 Certificates</td>
<td>✅ Valid Apple CA chain<br/>Properly signed</td>
<td>🔴 Auto-installed during DFU</td>
</tr>
<tr>
<td>🌐 Network</td>
<td>✅ Standard VPN/cellular<br/>Normal protocols</td>
<td>🔴 Traffic routed to 172.31.x.x</td>
</tr>
<tr>
<td>🛡️ Security Scan</td>
<td>✅ No malware detected<br/>Clean device</td>
<td>🔴 Monitoring infrastructure active</td>
</tr>
</table>


---

###  Finding #6: Survives Factory Reset

**DFU restore does NOT erase:**

```
┌──────────────────────────────────────────────────────────────┐
│                                                               │
│   TIMESTAMP EVIDENCE                                        │
│                                                               │
│  iPhone 12 Log: "Sun Dec 14 13:59:54 2025"                  │
│  Capture Date:   January 13, 2026                            │
│  Age:            30 days old ⚠️                               │
│                                                               │
│  iPhone 12 Log: "Mon Jan 5 13:58:41 2026"                   │
│  Capture Date:   January 13, 2026                            │
│  Age:            8 days old ⚠️                                │
│                                                               │
│    How do pre-reset logs exist post-reset?                  │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

**What survives:**
-  System logs (timestamped 8-30 days pre-DFU)
-  Monitoring configurations (accessibility plists)
-  Process control state (RunningBoard)
-  Device identifiers (IMEI, Serial, UDID)
-  Encryption keys (SEP bypass prevents erasure)

**Hardware mechanisms enabling persistence:**

```
╔═══════════════════════════════════════════════════════════╗
║  🔧 Hardware Protection Stack                              ║
╠═══════════════════════════════════════════════════════════╣
║                                                            ║
║  1️⃣  Volume Protection Flags                              ║
║     • disk1s2: (apfs, protect) ✓                          ║
║     • disk1s8: (apfs, protect) ✓                          ║
║                                                            ║
║  2️⃣  SEP Bypass Mechanism                                 ║
║     • Flag: effaceable_is_disabled                        ║
║     • Effect: Keys not erased during DFU                  ║
║                                                            ║
║  3️⃣  NAND Controller Block Skipping                       ║
║     • RTBuddy(ANS2/ANS3) queries SEP                      ║
║     • Protected blocks skipped during erase               ║
║                                                            ║
║  Result: Physical storage NOT wiped                        ║
║                                                            ║
╚═══════════════════════════════════════════════════════════╝
```


---

##  Scope and Capabilities

When monitoring infrastructure + network routing combine:

<table>
<tr><td width="33%">

**📺 Screen Monitoring**
- VoiceOver infrastructure
- All UI interactions
- Text input
- App content

</td><td width="33%">

**🎤 Audio Monitoring**
- Sound detection
- Audio routing
- Environmental audio
- Call audio

</td><td width="33%">

**⌨️ Input Tracking**
- Text selection
- Typing patterns
- Reading behavior
- User interactions

</td></tr>
<tr><td width="33%">

**⚙️ Process Control**
- App execution
- Resource allocation
- Background services
- System state

</td><td width="33%">

**🌐 Network Interception**
- All traffic routing
- DNS queries
- TLS decryption
- Complete visibility

</td><td width="33%">

**💾 Data Persistence**
- Survives factory reset
- Hardware-enforced
- Cannot be removed
- Invisible operation

</td></tr>
</table>

---

## Key Questions

**Questions Requiring Further Investigation:**

1. Is this behavior universal?
2. What is the infrastructure's purpose?
3. Who operates the backend systems?
4. How widespread is deployment?

---

##  Why This Matters

###  For Users

```
❌ Factory reset ≠ Clean device
❌ Security scan ≠ Actual security  
❌ Normal behavior ≠ Safe behavior
✅ Monitoring active = Invisible
```

**Privacy impact:**
- All screen content potentially monitored
- All audio potentially recorded
- All text input potentially tracked
- All network traffic potentially intercepted
- Cannot verify or disable

---

###  For Organizations

```
⚠️  Device sanitization procedures may be ineffective
⚠️  Factory reset cannot guarantee data removal
⚠️  Enterprise security assumptions may not hold
⚠️  Compliance requirements may not be met
```

---

###  For Security Professionals

```
🔴 CRITICAL: Standard validation produces false negatives
🔴 Factory reset insufficient for forensic sanitization
🔴 Hidden infrastructure undetectable by current tools
🔴 Cross-carrier coordination suggests organized platform
```

---

##  Repository Structure

```
stepped-on-silicon/
│
├── 📄 README.md                    ← Overview (this file)
├── 📄 IMPACT.md                    ← Detailed implications
│
├── 📂 devices/                     ← Per-device analysis
│   ├── iphone12/
│   └── iphone14_pro_max/   
│
└── 📂 infrastructure/              ← Network topology
    ├── taiwan_mobile.md  
    └── tmobile_usa.md
```


---

##  Disclosure Status

```
Analysis Period:  January 2026
Devices Examined: 3 (iPhone 12, 14 Pro Max, 15 Pro Max)
Device Type:      Consumer (no enterprise management)
This Disclosure: January 14, 2026
```

---

## Conclusion

Factory reset on iOS does not erase data as commonly understood. Multiple lines of evidence demonstrate:

1. **Pre-configured monitoring infrastructure** on clean devices
2. **Historical data surviving** factory reset (timestamped proof)
3. **Cross-carrier coordination** routing to shared private infrastructure  
4. **Invisible operation** using legitimate processes and certificates
5. **Hardware-enforced persistence** preventing removal

**The core issue:** A device can appear forensically "clean" while maintaining comprehensive monitoring capabilities and persistent configurations. Standard security validation produces false negatives. Users see normal behavior. Reality differs.

**For users:** Factory reset alone may not guarantee data removal or prevent device tracking.

**For security professionals:** Standard sanitization procedures may not achieve documented effectiveness.

**For researchers:** Independent verification strongly encouraged.

---

---

<p align="center">
<sub><em>This research documents observations under specific conditions. The gap between user expectations and actual behavior-regardless of underlying cause-represents a security and privacy concern worthy of examination.</em></sub>
</p>

---

**⚠️ Note:** This repository contains technical analysis and documentation. It does NOT contain exploit code, implementation details, or methods to circumvent security measures.
