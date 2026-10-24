# Security Investigation - VoIP Vishing Call Analysis

## Source
LetsDefend - VoIP Challenge

## Scenario
James received a suspicious call from someone claiming to represent his bank and asking for sensitive information. The supplied `Traffic.pcapng` capture was analyzed in Wireshark to identify the call, its participants and duration, and information disclosed during the conversation.

## Tasks
| Challenge question | Answer |
|---|---|
| How many RTP packets were in the traffic? | **18,348** |
| When did the fake call with James start? | **2024-05-03 20:36:36** |
| What is James's phone number? | **7001** |
| How long was the call with the bank? | **00:01:35** |
| What is the phone number of the bank caller? | **01326947697** |
| What is the name of the bank calling? | **Bank of Wealth** |
| What is James's Social Number? | **5678** |

## Tools & Technologies
- Wireshark
- RTP Player (build in)

---

# Investigation

### VoIP background

Voice over IP (VoIP) carries voice calls over IP networks. Call setup and control are commonly handled by **SIP (Session Initiation Protocol)**, while the voice itself is commonly carried in **RTP (Real-time Transport Protocol)** packets, usually over UDP. A typical call includes SIP signaling such as `INVITE`, provisional responses like `100 Trying` and `180 Ringing`, a final response such as `200 OK`, and acknowledgements. `BYE` is used to end the call. RTP packets carry encoded audio during the conversation.

This separation matters during an investigation: SIP can reveal caller/callee identifiers and call setup/teardown timing, while RTP packet counts and playback help analyze the media. In this challenge, listening to the reconstructed audio was necessary to identify the claimed bank and the sensitive information disclosed.

### 1. Initial Alert / Detection

The challenge supplied an archive named `Bank Incident.7z`, containing `Traffic.pcapng`. The scenario described a suspected vishing attack: a caller impersonated James's bank and requested sensitive information.

I opened the capture in Wireshark and began by filtering for RTP traffic.

![Opening the supplied capture in Wireshark](https://github.com/user-attachments/assets/966e3bec-58aa-4d53-a76c-042e4ceace91)

### 2. Triage

I used the display filter:

```wireshark
rtp
```

The filtered packet count showed **18,348 RTP packets**. To answer the start-time question in the requested date-and-time format, I changed Wireshark's display setting to **View → Time Display Format → Date and Time of Day**, with seconds displayed.

![RTP packet filter and packet list](https://github.com/user-attachments/assets/23e71ad8-33db-4f77-847a-094ad7202fc7)

![Changing Wireshark time display format](https://github.com/user-attachments/assets/1daf16e8-6326-46da-8bdd-e2743589875c)

### 3. Evidence Analysis

I then inspected SIP signaling using:

```wireshark
sip
```

Following the relevant SIP call showed the `INVITE` and call setup messages. The `To` field identified James's extension/number as `7001`. The packet list showed signaling between `192.168.245.1`, `192.168.245.128`, and `192.168.245.130`; the screenshots alone do not establish that these IPs map to specific real-world people.

![SIP signaling packets](https://github.com/user-attachments/assets/b6f67e42-4622-4643-b3ab-cd49d825e88e)

![Following the SIP call](https://github.com/user-attachments/assets/b36cf824-4cee-4d74-b241-756c929c4e1b)

Next, I opened **Telephony → VoIP Calls**. The call summary showed a duration of **00:01:35**, James as `7001`, and the caller number `01326947697`.

![Wireshark VoIP Calls summary](https://github.com/user-attachments/assets/9fe02ac6-d500-4cf3-80a0-f15956e075b1)

### 4. Correlation

The SIP and RTP evidence supports this call sequence:

- **2024-05-03 20:36:36:** the call begins, based on the first relevant RTP packet timestamp and the challenge's requested time format.
- SIP signaling establishes the call between the caller and James (`7001`).
- The VoIP Calls summary reports a duration of **1 minute 35 seconds**.
- The call is completed; the SIP sequence includes call teardown (`BYE`) and a successful response.

![SIP call setup and teardown sequence](https://github.com/user-attachments/assets/6d750188-785e-48bd-9793-b292da19cb4f)

### 5. Threat Intelligence

The primary evidence in this case is the packet capture and reconstructed call, rather than an IP/domain reputation indicator. The decisive evidence came from the call's signaling metadata and the audio content. The caller claimed to represent **Bank of Wealth** and James disclosed the number **5678** as his Social Number during the conversation.

I used Wireshark's **Telephony → VoIP Calls → Play Streams** workflow to open the RTP Player and listen to the reconstructed audio. The screenshots document the call list and RTP playback interface; the bank name and number are answers obtained by listening to the call, not values inferred from packet metadata alone.

![VoIP Calls window and call details](https://github.com/user-attachments/assets/1bac23f4-0c0c-4624-8090-b19d67f252a8)

![RTP Player with reconstructed audio streams](https://github.com/user-attachments/assets/da16784e-7a75-4583-8fd4-c96c05c77acd)

### 6. MITRE ATT&CK

| Technique | ID | Evidence |
|---|---|---|
| Phishing: Spearphishing Voice *(possible mapping)* | T1566.004 | A caller used a voice call to solicit sensitive information while impersonating a bank. |
| Impersonation | T1656 | The caller claimed to represent “Bank of Wealth” to gain James's trust. |

These mappings describe the observed social-engineering behavior; the packet capture does not establish the caller's real identity or broader infrastructure.

## Findings

The PCAP contains a VoIP call consistent with a vishing attack. SIP analysis identified James's number (`7001`), the caller number (`01326947697`), the call start time (`2024-05-03 20:36:36`) and duration (`00:01:35`). Listening to the RTP audio identified the claimed bank as **Bank of Wealth** and confirmed that James disclosed the value **5678** as his Social Number.

**Verdict:** True Positive - vishing / voice-based social engineering
**Severity:** High - sensitive personal information was disclosed during the call
**Affected Assets:** James (`7001`); VoIP call represented in `Traffic.pcapng`; exposed Social Number value `5678`

## Response & Recommendations

- Contact James through a trusted channel, explain the impersonation attempt, and advise him not to provide personal information to unsolicited callers.
- If `5678` is a real sensitive identifier, follow the organization's privacy/identity-protection process; avoid reproducing it in broadly accessible incident systems beyond what is required for the lab.
- Preserve the PCAP and document the caller number `01326947697`; correlate it with other reports and available telephony logs before blocking or attributing it.
- Consider awareness training and a clear procedure for verifying bank requests through independently obtained official contact details.
