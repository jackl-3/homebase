
cd and view exercise files

use command `tshark -r directory-curiosity.pcap -T fields -e http.host -E header=y | awk NF | sort -r | uniq -c`

![[1.1.png]]

try `jx2-bavuong.com` in VirusTotal under "URL" option

![[1.2.png]]

spooky

2nd question is from 1st question's answer 

![[1. Write-Ups/tryhackme/challenges/tshark challenge 2 write-up/2.png]]

use command to find ip addresses captured

![[3.1.png]]

try the first IP `141.164.41.174` is the first shown IP and appears to be sending HTTP traffic. checking the related domains on VirusTotal (page search)

![[3.2.png]]

looks like this IP is associated with the bad domain

look for stream info

![[1. Write-Ups/tryhackme/challenges/tshark challenge 2 write-up/4.1.png]]

close but now follow node 1

![[1. Write-Ups/tryhackme/challenges/tshark challenge 2 write-up/4.2.png]]

we now see the server info for the domain

follow tcp stream 0 to find first file

![[1. Write-Ups/tryhackme/challenges/tshark challenge 2 write-up/5.1.png]]

we need to view the html body. there are three files: `vlauto.exe, 123.php, vlauto.php`

![[6.png]]

first two probably entries are actually images but 3rd one is an actual file `123.php`

export http traffic objects to desktop

![[7.1.png]]

objects were extracted

look into folder to find executable

![[7.2.png]]

we found it `vlauto[.]exe`

sha256 value of the bad file

![[8.png]]

search up sha256 value on VirusTotal

![[9.1.png]]

WOW

PEiD packer value in "Details" tab

![[9.2.png]]

look in "Behaviors" tab for detection classification

![[10.png]]

and we're DONE!