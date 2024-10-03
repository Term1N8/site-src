---
title: "OpenCanary"
date: 2020-06-08T08:06:25+06:00
description: Sample post with multiple images, embedded video etc.
menu:
  sidebar:
    name: OpenCanary
    identifier: opencanary
    parent: tools
    weight: 10
hero: images/canary.jpg
categories:
- Basic
---

### What is OpenCanary?

[OpenCanary](https://opencanary.readthedocs.io/en/latest/) is the free open source version of [Canary](https://canary.tools/), a popular and advanced honeypot. OpenCanary takes a different approach to intrusion detection. Instead of monitoring network traffic for known malicious actions and signatures, OpenCanary is system meant to be attractive to attackers that will alert an administrator when an attack is detected.

### Problems With Typical IDS

If you have ever run an IDS like Snort, Suricata, Palo Alto, or similar, you know they flag a lot of traffic and make a lot of noise. Without fine-tuning the IDS properly, any legitimate warnings will be drowned in a mass of false positives and non-critical alerts. But even if the IDS administrator properly configures the system to decrease the alerts, there are still ways attackers can bypass an IDS. An attacker may use an unrecognizable zero day exploit, obfuscate their code, already be physically located inside with direct LAN access or leverage authentic means of remote access like a VPN to get access to a network without an IDS flagging the traffic.

### OpenCanary’s Solution

Thinking like an attacker, once they somehow manage to get a foothold into a network, they often attempt to pivot to other systems and gain as much control as possible. In many security breaches reviewed, the hackers had been active within the target’s network for weeks or longer laterally moving throughout the network. OpenCanary attempts to identify attackers that have managed to gain access by presenting itself as a target for the attacker. OpenCanary can host a variety of services like HTTP, SQL and FTP. When an attacker attempts to enumerate and exploit the OpenCanary services, notifications are sent to announce the intrusion attempt. Because of the level of interaction required to trigger an OpenCanary alert, false positives are extremely unlikely. Unless you happen to have an inquisitive tech savvy employee enumerating your network, alerts from the OpenCanary likely mean an attacker is actively within your network attempting to expand control.

### My Lab Setup

To try out OpenCanary, I am using Ubuntu 20.10 as my OpenCanary system and Kali 2020.3 as my “attacker”. Both of these systems are deployed using VirtualBox and configured within the same NAT network so they will have LAN communication.

![layout](/images/lab.png)

### Installation Steps

Update and install Python & other dependencies

```
sudo apt update && sudo apt upgrade && sudo apt dist-upgrade

sudo apt install python3-dev python3-pip python3-virtualenv
sudo apt install build-essential libssl-dev libffi-dev libpcap0.8-dev
```

Create a directory for OpenCanary and create a python virtual environment

```
mkdir yourdirectory
cd yourdirectory
virtualenv env/
. env/bin/activate
```

Install Python packages

```
pip install opencanary
pip install scapy
pip install pcapy
pip install rdpy
```

Create the config file

```
opencanaryd --copyconfig
nano /etc/opencanardy/opencanary.conf
```

Here, you can select which services you want to enable by changing the entry to “true”.

![layout](/images/options.PNG)

I recommend logging be set up using syslog using this configuration:

```
            "syslog-unix": {
                "class": "logging.handlers.SysLogHandler",
                "formatter":"syslog_rfc",
                "address": [
                    "localhost",
                    514
                ],
```

I will be using SMTP logging because I do not have a syslog server setup in this virtual environment:

```
        "handlers": {
            "SMTP": {
                "class": "logging.handlers.SMTPHandler",
                "mailhost": ["smtp.gmail.com", 587],
                "fromaddr": "noreply@yourdomain.com",
                "toaddrs" : ["youraddress@gmail.com"],
                "subject" : "OpenCanary Alert",
                "credentials" : ["youraddress", "abcdefghijklmnop"],
                "secure" : []
             }
         }
     }
 }
```

Once everything is configured as you want, use

```
opencanaryd --start
```

to start the service. You should soon receive an alert stating OpenCanary is running and a notification for each service started.

![layout](/images/email.PNG)

Simulating an attacker enumerating a device, I scanned the host and then tried to login to the discovered service:

![layout](/images/ftpfail.PNG)

As expected, I soon received an email notifying me of both the nmap scan and the failed login attempt:

![layout](/images/notify.PNG) I then spent some time testing different enumeration/exploitation methods to see if OpenCanary would detect it and was impressed with the accuracy of alerts as well as the details within the alerts. Even if an attacker manages to take control of the OpenCanary device, OpenCanary will be able to send a few warning alerts before it is compromised.

### Review

#### Things I like

*   Lots of service options to choose from
*   Variety of logging options
*   Low false-positive rate
*   Free way to detect threats when they matter most

#### Things I don’t like

*   You may need to create a new email to send alerts from
*   Email credentials stored in plain-text in configuration file
*   Setup is not super intuitive & you may need to engineer your own way to automate startup

### Configuration Tips

*   Avoid the services that may be recognized as OpenCanary (for example, the HTTP DiskStation login may be a give away if the attacker is familiar with OpenCanary)
*   Avoid services with conflicting Operating System identifiers
*   Fewer services will generate fewer alerts, try sticking to one or two and always monitor the alerts
*   Enabling scan detection will likely generate too much traffic to be taken seriously (most scans will already generate logs because of the fingerprinting methods used)
