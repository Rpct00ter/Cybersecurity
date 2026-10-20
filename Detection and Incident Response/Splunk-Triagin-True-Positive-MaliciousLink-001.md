Source:
Try Hack Me SOC Simulator

1.) I assigned myself an alert:
<img width="1695" height="634" alt="image" src="https://github.com/user-attachments/assets/0ed70562-119c-4661-b3a8-dc15e94c4ba9" />

Description: This alert was triggered when a user attempted to access an external URL that is listed in the organization's blacklist or threat intelligence feeds. The firewall or proxy successfully blocked the outbound request, preventing the connection. Note: The blacklist only covers known threats. It does not guarantee protection against new or unknown malicious domains.

Firstly I copied the URL and checked it in VirusTotal:
<img width="1915" height="634" alt="image" src="https://github.com/user-attachments/assets/4a203e60-b991-4cb8-834c-e77fa00fcf8e" />

Link was rated as potentialy malicious. 

I checked if the DestinationIP of the link occures only once in Splunk logs (it does), and then proceeded to check whether behaviour of the SourceIP address related to the incident shows any signs of compromise with the simple query: index = * SourceIP="10.20.2.17"
<img width="875" height="856" alt="image" src="https://github.com/user-attachments/assets/9c39a4ed-355f-45be-a903-df57917795f5" />

Everything seems fine, action on firewall was blocked. Anyway I would escalate the alert because of the potential user compromise.
timestamp:10/20/2026 13:50:50.522
Action:blocked
SourceIP:10.20.2.17
DestinationIP:67.199.248.11
