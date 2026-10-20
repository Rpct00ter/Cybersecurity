IntroductionToPhishing
Lab Loaded, first alert:
<img width="1918" height="961" alt="image" src="https://github.com/user-attachments/assets/ac5a56a3-8f50-467c-b967-cb63ce4db716" />

Assigned an Alert, 2'nd alert occured in the meantime:
<img width="1920" height="964" alt="image" src="https://github.com/user-attachments/assets/f8dedca9-cfa2-4221-b682-9e6eb754e9f1" />

#1: Analysis of the first alert:
I entered the playbook
<img width="1920" height="964" alt="image" src="https://github.com/user-attachments/assets/263f1541-8ca5-4c32-85ed-e3e036d86fb7" />

First thing I did was to find the log that created an alert. To do so I quered:
"index = *
| search sender = "onboarding@hrconnex.thm"":
<img width="1639" height="744" alt="image" src="https://github.com/user-attachments/assets/b5137ee1-f5a2-42c3-bbf8-309de78a2f2a" />

The first thing I noticed is the sender looks very convincing. onboarding@hrconnex.thm sound like a legitible email address. Link URL doesn't look sketchy as well - it has the similar domain as sender. IP is 10.10.160.225:8989.

Now I change my query to look for a recipient:
"index = *
| search recipient = "j.garcia@thetrydaily.thm""
<img width="626" height="962" alt="image" src="https://github.com/user-attachments/assets/2d7a551a-04cc-490b-85f3-3d0643d9c334" />
There are no other messages that would rise suspicions, especially that content of some messages directs that the recepient could be after interview.

Decision:
I'm deciding, that the alert is False Positive. And I'm starting to fill the documentation.
<img width="1597" height="664" alt="image" src="https://github.com/user-attachments/assets/2b1b62d1-2d7f-4bfd-9762-7184cc633d0c" />

Documentation:
<img width="1916" height="959" alt="image" src="https://github.com/user-attachments/assets/d77fc6b0-95ce-4faa-9564-6ed017eb7f50" />

Alert Closed
