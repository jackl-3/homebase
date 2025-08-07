
cd and view exercise files

use command `tshark -r directory-curiosity.pcap -T fields -e http.host -E header=y | awk NF | sort -r | uniq -c`

![[tsharkdirectories_command1.png]]

try `jx2-bavuong.com` in VirusTotal under "URL" option

![[tsharkdirectories_vtotal1.png]]

spooky

2nd question is from 1st question's answer 

![[tsharkdirectories_command2.png]]

use command to find ip addresses captured

![[tsharkdirectories_command3.png]]

try the first IP `141.164.41.174` is the first shown IP and appears to be sending HTTP traffic. checking the related domains on VirusTotal (page search)

![[tsharkdirectories_vtotaladdress.png]]

looks like this IP is associated with the bad domain

look for stream info

![[tsharkdirectories_command4.png]]

close but now follow node 1

![[tsharkdirectories_command5.png]]

we now see the server info for the domain

follow tcp stream 0 to find first file

![[tsharkdirectories_command6.png]]

we need to view the html body. there are three files: `vlauto.exe, 123.php, vlauto.php`

![[tsharkdirectories_objhighlight.png]]

first two probably entries are actually images but 3rd one is an actual file `123.php`

export http traffic objects to desktop

![[tsharkdirectories_command7.png]]

objects were extracted

look into folder to find executable

![[tsharkdirectories_lsfolder.png]]

we found it `vlauto[.]exe`

sha256 value of the bad file

![[tsharkdirectories_sha256sum.png]]

search up sha256 value on VirusTotal

![[tsharkdirectories_vtotal2.png]]

WOW

PEiD packer value in "Details" tab

![[tsharkdirectories_vtotalpeidpacker.png]]

look in "Behaviors" tab for detection classification

![[tsharkdirectories_vtotalclass.png]]

and we're DONE!