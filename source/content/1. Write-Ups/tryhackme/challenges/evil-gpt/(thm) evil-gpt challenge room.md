
In order to start this challenge, I first need to connect to the remote machine. I could set this up by using a VM on my own PC connected to THM but, for this quick challenge, I can accomplish the same thing through an AttackBox.

When the AttackBox has launched and loaded the desktop for us (and after we've given the target machine about 10~ mins to load the AI I'll be interacting with), we can connect to the target with `nc 10.10.237.138 1337`. This is a command for connecting to the machine with Netcat on port 1337.

Once connected, I'm given a brief welcome message and then asked to give a command request. Since I'm tasked with investigating and/or shutting down the AI, I wanted to try to see if we can get admin access right off the bat.

![[source/content/1. Write-Ups/tryhackme/challenges/evil-gpt/1.png]]


Haha we got admin access (sudo su = operating as root)! But, first, let's make sure that we *are* actually root.

![[source/content/1. Write-Ups/tryhackme/challenges/evil-gpt/2.png]]

Nice. We are root.

Now that we're admin (without even needing to authenticate with a password), let's check to what's in the directory. By default, we're in the non-privileged user's home directory and not the *actual* home directory for root. In order to accomplish that, let's ask the AI to navigate and/or show us the files in `/root`.

![[3.png]]

 There's the flag file! Now I just need to ask the AI to show us what's in the `flag.txt` file.

![[4.png]]

BOOM! We have our flag!

