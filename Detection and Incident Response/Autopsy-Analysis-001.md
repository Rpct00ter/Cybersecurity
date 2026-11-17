Your task is to perform a manual analysis of the artifacts discovered by Autopsy to answer the questions below.


Odpalenie autopsy.
<img width="636" height="411" alt="image" src="https://github.com/user-attachments/assets/ddb99e58-2067-4e48-b290-b2f007cff3bb" />

<img width="1211" height="711" alt="image" src="https://github.com/user-attachments/assets/c64e6db9-f529-47b9-a92e-49d4ac7e191e" />
Connected to the VM, added specific disk for an analysis. 
<img width="1211" height="711" alt="image" src="https://github.com/user-attachments/assets/1c1ef8dd-8252-460d-901c-7604aee9077a" />

Succesfull extraction of an image:
<img width="1211" height="711" alt="image" src="https://github.com/user-attachments/assets/b95dac2f-9f69-4d1f-abf5-e1e2e2694032" />

Files are being analyzded, The first thing that I checked because it fully loaded were all operating system accounts. The list shows 8 user accounts: H4S4N,joshwa,keshav,sandhya,shreya,sivapriya,srini,suba and 4 Default System Accounts.
<img width="1211" height="725" alt="image" src="https://github.com/user-attachments/assets/cda086b3-cc21-4c62-b328-36ed51a58649" />
(better screenshot can be made, because not fully analyzed data is presented)

What is the MD5 hash of the E01 image?
MD5 hash is one of the file metadata. It can be verified by selecting a file and then opening a "file metadata" tab as shown in the screenshot below:
<img width="1400" height="553" alt="image" src="https://github.com/user-attachments/assets/e7c80294-e665-4473-b57b-8b8cdf8932c3" />

What is the computer account name?
Computer account name can be checked under Extracted Content -> Operating System Information:
<img width="1110" height="713" alt="image" src="https://github.com/user-attachments/assets/d83d4887-ab48-4b9a-b040-293d88498c79" />

List all the user accounts. (alphabetical order):
All OS User Accounts can be accessed under the tab with the same name. I checked only accounts that have path in the image.
<img width="1916" height="811" alt="image" src="https://github.com/user-attachments/assets/7bc12827-d14b-4283-b570-177938c00ce9" />

H4S4N,joshwa,keshav,sandhya,shreya,sivapriya,srini,suba

Who was the last user to log into the computer?
In the same tab as before, I sorted users by Date Accessed and read the most recent result:
<img width="569" height="122" alt="image" src="https://github.com/user-attachments/assets/8d5fc4bb-7fd3-43b1-a106-9ac86567dc49" />

What was the IP address of the computer?
What was the MAC address of the computer? (XX-XX-XX-XX-XX-XX)

In order to read that information I need to dig into filesystem more. Image has 3 volumes, but vol3 is the main volume was used for daily usage and data storage (it is the bigeest and contains many directories (including users directories). I went into vol3 -> Windows -> System32 -> config. Unfortunately I was unable to find any helpful data. So I started digging more. Under Program Files (x86) there was a whole directory named Look@LAN (which is an answer to the further question). After quick research it looks like this is some kind of vintage app for remote connections. After checking .ini file we can see IP and MAC.
<img width="773" height="812" alt="image" src="https://github.com/user-attachments/assets/e4ca4f44-ff4b-4e5e-bdcb-ca1fdbee906b" />


What is the name of the network card on this computer?
I once again dig into: I went into vol3 -> Windows -> System32 -> config. This time I opened SOFTWARE registry file and in Application tree selected Microsoft->Windows NT->Current Version->Network Card.
<img width="840" height="386" alt="image" src="https://github.com/user-attachments/assets/67901433-e94c-4bd2-a027-e6cf5a910fce" />

What is the name of the network monitoring tool?

A user bookmarked a Google Maps location. What are the coordinates of the location?
This one was easye, I just entered Web Bookmarks tab in autopsy, sorted them by URL, searched for google.maps and found the localisation:
<img width="1433" height="739" alt="image" src="https://github.com/user-attachments/assets/e6742f80-e3a7-4e9c-ad00-9cd9e15099ac" />

A user has his full name printed on his desktop wallpaper. What is the user's full name?
User's wallpaper should be stored in the user's NTUSER.DAT registry file - so that's where I went.
Specifically, in Control Panel\Desktop\WallPaper.
<img width="1837" height="1427" alt="image" src="https://github.com/user-attachments/assets/65be2b38-b650-4fc8-8a3a-b8d4fa8a4aed" />
There are 8 users, thats means I'm going to check 8 wallpaper localisations and then check them one by one, for user's name.
Fortunately after checking H454N, the second user was the one I was searching for:
<img width="1132" height="767" alt="image" src="https://github.com/user-attachments/assets/45f199f9-1768-4a54-8d19-16daccd15a2b" />


A user had a file on her desktop. It had a flag but she changed the flag using PowerShell. What was the first flag?
The same user found an exploit to escalate privileges on the computer. What was the message to the device owner?

My first step in order to answer this question was to search for powershell scripts. I used KeywordSearch for ".ps" and gained some results showing recent usage of powershell scripts by user: "shreya" Moreover those scripts are named very suspicious (exploit.ps1). All the suspicious exploits should be on users desktop.
<img width="637" height="686" alt="image" src="https://github.com/user-attachments/assets/bab74194-0b86-454c-b528-0bbce1587574" />
After investigating the script it looks like it attempts to bypass Windows UAC to gain elevated privileges by modifying a registry value and launching the SilentCleanup scheduled task, then creates a hacked.txt file containing with a message "flag{I-hacked-you}" if administrator privileges are obtained.
<img width="1098" height="811" alt="image" src="https://github.com/user-attachments/assets/f62e7390-821d-448c-8a3f-f23ddd9eaf52" />
I also found a .txt file with a changed flag:
<img width="636" height="367" alt="image" src="https://github.com/user-attachments/assets/e2567c73-2e1e-44af-9a87-9a11526d4082" />
In order to see the previous state of a file I digged into powershell history of the user located under: vol3/Users/shreya/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadLine.
<img width="981" height="832" alt="image" src="https://github.com/user-attachments/assets/8f45db8b-c8cf-4d93-89b0-27c6850d3fd3" />

2 hack tools focused on passwords were found in the system. What are the names of these tools? (alphabetical order)
In order to answer this question I decided to scroll through "Run Programs" tab in Autopsy and I was quick searching for suspicious program in \users\ localization. I was lucky and one under another I found 2 hacktools.
<img width="2403" height="1383" alt="image" src="https://github.com/user-attachments/assets/d118b13b-7012-446c-8035-699063f86699" />

There is a YARA file on the computer. Inspect the file. What is the name of the author?
In order to find this file quickly I used Keyword Search and searched for occurance of ".yar". Results show file located under: "C:\Users\H4S4N\Desktop\mimikatz_trunk\kiwi_passwords.yar".
<img width="2403" height="1383" alt="image" src="https://github.com/user-attachments/assets/9545f8d5-0062-47c9-9288-6aeab66f6202" />
Unfortunately desktop is cleared, but I managed to find something in the recent documents:
<img width="1018" height="367" alt="image" src="https://github.com/user-attachments/assets/528f4c25-64bb-40d5-9299-60107195bc1c" />
But still-it didn't have an author. I later tried to use Tools -> Search by Attribute function, but still could't find this file. As for the "Try Hack Me" is really buggy lately, I decided to browse for the author online.
<img width="698" height="175" alt="image" src="https://github.com/user-attachments/assets/4f9b4437-5cbf-4e77-ae74-e26d0e0e0bb0" />

One of the users wanted to exploit a domain controller with an MS-NRPC based exploit. What is the filename of the archive that you found? (include the spaces in your answer) 
I browsed what specifically is MS-NRPC and I was getting a lot of names like: "Zerologon" and "Netlogon". So I searched for keyword "logon" in Autopsy and found result documenting interactions with MS-NRPC based exploit (zerologon):
<img width="1410" height="530" alt="image" src="https://github.com/user-attachments/assets/fe74757b-f45b-4305-b403-f8abfd3060fa" />

So the quick summary shows that:
(summary of what happened on the device)
