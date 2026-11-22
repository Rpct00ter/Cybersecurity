# Digital Forensics Investigation - Autopsy-Analysis-001

## Source:
Try hack me: Disk Analysis & Autopsy

## Scenario:  
Your task is to perform a manual analysis of the artifacts discovered by Autopsy to answer the questions below.

# Tasks:

## Starting Autopsy and loading the evidence

I started Autopsy and connected to the VM. I then added the disk image as a data source for analysis.

![Starting Autopsy](https://github.com/user-attachments/assets/ddb99e58-2067-4e48-b290-b2f007cff3bb)

![Autopsy case/data source view](https://github.com/user-attachments/assets/c64e6db9-f529-47b9-a92e-49d4ac7e191e)

I connected to the VM and added the specific disk for analysis. The image was successfully loaded and its files became available for analysis.

![Successful image extraction](https://github.com/user-attachments/assets/b95dac2f-9f69-4d1f-abf5-e1e2e2694032)

## Initial review of operating-system accounts

As the data was being analyzed, the first thing I checked-once the relevant results were available-was the list of operating-system accounts. The extracted results showed eight named user accounts: `H4S4N`, `joshwa`, `keshav`, `sandhya`, `shreya`, `sivapriya`, `srini`, and `suba`. The image also shows default/system accounts; I focused on the named accounts associated with user profiles.

![Operating-system user accounts](https://github.com/user-attachments/assets/cda086b3-cc21-4c62-b328-36ed51a58649)

The named user accounts, in alphabetical order, were:

`H4S4N, joshwa, keshav, sandhya, shreya, sivapriya, srini, suba`

The account identified as the last user to log into the computer was `sivapriya`. To determine this, I returned to the same user-account results and sorted by the Date Accessed information, then inspected the most recent entry.

![Last-user information](https://github.com/user-attachments/assets/8d5fc4bb-7fd3-43b1-a106-9ac86567dc49)

The computer account name was also identified in Autopsy under **Extracted Content → Operating System Information**. The computer account name was `DESKTOP-0R59DJ3`.
<img width="1110" height="713" alt="image" src="https://github.com/user-attachments/assets/d83d4887-ab48-4b9a-b040-293d88498c79" />
## What is the MD5 hash of the E01 image?

The MD5 hash is available in the image's file metadata. I selected the image and opened the **File Metadata** tab to inspect the metadata shown by Autopsy.

![E01 hash in File Metadata](https://github.com/user-attachments/assets/e7c80294-e665-4473-b57b-8b8cdf8932c3)

The MD5 hash of the E01 image was:

`3f08c518adb3b5c1359849657a9b2079`

## What was the IP address and MAC address of the computer?

In order to read that information I need to dig into filesystem more. Image has 3 volumes, but vol3 is the main volume was used for daily usage and data storage (it is the bigeest and contains many directories (including users directories). I went into vol3 -> Windows -> System32 -> config. Unfortunately I was unable to find any helpful data. So I started digging more. Under Program Files (x86) there was a whole directory named Look@LAN (which is an answer to the further question). After quick research it looks like this is some kind of vintage app for remote connections. After checking .ini file we can see IP and MAC.

![Look@LAN configuration artifact](https://github.com/user-attachments/assets/e4ca4f44-ff4b-4e5e-bdcb-ca1fdbee906b)

The recovered network information was:

- IP address: `192.168.130.216`
- MAC address: `08-00-27-2c-c4-b9`

## What is the name of the network card on this computer?

I returned to the registry artifacts in `vol3 → Windows → System32 → config`. This time I opened the `SOFTWARE` registry hive and navigated through the Application/registry tree to **Microsoft → Windows NT → Current Version → Network Card** to inspect the network-card information.

![Network-card registry information](https://github.com/user-attachments/assets/67901433-e94c-4bd2-a027-e6cf5a910fce)

The network-card name was:

`Intel(R) PRO/1000 MT Desktop Adapter`

## What is the name of the network monitoring tool?

While searching the filesystem for the IP and MAC information, I noticed the `Look@LAN` directory under **Program Files (x86)**. Inspecting the application's configuration file provided the network information described above. The network-monitoring tool was `Look@LAN`.

## A user bookmarked a Google Maps location. What are the coordinates of the location?

This was relatively straightforward. I opened the **Web Bookmarks** artifacts in Autopsy, sorted/inspected the URLs, and searched for a Google Maps bookmark. The bookmark contained the location information.

![Google Maps bookmark](https://github.com/user-attachments/assets/e6742f80-e3a7-4e9c-ad00-9cd9e15099ac)

The coordinates were:

`12°52'23.0"N 80°13'25.0"E`

## A user has their full name printed on their desktop wallpaper. What is the user's full name?

I expected the user's wallpaper setting to be stored in their `NTUSER.DAT` registry hive, so I examined the wallpaper setting under:

`Control Panel\Desktop\WallPaper`

![Wallpaper registry location](https://github.com/user-attachments/assets/65be2b38-b650-4fc8-8a3a-b8d4fa8a4aed)

There were eight user profiles, which meant I would have to check the wallpaper locations for each profile and inspect the images individually. Fortunately, after checking `H4S4N`, the second user I examined was the one I was looking for.

![Wallpaper showing the user's name](https://github.com/user-attachments/assets/45f199f9-1768-4a54-8d19-16daccd15a2b)

The name printed on the wallpaper was `Anto Joshwa`.

## A user had a file on her Desktop with a flag but changed the flag using PowerShell. What was the first flag? The same user found an exploit to escalate privileges. What was the message to the device owner?

My first step in investigating this was to search for PowerShell scripts. I used Autopsy's Keyword Search for `.ps` and found results indicating recent PowerShell script activity associated with the user `shreya`. One of the scripts had a suspicious name: `exploit.ps1`. As the suspicious script appeared to be located on the user's Desktop, I investigated it further.

![PowerShell keyword-search results](https://github.com/user-attachments/assets/bab74194-0b86-454c-b528-0bbce1587574)

After examining the script, it appeared to attempt a UAC bypass to obtain elevated privileges. It does this by manipulating a registry value and launching the SilentCleanup scheduled task. The script also contains logic intended to create a `hacked.txt` file with the message `flag{I-hacked-you}` if the relevant administrator-privilege condition is met.

![Contents of exploit.ps1](https://github.com/user-attachments/assets/f62e7390-821d-448c-8a3f-f23ddd9eaf52)

I also found a `.txt` file containing a flag that had been changed.

![Text file with changed flag](https://github.com/user-attachments/assets/e2567c73-2e1e-44af-9a87-9a11526d4082)

To inspect the previous state and determine what had been changed, I examined the user's PowerShell history located at:

`vol3/Users/shreya/AppData/Roaming/Microsoft/Windows/PowerShell/PSReadLine`

![PowerShell history](https://github.com/user-attachments/assets/8f45db8b-c8cf-4d93-89b0-27c6850d3fd3)

The history showed commands interacting with the text file and changing its contents. From the recorded command history, I recovered the original flag:

`flag{HarleyQuinnForQueen}`

The message the exploit was designed to place in `hacked.txt` was:

`flag{I-hacked-you}`

## Two password-focused hacking tools were found on the system. What are their names?

In order to answer this question I decided to scroll through "Run Programs" tab in Autopsy and I was quick searching for suspicious program in \users\ localization. I was lucky and one under another I found 2 hacktools.
<img width="2403" height="1383" alt="image" src="https://github.com/user-attachments/assets/d118b13b-7012-446c-8035-699063f86699" />

## There is a YARA file on the computer. Inspect the file. What is the name of the author?

To find the YARA file, I used Keyword Search and searched for `.yar`. The search results pointed to a file in the following location:

`C:\Users\H4S4N\Desktop\mimikatz_trunk\kiwi_passwords.yar`

![YARA keyword-search result](https://github.com/user-attachments/assets/9545f8d5-0062-47c9-9288-6aeab66f6202)

Unfortunately desktop is cleared, but I managed to find something in the recent documents:
<img width="1018" height="367" alt="image" src="https://github.com/user-attachments/assets/528f4c25-64bb-40d5-9299-60107195bc1c" />
But still-it didn't have an author. I later tried to use Tools -> Search by Attribute function, but still could't find this file. As for the "Try Hack Me" is really buggy lately, I decided to browse for the author online.
<img width="698" height="175" alt="image" src="https://github.com/user-attachments/assets/4f9b4437-5cbf-4e77-ae74-e26d0e0e0bb0" />
  

The author was `Benjamin DELPY (gentilkiwi)`.

## One of the users wanted to exploit a domain controller with an MS-NRPC-based exploit. What was the filename of the archive?

I first looked into what MS-NRPC exploitation referred to and came across names such as Zerologon and Netlogon. I then searched for the keyword `logon` in Autopsy and found an artifact associated with the Zerologon exploit archive.

![Zerologon archive artifact](https://github.com/user-attachments/assets/fe74757b-f45b-4305-b403-f8abfd3060fa)

The archive filename was:

`2.2.0 20200918 Zerologon encrypted.zip`

## Chronological summary of the activity

Based on the artifacts and timestamps visible in the screenshots, the following is a cautious reconstruction of the activity and findings. This is not a complete event timeline, and finding a tool or archive does not, by itself, prove that it was successfully used.

- **February 6, 2021 - PowerShell script on Shreya's Desktop:** `exploit.ps1` was present on the Desktop, with a timestamp around 03:53 PST. Its contents suggest an attempted UAC bypass/privilege escalation involving registry modification and the SilentCleanup scheduled task.
- **February 6, 2021 - Wallpaper artifact:** An image in Joshwa's Downloads folder displayed the name Anto Joshwa. The artifact shown in the screenshot has a timestamp around 04:12 PST.
- **February 6, 2021 - Flag modification evidence:** The PowerShell history and text-file artifacts associated with Shreya show commands that interacted with a file and changed flag content. The earlier flag was recovered from the history: `flag{HarleyQuinnForQueen}`.
- **February 6, 2021 - PowerShell activity:** The script also contained logic to create `hacked.txt` with `flag{I-hacked-you}` if its privilege condition was satisfied. The script's presence and contents demonstrate the intended behavior, but do not alone establish that the task succeeded.
- **February 6, 2021 - Zerologon archive:** An archive named `2.2.0 20200918 Zerologon encrypted.zip` was identified in the artifacts in connection with MS-NRPC/Zerologon.
- **Other findings during the examination:** The system contained artifacts associated with Lazagne and Mimikatz, a Mimikatz-related YARA file attributed to Benjamin DELPY (gentilkiwi), Look@LAN network-monitoring software, network configuration information, and a Google Maps bookmark.

Overall, the artifacts indicate that this computer contained tools and files associated with password extraction, PowerShell-based privilege-escalation activity, and Zerologon/MS-NRPC exploitation. The report records the findings and the investigative steps used to locate them, while distinguishing artifact evidence from conclusions about successful execution.
