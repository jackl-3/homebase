
Once the target machine has fully booted and loaded the DocIntel application, I can begin looking at the correspondence from the business that sent over the suspicious files for investigation (on this machine, the web browser already has the login credentials saved so login is pretty simple).

Upon opening the application, to answer the first question as to who sent the message, we can immediately see the author:

![[emailauthor.png]]

His name is Oliver Bennett. 😁

Now, before I can continue, I'll need to grab the files from the message itself by clicking on and the downloading the zip file attached:

![[archivepwd.png]]

Once I've downloaded the zip, the next step is to navigate to the downloads folder (`~/Downloads` or `/home/ericatracy/Downloads`) and extract the files from the archive with password `Panda321!` (password is obfuscated in prompt):

![[extractwithpwd.png]]

Once finished, the next thing to do is to, within a terminal, navigate to the folder where the samples were extracted in order to get the SHA1 value of `pRsm.dll`:

![[sha1sum.png]]

Now here I momentarily thought that it was that SHA256 hash that was needed before I realized that was not the right one 😂. Rather, the SHA1 hash that I need is `9d1ecbbe8637fed0d89fca1af35ea821277ad2e8`.

So, armed with the correct hash, the next step was to plug that hash of `pRsm.dll` into [VirusTotal](https://www.virustotal.com/) and see what it reports:

![[vtotal1.png]]

Wow, it's a bad file. That's for sure.

The next question is asking us to locate the malware framework that uses this file. Out of curiosity, I was exploring the "Community" tab and discovered that a recent user submission shows the file is used by "MgBot" to capture audio:

![[vtotal2.png]]

(It's worth noting that the APT group associated with this `.dll` has the same name/word that was used in the password for the `.zip` from earlier 👀)

Turns out that MgBot is the malicious framework that uses these DLLs. However, the hint from the room says we can also search the internet as well for an article that describes the framework using these files:

![[googleresult.png]]

Probably note necessarily intended but Google's Search Labs AI gives us an answer right off the bat but, for the sake of the room, I went to the WeLiveSecurity article anyway:

![[article1.png]]

Cool. The article is not very long but goes into good detail on the Panda APT group and its distribution of malware. Reading through it reveals this tidbit:

![[article2.png]]

This shows what plugins Panda are using and confirms that the files were given to investigate are the same.

At this point, I can say with confidence that we are definitely looking at files that would be/are being use by an actual APT (Advanced Persistent Threat) group. Cool and a little spooky 👀.

The next question asks: what technique does `pRsm.dll` use? Well, since it's asking in context of MITRE ATT&CK, I decided to start with a simple search through their site to see if MgBot was cataloged:

![[mitre1.png]]

Sure enough, the first result is what I'm looking for and I'm presented with the following information:

![[mitre2.png]]

In the "Techniques Used" table, I can see different IDs assigned to different use cases or actions that MgBot is capable of performing against a system. Since I know that `pRsm.dll` is used for reported as being used audio capture from VirusTotal, the corresponding technique it uses has an ID of "T1123".

For our ongoing investigation, the next thing I need to do is find the URL that was reported as malicious on the date of November 2nd of 2020 and then provide the address as defanged.

Defanging, which preserves and address as human-readable but prevents programs from creating clickable links (protects against accidents), is pretty simple but can be tedious to do manually by hand. I'll use [CyberChef](https://cyberchef.org/) to paste the address and have it automatically convert it into a manner that is defanged.

 Going back to the article that I found from my Google search, I scanned for any mention of a report date. Sure enough, in Table 1, a *partially* defanged URL is shown as reported for 2020-11-02 or Nov 2, 2020.

![[article3.png]]

However, this is only *partially* defanged. Completing the defanging is pretty easy: pasting the URL into the "Input" box, removing the two brackets in the address, and then hitting **BAKE!** gives me the following result in the "Output" box:

![[defangedurl.png]]

The completely defanged URL is `hxxp[://]update[.]browser[.]qq[.]com/qmbs/QQ/QQUrlMgr_QQ88_4296[.]exe`.

The next question I need to answer is similar to the previous: I now need to locate the IP address of the C&C (Command and Control) server that was reported September 14th, 2019.

Luckily, in the same article, I can see under a section labeled "Network" that the there is a IP reported on 2019-09-14 but, like the previous address, is again only partly defanged. 

![[article4.png]]

Quickly throwing the IP into CyberChef provides me the defanged IP address of  `122[.]10[.]90[.]112`.

Advancing onward, the final portion of the challenge is needing me to determine the MD5 hash of the Android spyware hosted on the same IP in June of 2025.

I decided to start this one off by looking up the IP in VirusTotal to see if there was anything there that could help me:

![[vtotal3.png]]

Hmmmmm... well, from one of the 4 files listed under "Communicating Files" in the "Relations" tab, there is one file that's indicated as being for Android. I decided to follow it (achieved by clicking on the name):

![[vtotal4.png]]

Boom. First entry in the "Basic properties" gives me the MD5 has of `951f41930489a8bfe963fced5d8dfd79`.

With that last bit of information, the challenge is completed! Looks like the firm that was hit with those suspicious files was/is definitely part of some sort of attack as they are registered as being used by an international APT group. Yikes 😬