---
title: "Bypassing Applocker+AMSI+Defender in 2024"
date: 2020-06-08T08:06:25+06:00
description: Sample post with multiple images, embedded video etc.
menu:
  sidebar:
    name: Bypassing Applocker+AMSI+Defender in 2024
    identifier: applocker
    parent: projects
    weight: 10
hero: images/vpn.jpg
categories:
- Basic
---

### Introduction

Malware development and EDR/AV evasion is a constant cat and mouse game. Evasion techniques are discovered, popularized, and then shortly detected. This is a combination of several techniques learned while taking Offsec's Offensive Security Experienced Professional (OSEP) course. As of July 2024, this method will successfully run Malicious PowerShell on a fully patched Windows system with Applocker, AMSI, Constrained Language Mode (CLM), and Defender. PowerShell can then be used to perform recon like WinPEAS.ps1, or run stagers for C2 malware.

### Primary Components

Several techniques will be used to successfully bypass the various protections in place. I will organize the code by the component that it will be bypassing.
*	InstallUtil Uninstall Method
*	Sleep check + encryption
*	Creating a new Powershell Runspace
*	Breaking AMSI through memory patching.


#### Bypassing Applocker

The first defense that needs to be bypassed is AppLocker. Everything else is dependent on the binary executing which will be blocked without some sort of bypass. The method I use in this code utilizes the Windows binary InstallUtil.exe. This binary is meant to help install and uninstall applications, and is allowed by default AppLocker rules because it is signed by Microsoft. By creating an Uninstall method in our code, we can execute the method by running our binary via InstallUtil.exe attempting to 'uninstall' the application. 


#### Bypassing Defender

Now that we are able to run the binary, we need to make it challenging for Defender to review the application for malicious activity. Windows Defender is relatively easy to bypass currently, and a simple sleep check and encryption will get the job done. 


#### Bypassing Constrained Language Mode (CLM)

PowerShell can be restricted to prevent calling .NET limiting PowerShell's capability for an attacker. Constrained Language Mode only monitors the runspace created by executing the PowerShell.exe binary. To bypass this, we can 'create our own' PowerShell by using the same DLLs that PowerShell does. This new Runspace will not be restricted by CLM. 

#### Bypassing AMSI

We can now break AMSI by memory patching. AMSI is currently fail open, so by interfering with the hook, commands will continue to work unmonitored. There are many AMSI bypasses, so I will leave this specific section of code out of this writeup, but you can include whichever works best for you. 

### Other Resources

*	https://github.com/api0cradle/UltimateAppLockerByPassList/blob/master/md/Installutil.exe.md
*	https://www.cyberark.com/resources/threat-research-blog/amsi-bypass-patching-technique
