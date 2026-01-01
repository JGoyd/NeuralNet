```
  
  Artifact                : netstat.txt
  SHA-256                 : b57e84f448867761dbbd1022b3142fe8bf10c2b8
                            9010db40cd2de34a8f46dcb3

  Artifact                : skywalk.txt
  SHA-256                 : 499b974a59b75633f25dce74efccf1ccd23f77a9
                            f977c97361b9084dbe8f861a

  Artifact                : ifconfig.txt
  SHA-256                 : 546d00ee292a18207d9b85132c81f2d83750a738
                            eb11ab26cf059f1d235a1488

---------------------------------------------------------------------


=====================================================================
```
| Potential Objection                                 | Evidence / Report Section     | Response / Explanation                                                                                                                                                                                                                                                                      |
| --------------------------------------------------- | ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **“This is just normal mDNS traffic”**              | Sections 4.2, 7.1, 12.1       | Observed volume: 84.5 MB (vs typical KB-scale for mDNS). Packet count (~2,994) and RX:TX ratio (67:1) exceed expected discovery traffic by 3+ orders of magnitude. Calculations in Section 11.2 show unrealistic beacon rates if this were legitimate discovery traffic.                    |
| **“Airplane Mode doesn’t guarantee zero traffic”**  | Sections 1, 2.1, 6.1, B.1     | Cellular interfaces pdp_ip0–10 confirmed off (`link quality: -2`) and WiFi infrastructure disabled. Despite this, AWDL and utun2 interfaces actively transmit and receive packets. Indicates **hardware control bypass**, not incidental background traffic.                                |
| **“Status reporting might be a bug”**               | Sections 1.1, 3.3, 10.2       | AWDL reports `inactive` while TXSTART flag and 2,657 packets show active transmission. Combined with utun2 parallel stack and IDS binding, false reporting appears intentional, not accidental.                                                                                             |
| **“Encrypted payloads could be benign”**            | Sections 3.2, 10.2, 11.1-11.4 | While payload content cannot be verified, traffic volume, sustained throughput (2.2–11 Mbps), encapsulation via utun2, and trusted process masquerading strongly suggest purpose beyond discovery. Not consistent with expected mDNS behavior.                                              |
| **“Telemetry capture might be artifact”**           | Sections 11.5, 12             | Multiple sources (ifconfig.txt, netstat.txt, skywalk.txt) show consistent results: interface flags, routing tables, multicast memberships, and traffic volumes all corroborate the same behavior. Single capture session validated across files.                                            |
| **“This could be a benign architectural choice”**   | Sections 10.2, 10.3           | Evidence shows deliberate design: parallel network stack (utun2), trusted process exploitation (mDNSResponder, IDSNexusAgent), false status reporting, and distributed service ecosystem (replicatord, symptomsd, fitnessintellig). Supports **autonomous mesh node operation**, not a bug. |
| **“Physical limitations might prevent throughput”** | Section 11.1                  | Shannon-Hartley calculation: AWDL 20 MHz, S/N ~15 dB → theoretical 12.6 MB/s capacity. Observed 2.2–11 Mbps throughput is well within physical limits. Activity is feasible without exotic encoding.                                                                                        |
| **“Process privilege not confirmed”**               | Sections 11.4, 3.2            | Process access inferred via kernel network stack control and flow-switch bindings. Direct privilege confirmation (ps/taskinfo/codesign) missing, but available telemetry strongly indicates elevated entitlements.                                                                          |
| **“User might still control device”**               | Sections 6.1, 10.3            | Airplane Mode command disables infrastructure radios, but AWDL and utun2 remain active. 84.5 MB transferred without user control. User cannot verify traffic, disable interfaces, or inspect payload.                                                                                       |
| **“Traffic could be legitimate IDS/messaging”**     | Sections 3.2, 4.2, 7.1        | Even if IDS traffic, volume is far above typical protocol expectations. Parallel tunnel stack and encapsulation obscure payload, suggesting possible data aggregation or covert transfer beyond intended messaging.                                                                         |
