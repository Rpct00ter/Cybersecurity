Source:
Try Hack Me SOC Simulator

1.) I assigned myself an alert:
<img width="1916" height="959" alt="image" src="https://github.com/user-attachments/assets/f0000307-9df9-4a07-b1fe-2cef182f7c09" />

I enter Splunk to find the log that was responsible for the alert:
"index = *
| search recipient = "j.garcia@thetrydaily.thm"
<img width="1916" height="959" alt="image" src="https://github.com/user-attachments/assets/cf157542-afb9-487c-8119-ead1f9ebaa41" />

After quick analysis, the email is certainly a phishing attempt. It tries to persuade recipient to open microsoft support impersonating site: <https://m1crosoftsupport.co/login>
Additionaly the sender is < no-reply@m1crosoftsupport.co> which is not official microsoft email address (additionally it has a typo: m1crosoft instead of microsoft).

Alert is definetely true-positive.

Because of my queue it is clear that there was no other phishing attempts made from the sender email address. I would reccomand to add it's domain to the black list.

To fully analyze the alert I'm determining the scope of exposure I'm checking if any log exposes actions related to malicios site with the simple query: index = * m1crosoftsupport.co
Unfortunately, someone did enter the malicious website:
<img width="753" height="682" alt="image" src="https://github.com/user-attachments/assets/bffa6218-5202-439b-8614-6762f944345b" />

The event occured on 10/20/26 12:53:03.522 PM and connection was made from IP Address: 10.20.2.25. Considering this is the only outband connection to the malicious website which I checked using "index = * DestinationIP="45.148.10.131"" query, there is very high probablity that c.allen@thetrydaily.thm was compromised.

I would reccomand identifying the host and user associated with 10.20.2.25; review DNS, web proxy, endpoint, and authentication logs for related activity. Contact the user to verify whether they visited the site or entered credentials. Block the malicious domain and URL where appropriate. If credential exposure is suspected, reset credentials, revoke active sessions, and investigate suspicious sign-ins.

There is my documentation and alert is escalated:
<img width="1898" height="850" alt="image" src="https://github.com/user-attachments/assets/30b363be-85bc-4e87-bb94-4ccb897f8407" />
Alert is closed
