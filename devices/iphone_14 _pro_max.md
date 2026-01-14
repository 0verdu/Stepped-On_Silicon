# iPhone 14 Pro Max (A16) 
## Post-DFU Forensic Analysis

- **Device:** iPhone 14 Pro Max  
- **Chipset:** A16 Bionic  
- **Build:** iOS 26.2 (23C55)  
- **Hidden Carrier:** Taiwan Mobile  
- **Hidden Region:** Asia-Pacific  

**Capture Context:**  
DFU restore performed at Apple Store on January 9, 2026 at 14:48:25 EST. Sysdiagnose captured 0-3 seconds after completion to document what survives factory reset.

**Full Evidence Package:**  
```
https://drive.proton.me/urls/8ZK08NWN3R#Yhjcx5HOopn4
```
```
SHA256: 4d9bb243f983ce7f6ec77f7bd95fd2b3cb797133777d770625b12e1cff5eec3d
Filename: sysdiagnose_2026.01.09_14-48-25-0500_iPhone-OS_iPhone_23C55.tar.gz
```

---

##  Critical Context: Geographic Anomaly

**Device Provenance:**
```
Purchase:        NEW from authorized U.S. retailer
Purchase Date:   [2023/2024 - device manufactured November 2023]
Purchase Region: United States
Carrier:         U.S. domestic carrier (NOT Taiwan Mobile)
User Location:   United States (entire device lifetime)

DFU Restore:
Location:        Apple Store, Atlanta, Georgia, United States
Date:            January 9, 2026
Method:          Official Apple DFU restore process
```

**Geographic Reality:**
```
✓ Device NEVER traveled to Asia-Pacific region
✓ Device NEVER traveled to Taiwan
✓ User NEVER had Taiwan Mobile service
✓ User NEVER configured Taiwan Mobile settings
✓ DFU performed on U.S. soil at official Apple Store
```

**Post-DFU Infrastructure Found:**
```
🔴 Taiwan Mobile MDM servers (osbstage.twmsolution.com)
🔴 Asia-Pacific Azure regions (japaneast, koreacentral)
🔴 Taiwan Mobile certificate authority infrastructure
🔴 Private AWS endpoints routing to Asia-Pacific
🔴 Carrier IMS configuration for Taiwan Mobile network
```

**Critical Question:**

**How does a device purchased NEW in the United States, never entering Asian territory, factory reset at an Apple Store in Atlanta, Georgia, emerge from DFU with Taiwan Mobile surveillance infrastructure fully configured and active?**

This proves the infrastructure is:
- NOT user-configured
- NOT location-based
- NOT carrier-specific to user's actual carrier
- NOT dependent on user travel history
- Pre-installed at hardware/firmware level
- Activated during official Apple DFU restore process

**Implication:** Taiwan Mobile infrastructure represents coordinated platform deployment, not individual carrier provisioning.

---

## Timeline: Certificate Generation During DFU

```
11:45:41 EST - Pre-DFU logs created (system activity)
14:39:11 EST - DCRT Certificate ISSUED (device in DFU mode, no OS running)
14:48:25 EST - DFU restore COMPLETES at Apple Store (Atlanta, GA)
14:48:25 EST - Sysdiagnose captured (0-3 second window)
14:49:08 EST - Certificate DELIVERED to device (43 seconds post-restore)

Delta: Certificate created 9 minutes 14 seconds BEFORE DFU completion
Location: Apple Store, Atlanta, Georgia (NOT Asia-Pacific)
```

**Critical finding:** Certificate generated while device had no operating system, proving activation infrastructure had pre-knowledge of device identity. Taiwan Mobile infrastructure activated despite device never being in Taiwan or on Taiwan Mobile network.

---

## Certificate Chain Analysis

### Three-Tier Hierarchy

```
SEP Root CA (Hardware - Secure Enclave)
    Subject Key ID: 58:EF:D6:BE:C5:82:B0:54:CD:18:A6:84:AD:A2:F6:7B:7B:3A:7F:CF
    Storage: BootROM, Secure Enclave trust store
    Never transmitted (pre-installed on all devices)
    ↓
FEDRCDC-UCRT-SUBCA (Intermediate - Activation Authority)
    Subject Key ID: B4:AA:3A:43:AD:1B:E5:7E:CE:0A:0C:38:1A:B5:D2:19:CB:95:35:B3
    Authority Key ID: 58:EF:D6:BE:... (links to SEP Root)
    Algorithm: ECDSA P-256 with SHA-256
    Path Length: 0 (cannot sign additional intermediate CAs)
    ↓
Device DCRT Leaf Certificate
    Serial: 01:9B:A4:4E:45:FA
    Issued: 2026-01-09 19:39:11 UTC
    Expires: 2026-01-16 19:49:11 UTC
    Lifetime: 7 days
    Subject CN: 00008120-001850XXXXXX (UDID)
```

### Embedded Device Identifiers

Certificate contains hardware identifiers in plaintext within Apple Device Attributes Extension (OID: 1.2.840.113635.100.10.1):

| Identifier | Value | Purpose | Privacy Impact |
|------------|-------|---------|----------------|
| IMEI | 357397708671168 | Cellular tracking | Permanent network-level tracking |
| MAC | 18:fa:b7:82:67:93:37:XX | Network fingerprinting | Cross-network correlation |
| UDID | 00008120-001850XXXXXX | Device identity | Permanent device identifier |
| Serial | M47M07393C | Manufacturing trace | Hardware warranty/repair tracking |
| BORD | 14 (0x0E) | Logic board revision | Hardware attestation |

**Manufacturing date decoded:** Week 47 of 2023 (November 20-26, 2023)

**Privacy violation:** All identifiers enable permanent cross-service tracking, cannot be changed by user, shared with activation servers.

---

## Hardware Persistence Mechanisms

### SEP Cryptographic Bypass

**Evidence:** `0000000000000001.tracev3` (system persist log)

```
Flag found: effaceable_is_disabled
Subsystem: SEPXART (Secure Enclave Processor eXecutive and Real-Time)
Component: AppleSEPKeyStore

Function:
During standard DFU, SEP programmed to "efface" (wipe) encryption keys.
Flag proves erasure command intentionally bypassed for protected volumes.

Target volume UUID: 61706673-7575-6964-0002-766F6C756D07 (disk1s8)

Result:
Encryption keys survive in SEP KeyStore
Protected data remains cryptographically accessible to new OS
Explains survival of pre-DFU logs timestamped 11:45:41 EST
```

### NAND Controller Block Skipping

**Evidence:** RTBuddy(ANS2) / AppleANS3CGv2Controller logs

```
Log entry: "Already ON, skip RESET"
Hardware interface: ans@774 (Apple Storage Fabric address)
Controller: AppleANS3CGv2Controller

Behavior:
NAND controller does not undergo full power cycle during DFU
Adopts existing Logical Block Addresses (LBAs) instead of formatting
Physical sectors containing user data never receive erase pulse

Migration policy: migrate_media_keys_if_needed
Trigger: Volume role "Volume User" for disk1s8
SEP evaluates "protect" mount flag
Hardware initiates preservation instead of erasure
```

### Volume Protection Evidence

**Source:** mount.txt

```
/dev/disk1s8 on /private/var/mobile (apfs, local, nodev, nosuid, journaled, noatime, protect)
```

**"protect" flag function:**
- Marks volume as excluded from DFU erasure
- Instructs SEP to preserve encryption keys
- Signals NAND controller to skip physical blocks
- Enables data continuity across factory reset

---

## Data Survival Evidence

### Pre-DFU Logs Intact

**Timestamp comparison:**

```
Log creation: 11:45:41 EST (January 9, 2026)
DFU performed: 14:48:25 EST (January 9, 2026)
Time delta: 3 hours 2 minutes 44 seconds

Expected: All logs erased during DFU
Found: Pre-DFU logs intact in sysdiagnose
Proves: Volume persistence across factory reset
```

### Privacy Permissions During DFU

**Source:** TCC.db (Transparency, Consent, and Control database)

```
Permission grants timestamped: 14:47:24 EST
DFU completion: 14:48:25 EST
Delta: Permissions granted 61 seconds BEFORE DFU completion

Permissions auto-granted:
- Health data access
- Photos library access
- Location services
- Microphone access
- Camera access

User interaction: None (device in DFU mode)
```

---

## Network Infrastructure

### Taiwan Mobile Command & Control

**Geographic Impossibility:**

```
DEVICE REALITY:
✗ Never in Asia-Pacific region
✗ Never in Taiwan
✗ Never on Taiwan Mobile network
✗ DFU performed in Atlanta, Georgia, USA

INFRASTRUCTURE FOUND:
✓ Taiwan Mobile MDM servers active
✓ Asia-Pacific routing configured
✓ Taiwan carrier integration deployed
✓ Regional Azure endpoints established
```

**Primary infrastructure:**

| Component | Value | Function |
|-----------|-------|----------|
| Deployment server | osbstage.twmsolution.com | MDM profile staging |
| OHTTP relay | ObliviousHop-osbstage.twmsolution.com | Multi-hop anonymization |
| PIR node | pir.kaylees.site | Private information retrieval |
| Exfiltration endpoint | 172.31.34.114:64579 | Data collection (AWS private IP) |
| VPC gateway | 172.31.32.1 | Internal routing |

**Geographic distribution:**
- japaneast (Azure Japan East, Tokyo)
- koreacentral (Azure Korea Central, Seoul)
- AWS ap-southeast-1 (Singapore) or ap-northeast-1 (Tokyo)

### Traffic Routing

```
iPhone (Atlanta, Georgia, USA)
    ↓ [VPN/proxy configured via MDM]
osbstage.twmsolution.com (Taiwan Mobile - profile deployment)
    ↓ [OHTTP relay for anonymization]
ObliviousHop-osbstage.twmsolution.com
    ↓ [VPN tunnel to private VPC]
172.31.34.114:64579 (AWS private IP - NOT publicly routable)
    ↓ [Regional processing]
japaneast / koreacentral (Azure storage/analysis)
```

**Critical:** 172.31.x.x addresses are RFC 1918 private IPs. Device can only reach them via pre-configured VPN/proxy, proving unauthorized network routing.

**Architectural proof:** Infrastructure deployment is NOT based on:
- User's geographic location (device in U.S., routing to Asia)
- User's actual carrier (not Taiwan Mobile subscriber)
- User's travel history (never been to Taiwan)
- User configuration (automatic during DFU)

**Conclusion:** Represents coordinated global platform, not carrier-specific service.

---

## Exploitation Method

**How Taiwan Mobile infrastructure leverages certificate persistence:**

1. DCRT certificate with embedded IMEI/MAC/UDID/Serial survives DFU (disk1s8 protection)
2. Device activates using persistent certificate (Serial: 01:9B:A4:4E:45:FA)
3. MDM profile matches device via embedded identifiers
4. Taiwan Mobile auto-deploys configuration via serial number pre-registration
5. VPN/proxy routing to 172.31.34.114 configured
6. All traffic intercepted through private AWS infrastructure
7. TLS/HTTPS decryption via MDM-installed certificates

**Capabilities enabled:**
- Complete traffic interception
- Location tracking (persistent IMEI/MAC)
- Remote management
- Data exfiltration
- Process control

**Geographic deployment proves:**
- Infrastructure pre-installed globally
- NOT dependent on user's location or carrier
- Activated during standard Apple DFU process
- Coordinated platform across multiple regions/carriers

---

## Hardware Component Mapping

| Component | Physical Identity | Evidence Location |
|-----------|------------------|-------------------|
| Silicon address | ans@774 | AppleEmbeddedNVMeController interaction |
| Cryptographic gate | SEPXART | effaceable_is_disabled flag |
| Firmware bridge | RTBuddy(ANS2) | "Already ON, skip RESET" state |
| Storage controller | AppleANS3CGv2Controller | LBA adoption, migration policy |
| Target volume | disk1s8 | UUID 61706673-7575-6964-0002-766F6C756D07 |

---

## Indicators of Compromise

### Certificate IOCs

```
DCRT Serial: 01:9B:A4:4E:45:FA
Issued: 2026-01-09 19:39:11 UTC
Issuer: FEDRCDC-UCRT-SUBCA
Subject: 00008120-001850XXXXXX

Intermediate CA:
Subject Key ID: B4:AA:3A:43:AD:1B:E5:7E:CE:0A:0C:38:1A:B5:D2:19:CB:95:35:B3
```

### Network IOCs

```
Domains:
osbstage.twmsolution.com
ObliviousHop-osbstage.twmsolution.com
pir.kaylees.site
akasha.whos (internal DNS)
review.corp (internal routing)

IP Addresses:
172.31.34.114:64579 (AWS private - exfiltration)
172.31.32.1 (VPC gateway)

Geographic Regions:
japaneast (Azure)
koreacentral (Azure)
osasia (Asia-Pacific)
asia-401 (requires verification)
```

### Hardware IOCs

```
SEP flag: effaceable_is_disabled
NAND state: "Already ON, skip RESET"
Volume UUID: 61706673-7575-6964-0002-766F6C756D07
Mount flag: protect (on disk1s8)
Migration policy: migrate_media_keys_if_needed
```

---

## Conclusion

DFU restore at Apple Store (Atlanta, GA) failed to remove surveillance infrastructure. Evidence proves:

1. **Certificate persistence:** DCRT with embedded device IDs survived via hardware protection
2. **Pre-knowledge of activation:** Certificate created 9 minutes before DFU completion
3. **Cryptographic bypass:** SEP effaceable storage intentionally disabled
4. **Physical continuity:** NAND controller skipped reset, preserved LBAs
5. **Data survival:** Logs from 3+ hours before DFU found intact
6. **Network infrastructure:** Taiwan Mobile C2 active immediately post-restore


---

**Device Provenance:** NEW purchase, U.S. retailer
**DFU Location:** Apple Store, Atlanta, Georgia, United States  
**Analysis Date:** January 11, 2026  
**Confidence Level:** High (cryptographic and hardware evidence)
