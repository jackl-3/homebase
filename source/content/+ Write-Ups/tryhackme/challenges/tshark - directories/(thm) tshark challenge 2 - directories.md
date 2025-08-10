
Once the machine has started and a connection to interact with it has been established, the first step is to change directory into  `~/Dowloads/excercise-files`.

Now that I'm in it, my first task is to find the malicious address and provide it in a defanged manner. To accomplish this, I construct my TShark command as follows:

`tshark -r directory-curiosity.pcap -T fields -e http.host -E header=y | awk NF | sort -r | uniq -c`

This tells TShark to filter by packets that have HTTP fields and to list unique address (to avoid repetition) and then indicate how many times each packet has been recorded.

![[tsharkdirectories_command1.png]]

So, most of them seem legit but... `jx2-bavuong.com` seems out of place compared to the rest. As I'm told that we need to use VirusTotal to help through this challenge, I'm going to search for this suspicious URL on that website:

![[tsharkdirectories_vtotal1.png]]

Spooky 👀. It is, indeed, a malicious website. However, in order to record it properly, the correct answer is `jx2-bavuon[.]com`. The brackets around the period before "com" should prevent software from automatically making the entry click-=able (and in turn avoid accidental navigation to the bad website).

The second piece of information I need to find (the amount of times the URL is captured) can actually be extrapolated from the results we used in our first answer:

![[tsharkdirectories_command2.png]]

The malicious site is recorded 14 times in the `.pcap` file.

Now that I know the website URL, what is the IP address? To determine this, I decided to tell TShark to sort for packets that contain any mention of `.com` (and sort so the results aren't super long):

![[tsharkdirectories_command3.png]]

At first glance, it may seem like a lot of information but it's actually only a few addresses! 

They are:

`141.164.41.174`
`192.168.100.116`
`204.79.197.200`

Cool thing about VirusTotal is that there's a lot of report information in regards to anything associated with a domain, file, or hash. Looking more into the associations with `jx2-bavuong.com`, the "Relations" tab gives us a section labeled "**Passive DNS Replication**". Under that, I am presented with many IP addresses that have been linked to this domain. 

Now, thanks to the functionality of web browsers, rather than going through each IP individually and comparing them to the three TShark highlighted for me, I will try doing a search off the page to see if they're listed, starting with `141.164.41.174`:

![[tsharkdirectories_vtotaladdress.png]]

Looks like this IP is associated with the bad domain. 😁

Continuing onward, I next need to report the server info belonging to domain. In this question, it's stated that I need to follow the "first TCP stream" and carefully consider the output. Now, the statement "first TCP stream" essentially means to have TShark follow the TCP stream (since we need descriptive info), displaying ASCII, and starting at position 0. The reason it needs to be zero is because TShark starts streams with 0 then increments onward rather starting at 1 and incrementing. The following command is instructing TShark to follow TCP stream 0 and display its content as ASCII (or human-readable):

![[tsharkdirectories_command6.png]]

In the second segment of information displayed, I can see that section "Server" is recorded as `Apache/2.2.11 (Win32) DAV/2 mod_ssl/2.2.11 OpenSSL/0.9.8i PHP/5.2.9`.

With the same output, it can also be determined that there are three files called on in the HTML body of the packet: `vlauto.exe, 123.php, vlauto.php`. 

This also solves the next part in determining the name of the first file:

![[tsharkdirectories_objhighlight.png]]

But the real answer needs to be defanged: `123[.]php`.

Moving on to the next part of the challenge, I'm instructed to now report on the SHA256 hash of the malicious file. Since everything I need is probably captured within the `.pcap` file, I decided to see if exporting the HTTP contents would help in this endeavor. To export HTTP objects, the command for doing so is as follows:

![[tsharkdirectories_command7.png]]

Perfect. Time to dive into the folder. 😎

What's in it?

![[tsharkdirectories_lsfolder.png]]

Well, the only executable extracted is `vlauto[.]exe` (the other `.exe` is just a duplicate of the other).

In Linux, running `sha256sum <filename>` will return the SHA256 value of said file:

![[tsharkdirectories_sha256sum.png]]

Perfect. Now it's time to see what VirusTotal has to say about it:

![[tsharkdirectories_vtotal2.png]]

Wow, this is a REAL bad file!

Our report needs to include, now, the "PEiD packer" value. But what is it?

From the previous image, the "Details" tab contains much more specifics on the file. Specifially (again using "Find in page" 😋, I can see that the value is..

![[tsharkdirectories_vtotalpeidpacker.png]]

..a .NET executable!

To wrap it all up, the last piece of information I need is what "Lastline" has flagged this particular file as. A quick look in the "Behaviors" tab shows that Lastline flagged it as:

![[tsharkdirectories_vtotalclass.png]]

A MALWARE TROJAN! Yikes!

Haha and, with that final answer, we have completed the challenge!