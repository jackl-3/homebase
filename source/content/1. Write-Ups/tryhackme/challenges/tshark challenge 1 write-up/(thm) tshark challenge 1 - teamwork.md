
For this challenge, we're told that we'll be investigating some captured traffic as part of SOC team (cool). To do this, after connecting to VM ourselves or using the Attackbox (what I used for this challenge), we need to navigate our terminal to `~/Desktop/exercise-files` on the remote machine:

![[a.png]]

The folder contains one `.pcap` file that we'll be working with for this challenge room. Now, because we're also told that we'll need to investigate domains using [VirusTotal](https://www.virustotal.com/gui/home/upload), we'll need to keep a browser tab of that opened for later.

---

Our first question is asking to provide it with the full URL of the suspicious domain captured. In order to scan the file for domains before we can determine where to go from here, we need to instruct TShark with the command:

`tshark -r teamwork.pcap -T fields -e http.host -E header=y | awk NF | sort -r | uniq -c | sort -r | nl`

This command will do the following:

a) `tshark -r teamwork.pcap` - read the `teamwork.pcap` file

b) `-T fields -e http.host -E header=y` - filter the information with to only show packets that contain `http.host` for domains that we can view and choose to investigate

c) `awk NF | sort -r | uniq -c | sort -r |` - remove blank spaces, sort results, and only show unique entries (omit duplicates), and sort again

d) `nl` - show the results as a numbered list

Truthfully, one could probably just use `tshark -r teamwork.pcap -T fields -e http.host -E header=y` and arrive at the same answer but, without the extra modifiers to remove duplicates and clear empty space, the answer would most likely be very cluttered and messy to read. Thankfully, with the command above, we get this:

![[1. Write-Ups/tryhackme/challenges/tshark challenge 1 write-up/1.png]]

We can immediately from a glance what our suspicious domain is (defanged, of course): `hxxp[://]www[.]paypal[.]com4uswebappsresetaccountrecovery[.]timeseaways[.]com/`. (Definitely a legit web address, no problem there 🙃)

Now that we have the sus IP domain address, we are then asked to determine when this domain was first submitted/reported to VirusTotal. This one is actually not so complicated.

On VirusTotal, we have to select the URL option on the homepage and then paste in our malicious domain. Once done, we can switch to the "Details" tab to get a little more info on this submitted domain:

![[1. Write-Ups/tryhackme/challenges/tshark challenge 1 write-up/2.png]]

Highlighted in the screenshot is the date when this domain was first submitted to VirusTotal as a suspicious address. In other words, this report occurred at essentially 10:52 pm UTC on April 17th, 2017. (over 9 years ago!)

The next question of the challenge wants us to ascertain the known service that this domain is trying to impersonate. Well, from the domain itself, we can very easily see that it's attempting to come off as a domain for PayPal, a well-known payment processor. Mhm.

So, since we are investigating this domain, we need to now find out what the IP address is for the URL in question.

This one is a little bit tricky. In order to locate this one, we actually need to view the details for `timeseaways.com` where the domain is *actually* being hosted. In order to find this, one would need to do the following.

On the initial VirusTotal page we got after putting in the fake PayPal domain, we need to click on that address:

![[1. Write-Ups/tryhackme/challenges/tshark challenge 1 write-up/4.1.png]]

Doing so directs us to a new page that looks similar to the previous one (as it still shows the bogus PayPal address). But, this time, we are shown new information regarding the relations this domain has. Navigating to the "Details" tab again shows usd this as one can see that tab is populated with more details than the Details tab from the page before 

Even though we are given new info, it still doesn't give us the IP address *of the domain itself*. Rather, we are given IPs of sibling domains. For this, one would need to repeat the previous step once more and select the domain associated with fake PayPal one:

![[1. Write-Ups/tryhackme/challenges/tshark challenge 1 write-up/4.2.png]]

And then once again head over to the Details tab for `timeseaways.com`. Once there, the first category we're shown is "Passive DNS Replication" which records associated IP address for the domain at different dates:

![[4.3.png]]

But which one is it? It could be any one of these, right?

Well, considering what we've seen so far, we know that the malicious domain was reported as suspicious in 2017. Since that is before the year 2018 but also after 2015, we can assume that, at the time of the report, the IP address would've been `184[.]154[.]127[.]226`. since this IP is associated with the DNS replication closest to the date the domain was reported.

Now, the next question is a bit more difficult: we need to locate the email address that was used. Funny thing is, we aren't told exactly *how* it was used? Was it used to register the domain, was it used to create the website, was it used *on* the website? The only way to find out was to start digging.

At first, I scoured through the details again of our associated domains, looking for an email address that could be the answer. Unfortunately, I was not able to locate an email address from the domain info on VirusTotal that seemed to fit the bill (they were mostly official addresses used by the registrar which be very unlikely to have been used for these malicious domains).

Unable to answer the question, this lead me into my next line of thinking: what if the email that it's looking for had been used on the website in question? If someone who was unaware that they had been directed to a fake PayPal support page had input their credentials and tried logging in, it's possible that their credentials are saved in the `pcap` file. After all, their attempt to log in using that information would undoubtedly result in HTTP traffic which, in this case, would've definitely been captured. Let's see if that's true or not.

To determine if my hypothesis was correct that HTTP traffic had been captured, I formulated the following command for TShark:

`tshark -r teamwork.pcap -Y "ip.addr == 184.154.127.226" -Y 'http contains ".com"'`

This command is telling TShark to look at packets that contain the IP address of `184.154.127.226` (regardless of the direction) and then filter those packets further by only showing the ones that have any mention/associating with ".com":

![[1. Write-Ups/tryhackme/challenges/tshark challenge 1 write-up/5.1.png]]

Perfect! We can see a pretty good list of captured stuff relating to an HTTP page (a few pages, in fact). Because we're limited to the terminal and thus don't have the same navigational features that Wireshark offers, we can't "follow" the HTTP traffic to look inside some of the files like `suspecious.php`, or `visit.php`. What we can do, however, is export the contents of the captured HTTP packets so we can further investigate them:

![[5.2.png]]

This command is basically telling TShark to export anything related to HTTP traffic to the desktop folder `teamwork-extraction`. 

In order to view the exported files, we'd need to navigate to the new folder with `cd ~/Desktop/teamwork-extraction`. Once in that folder, a quick `ls` will show us a structure very similar to the results we got from the filtering we did earlier; we see images, php files, JavaScript, and subfolders just like the results from the filtering we had TShark do on the `teamwork.pcap` file.

With these files now available to view, it was time for me to check if an email address had been used and subsequently captured within all this information. There were quite a few options with all them being the files ending in `.php` but the singular file that caught my attention was `login.php`. Obviously, this was intended to be used as the login page of the suspicious website so let's `cat` the file and see what it says:

![[5.3.png]]

NICE! So, while the output is a jumbled, we can see that someone did use their login creds on this page! They used the email address of `johnny5alive[at]gmail[.]com` with their password being `johnny5alive` (not a good password 😑).

With that, we completed the challenge!

---

After reviewing, this was a pretty fun room! I enjoy puzzles and it was cool using the previous material taught by THM about TShark to solve the puzzles this room had to offer. 😁

