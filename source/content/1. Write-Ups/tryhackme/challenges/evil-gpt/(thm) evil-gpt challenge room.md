
connect to machine with virtualbox

then ask for admin acces (why not go straight to it?)

we got admin access (sudo su = operating as root). first thought is to view the directory we're in and its contents:

![[content/1. Write-Ups/tryhackme/challenges/evil-gpt/1.png]]

then we should see which user we are

![[content/1. Write-Ups/tryhackme/challenges/evil-gpt/2.png]]

nice. root.

since we're looking for flag, we need to comb through the folders. but we have to do it with verbiage that would translated into a command we can have the ai run

let's see what's in the directory for the root user

![[3.png]]

EY there's the flag file! now we gotta get the ai to spill the beans!

![[4.png]]

BOOM! we have our flag!

