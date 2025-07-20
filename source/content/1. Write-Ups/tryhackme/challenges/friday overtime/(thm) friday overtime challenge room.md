%%  %%
login using creds (they were already saved in the browser on the attackbox)

we can immediately see the name of the person who sent us the files to view

![[1. Write-Ups/tryhackme/challenges/friday overtime/1.png]]

Oliver Bennett 😁

before we can move forward, we need to grab the files by looking at message itself with the zip file attached:

![[2.1.png]]

download the zip, navigate to download folder, and extract with password `Panda321!`

![[2.2.png]]

next, navigate to samples folder within terminal to determine SHA1 value of `pRsm.dll`

![[2.4.png]]

our SHA1 hash is `9d1ecbbe8637fed0d89fca1af35ea821277ad2e8 `(also got the SHA256 value as well thinking i was getting the SHA1 hash lol)

plugging in the SHA1 hash of `pRsm.dll` into VirusTotal

![[1. Write-Ups/tryhackme/challenges/friday overtime/3.1.png]]

it's a bad file, to be sure

out of curiosity, the "Community" tab has a recent user submission that states the file is used by MgBot to capture audio plus other

![[1. Write-Ups/tryhackme/challenges/friday overtime/3.2.png]]

(even has Panda in the name of the APT group too)

turns out MgBot is the bad framework that uses these dlls. however, the hint from the room says we can also search the internet as well for an article that describes the framework that uses these files

![[3.3.png]]

Google's ai already gives us the answer but clicking the link of the first article

![[3.4.png]]

gives us a brief read of Evasive Panda APT using these files

![[3.5.png]]

and also confirms what plugins they're using (the same ones we were given to investigate)

to find out what technique the `pRsm.dll` is classified by MITRE, we'd need to visit the site search for MgBot

![[1. Write-Ups/tryhackme/challenges/friday overtime/4.1.png]]

and then we're presented with this page

![[1. Write-Ups/tryhackme/challenges/friday overtime/4.2.png]]

since we know that the dll is used for audio capture, the MITRE technique it uses is "T1123"

to find the defanged url of the malicious download location first seen on 11/02/2020, we can actually look back at the article we first found

![[1. Write-Ups/tryhackme/challenges/friday overtime/5.1.png]]

however, this is only *partially* defanged. we can finish the defanging by plugging into CyberChef (after removing the partial defang from the article's URL)

![[1. Write-Ups/tryhackme/challenges/friday overtime/5.2.png]]

our defanged url is `hxxp[://]update[.]browser[.]qq[.]com/qmbs/QQ/QQUrlMgr_QQ88_4296[.]exe`

and for the defanged IP of the C&C server reported in 9/14/2019, we again can see from the article

![[1. Write-Ups/tryhackme/challenges/friday overtime/6.png]]

meaning that our C&C IP is `122[.]10[.]90[.]112`

for the last question, have to find the md5 hash of the Android spyware hosted on the same IP in June 2025

plugging 120.10.90.112 into VirusTotal

![[1. Write-Ups/tryhackme/challenges/friday overtime/7.1.png]]

hmm well one of the 4 files is for the Android platform. let's click on it

![[1. Write-Ups/tryhackme/challenges/friday overtime/7.2.png]]

boom. the details show that the md5 hash is `951f41930489a8bfe963fced5d8dfd79`

All done!
