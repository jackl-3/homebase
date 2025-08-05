Building the concepts taught in [the previous room](https://tryhackme.com/room/tsharkthebasics) that covered basic use of TShark, this room goes into more detail of its features that are also available in Wireshark such as getting statistics, more advanced filtering, and gathering more info from `pcap` files.

---

The first two modules cover the usage of TShark to obtain statistics from `.pcapng` files containing captured packets. At first, we are given a brief summary that:

+ stats are applied to all packets unless a filter for displaying results is specified
+ most commands shown in the room are command-line interface version of features in Wireshark (further expanded on in [this room](https://tryhackme.com/room/wiresharkpacketoperations))
+ that TShark explains/shows the parameters used at the beginning part of the output line
+ if desired (which I can see being pretty helpful for at-a-glance identification), use of the flag `--color` instructs TShark to show the results with a color mimicking Wireshark packet highlighting

Additionally, we are provided with a small table outlining some helpful options:

| **Parameter** | **Purpose**                                                                                                                                                                                                                                                                                                                                                                                               |
| :-----------: | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|    --color    | Wireshark-like colourised output:<br><br>- `tshark --color`                                                                                                                                                                                                                                                                                                                                               |
|      -z       | Statistics<br><br>There are multiple options available under this parameter. You can view the available filters under this parameter with:<br><br> `tshark -z help`<br><br>Sample usage:<br><br>- `tshark -z filter`<br><br>Each time you filter the statistics, packets are shown first, then the statistics provided. You can suppress packets and focus on the statistics by using the `-q` parameter. |

The first section briefly covers the use of `-z io,phs -q` to show the protocol hierarchy of the packets in the `pcapng` file. According to the author, it can be used to get a better picture in order to locate an abnormal event:

```
user@ubuntu$ tshark -r demo.pcapng -z io,phs -q
===================================================================
Protocol Hierarchy Statistics
Filter: 

  eth                                    frames:43 bytes:25091
    ip                                   frames:43 bytes:25091
      tcp                                frames:41 bytes:24814
        http                             frames:4 bytes:2000
          data-text-lines                frames:1 bytes:214
            tcp.segments                 frames:1 bytes:214
          xml                            frames:1 bytes:478
            tcp.segments                 frames:1 bytes:478
      udp                                frames:2 bytes:277
        dns                              frames:2 bytes:277
===================================================================
```
(pasted from room)

The keyword `udp` could also be added to filter/show stats based on the UDP protocol:

```
user@ubuntu$ tshark -r demo.pcapng -z io,phs,udp -q
===================================================================
Protocol Hierarchy Statistics
Filter: udp

  eth                                    frames:2 bytes:277
    ip                                   frames:2 bytes:277
      udp                                frames:2 bytes:277
        dns                              frames:2 bytes:277
===================================================================
```


Or we can use the parameter that allows us to see packet lengths in a tree view using `-z plen,tree -q` to show them:

```
user@ubuntu$ tshark -r demo.pcapng -z plen,tree -q

=========================================================================================================================
Packet Lengths:
Topic / Item       Count     Average       Min val       Max val     Rate (ms)     Percent     Burst rate    Burst start  
-------------------------------------------------------------------------------------------------------------------------
Packet Lengths     43        583.51        54            1484        0.0014        100         0.0400        2.554        
 0-19              0         -             -             -           0.0000        0.00        -             -            
 20-39             0         -             -             -           0.0000        0.00        -             -            
 40-79             22        54.73         54            62          0.0007        51.16       0.0200        0.911        
 80-159            1         89.00         89            89          0.0000        2.33        0.0100        2.554        
 160-319           2         201.00        188           214         0.0001        4.65        0.0100        2.914        
 320-639           2         505.50        478           533         0.0001        4.65        0.0100        0.911        
 640-1279          1         775.00        775           775         0.0000        2.33        0.0100        2.984        
 1280-2559         15        1440.67       1434          1484        0.0005        34.88       0.0200        2.554        
 2560-5119         0         -             -             -           0.0000        0.00        -             -            
 5120 and greater  0         -             -             -           0.0000        0.00        -             -            
-------------------------------------------------------------------------------------------------------------------------
```

From the display, there are 15 packets that range from 1280-5119 bytes long with an average length in that category of 1440.67 bytes in length, for instance. Good to know.

We can also construct our command to TShark to look for endpoints with the use of `-z endpoints,ip -q` or other common options such as:

| **Filter** | **Purpose**                                       |
| ---------- | ------------------------------------------------- |
| eth        | - Ethernet addresses                              |
| ip         | - IPv4 addresses                                  |
| ipv6       | - IPv6 addresses                                  |
| tcp        | - TCP addresses<br>- Valid for both IPv4 and IPv6 |
| udp        | - UDP addresses<br>- Valid for both IPv4 and IPv6 |
| wlan       | - IEEE 802.11 addresses                           |
With the provided demonstration show if we, for example, wanted to view endpoints of IPv4 addresses:

```
user@ubuntu$ tshark -r demo.pcapng -z endpoints,ip -q
================================================================================
IPv4 Endpoints
Filter:
                       |  Packets  | |  Bytes  | | Tx Packets | | Tx Bytes | | Rx Packets | | Rx Bytes |
145.254.160.237               43         25091         20            2323          23           22768   
65.208.228.223                34         20695         18           19344          16            1351   
216.239.59.99                  7          4119          4            3236           3             883   
145.253.2.203                  2           277          1             188           1              89   
================================================================================
```


Conversations between addresses enables us to see the flow of traffic between two different endpoints (and can be viewed in multiple formats as well). This option can be utilized by passing `-z conv,ip -q` with the rest of the command in the terminal to show conversations between addresses:

```
user@ubuntu$ tshark -r demo.pcapng -z conv,ip -q  
================================================================================
IPv4 Conversations
Filter:
                                           |       <-      | |       ->      | |     Total     |    Relative    |   Duration
                                           | Frames  Bytes | | Frames  Bytes | | Frames  Bytes |      Start     |             65.208.228.223   <-> 145.254.160.237           16      1351      18     19344      34     20695     0.000000000        30.3937
145.254.160.237  <-> 216.239.59.99              4      3236       3       883       7      4119     2.984291000         1.7926
145.253.2.203    <-> 145.254.160.237            1        89       1       188       2       277     2.553672000         0.3605
================================================================================
```

The results are somewhat long and are a little difficult to make out exactly what's going on. But we we can see from this example that it's showing us stats from communication between 3 unique addresses.

The final part of the module shows us that we can pass  `-z expert -q` with our command to show automatic comments that one could also view in Wireshark's expert info:

```
user@ubuntu$ tshark -r demo.pcapng -z expert -q

Notes (3)
=============
   Frequency      Group           Protocol  Summary
           1   Sequence                TCP  This frame is a (suspected) spurious retransmission
           1   Sequence                TCP  This frame is a (suspected) retransmission
           1   Sequence                TCP  Duplicate ACK (#1)

Chats (8)
=============
   Frequency      Group           Protocol  Summary
           1   Sequence                TCP  Connection establish request (SYN): server port 80
           1   Sequence                TCP  Connection establish acknowledge (SYN+ACK): server port 80
           1   Sequence               HTTP  GET /download.html HTTP/1.1\r\n
           1   Sequence               HTTP  GET /pagead/ads?client=ca-pub-2309191948673629&random=1084443430285&lmt=1082467020
           2   Sequence               HTTP  HTTP/1.1 200 OK\r\n
           2   Sequence                TCP  Connection finish (FIN)
```


Now that we know this, our first task is to determine what the byte value of the TCP protocol is in the `write-demo.pcapng` file. 

Before we can begin, we need to first navigate to where the files for this room's exercise are stored. The room states that all the files are contained in the `exercise-files` folder located on the desktop. Thus, we need to have our terminal navigate to said folder with `"cd Desktop/exercise-files"`. After that, let's use `ls` to see what's in the folder:

![[tsharkfeatures_task1.png]]

We've got a few files to work with, it seems. Now that we know we have the `write-demo.pcapng` file, let's write out command. We'll want to use the `phs` parameter to show us a nice breakdown of each protocol and their associated byte value:

![[tsharkfeatures_task2q1.png]]

From this file, the recorded byte value of TCP is 62 bytes. Pretty easy.

Our next question asks to now locate this packet in packet lengths row. We know from earlier that a lengths row is associated with showing a tree view of statistics for captured packets. This means that we'll need to combine `tree` and `plen` to have TShark show us rows of packet lengths:

![[tsharkfeatures_task2q2.png]]

Okay, from the resulting rows, we see that packet lengths 40-79 contain our packet in question. This is evident by a) the "count" field for that packet length reports only 1 packet and b) the average, min, and max value of that category is 62.00.

We need to next view the expert info and report the summary of the TCP packet that was captured:

![[tsharkfeatures_task2q3.png]]

Summary of the TCP packet is `Connection establish request (SYN): server port 80`. 👍

For the final question of this module, we need to pivot to the `demo.pcapng` file and report on which IP address is shown in all endpoint conversations. This means we'll use `-z conv,ip -q` to show us the communications between the different IP addresses in the file:

![[tsharkfeatures_task2q4.png]]

There aren't a lot of addresses but, from what we can see, the recurring address in each conversation is `145.254.160.237`.

BUT.

In order to answer the question, we need to do so in a defanged format. Defanging this answer requires that we type it out in a way that's still human-readable but can't be read by a program and automatically converted in a click-able link/address (accidents can happen). We can accomplish this by encasing the periods, separating each octet of the IP address, in brackets `[]`. 

A defanged answer to the question then would be `145[.]254[.]160[.]237`.

---

For this next module, we'll be expanding on use of statistics covered in the previous module. 

Our first section shows us that we can use `-q ptype,tree -q` to show us the IP protocols that are available to view:

```
user@ubuntu$ tshark -r demo.pcapng -z ptype,tree -q
==========================================================================================================================
IPv4 Statistics/IP Protocol Types:
Topic / Item       Count         Average       Min val       Max val Rate (ms)     Percent       Burst rate    Burst start  
--------------------------------------------------------------------------------------------------------------------------
IP Protocol Types  43                                                0.0014        100          0.0400        2.554        
 TCP               41                                                0.0013        95.35        0.0300        0.911        
 UDP               2                                                 0.0001        4.65         0.0100        2.554        
--------------------------------------------------------------------------------------------------------------------------
```


Additionally, we can also use `-z ip_hosts, tree -q` and `z ipv6_host,tree -z` to filter IPV4 and IPV6 address respectfully:

```
user@ubuntu$ tshark -r demo.pcapng -z ip_srcdst,tree -q
==========================================================================================================================
IPv4 Statistics/Source and Destination Addresses:
Topic / Item                     Count         Average       Min val       Max val  Rate (ms)     Percent       Burst rate    Burst start  
--------------------------------------------------------------------------------------------------------------------------
Source IPv4 Addresses            43                                                 0.0014        100          0.0400              
 145.254.160.237                 20                                                 0.0007        46.51        0.0200               
 65.208.228.223                  18                                                 0.0006        41.86        0.0200
...                        
Destination IPv4 Addresses       43                                                 0.0014        100          0.0400             
 145.254.160.237                 23                                                 0.0008        53.49        0.0200             
 65.208.228.223                  16                                                 0.0005        37.21        0.0200
...                          
------------------------------------------------------------------------------------------------------------------------
```


If needed, we can also instruct TShark to filter out results based on the addresses of the source and destinations...

```
user@ubuntu$ tshark -r demo.pcapng -z dests,tree -q
=============================================================================================================================
IPv4 Statistics/Destinations and Ports:
Topic / Item            Count         Average       Min val       Max val       Rate (ms)     Percent       Burst rate    Burst start  
-----------------------------------------------------------------------------------------------------------------------------
Destinations and Ports  43                                                      0.0014        100          0.0400        2.554        
 145.254.160.237        23                                                      0.0008        53.49        0.0200        2.554        
  TCP                   22                                                      0.0007        95.65        0.0200        2.554        
   3372                 18                                                      0.0006        81.82        0.0200        2.554        
   3371                 4                                                       0.0001        18.18        0.0200        3.916        
  UDP                   1                                                       0.0000        4.35         0.0100        2.914        
   3009                 1                                                       0.0000        100.00       0.0100        2.914        
 65.208.228.223         16                                                      0.0005        37.21        0.0200        0.911        
 ...
-----------------------------------------------------------------------------------------------------------------------------
```


...and `-z dns,tree -q` allows us to, as contained within the parameter, view the DNS packets in the capture file:

```
user@ubuntu$ tshark -r demo.pcapng -z dns,tree -q
===========================================================================================================================
DNS:
Topic / Item                   Count         Average       Min val       Max val       Rate (ms)     Percent       Burst rate    Burst start  
---------------------------------------------------------------------------------------------------------------------------
Total Packets                  2                                             0.0055        100          0.0100        2.554        
 rcode                         2                                             0.0055        100.00       0.0100        2.554        
  No error                     2                                             0.0055        100.00       0.0100        2.554        
 opcodes                       2                                             0.0055        100.00       0.0100        2.554        
  Standard query               2                                             0.0055        100.00       0.0100        2.554                  
 ...
-------------------------------------------------------------------------------------------------------------------------
```


The last part of this module explains that we can use http filter parameters to get requests, packets, summarize load distribution, and status info:

+ `-z http,tree -q` - packet and status counter for HTTP
+ `-z http2,tree -q` - same as above but for [HTTP2](https://en.wikipedia.org/wiki/HTTP/2#:~:text=HTTP%2F2%20(originally%20named%20HTTP,protocol%2C%20originally%20developed%20by%20Google.)
+ `-z http_srv,tree -q` - show us the load distribution
+ `-z http_req,tree -q` - get http requests (a little obvious)
+ `-z http_seq,tree -q` - get the requests and responses of http

```
user@ubuntu$ tshark -r demo.pcapng -z http,tree -q
=============================================================================================================================
HTTP/Packet Counter:
Topic / Item            Count         Average       Min val       Max val       Rate (ms)     Percent     Burst rate  Burst start  
----------------------------------------------------------------------------------------------------------------------------
Total HTTP Packets      4                                                       0.0010        100          0.0100     0.911        
 HTTP Response Packets  2                                                       0.0005        50.00        0.0100     3.956        
  2xx: Success          2                                                       0.0005        100.00       0.0100     3.956        
   200 OK               2                                                       0.0005        100.00       0.0100     3.956        
  ???: broken           0                                                       0.0000        0.00         -          -                     
  3xx: Redirection      0                                                       0.0000        0.00         -          -                    
 ...
-----------------------------------------------------------------------------------------------------------------------
```


With this new information, we need pivot back to the `demo.pcapng` file and 1) locate an IP address that appears just 7 times and 2) record that answer in a defanged format.

To locate the address that shows up 7 times, our command needs to be `tshark -r demo.pcapng -z ip_hosts,tree -q`. With this, we are instructing TShark to read the `demo.pcapng` files and show us the IPv4 hosts contained within:

![[tsharkfeatures_task3q1.png]]


Ok, cool. As shown in the column denoted as "Count", the address `216.239.59.99` is recorded 7 times. Which means that our answer, defanged, is `216[.]239[.]59[.]99`.

Now, for the following question, how would we use filters to find out the percentage of destination addresses the previous address is? Earlier we were shown that we can filter packets by the source destination address by using `ip_srcdst,tree` in our command to TShark:

![[tsharkfeatures_task3q2.png]]


Thus, we can see from the results here that the address `216.239.59.99` constitutes 6.98% of the amount of destination addresses in the file.

But what about 2.33%? Well, looking again to the output, it can be seen that `145.253.2.203`, or `145[.]253[.]2[.]203`, makes up for 2.33% of addresses that end up being the destination. 

For the final question of the 3rd module, we need to locate the average "Qname Length" value.  As the `dns,tree` parameter was shown to report query stats, we'll try `tshark -r demo.pcapng -z dns,tree -q`:

![[tsharkfeatures_task3q4.png]]


It's located at the bottom but the report tells us the topic "Qname Len" (or Qname Length) has an average value of 29.00.

---

Up next is using filters to follow streams, export objects, and parse credentials.

For following a stream, we see:

| **Main Parameter** | **Protocol**                        | **View Mode**    | **Stream Number**    | **Additional Parameter** |
| ------------------ | ----------------------------------- | ---------------- | -------------------- | ------------------------ |
| -z follow          | - TCP<br>- UDP<br>- HTTP<br>- HTTP2 | - HEX<br>- ASCII | 0 \| 1 \| 2 \| 3 ... | -q                       |

We are also told that streams start a "0", not "1", so following a stream at its start would need to look something like:

+ `-z follow,tcp,ascii,0 -q` for TCP streams,
+ `-z follow,udp,ascii,0 -q` for UDP streams and,
+ `-z follow,http,ascii,0 -q` for HTTP streams

The following proivded example shows us what an output would look like in response to one of the three filter queries:

```
user@ubuntu$ tshark -r demo.pcapng -z follow,tcp,ascii,1 -q
===================================================================
Follow: tcp,ascii
Filter: tcp.stream eq 1
Node 0: 145.254.160.237:3371
Node 1: 216.239.59.99:80
GET /pagead/ads?client=ca-pub-2309191948673629&random=1084443430285&lmt=1082467020&format=468x60_as&outp...
Host: pagead2.googlesyndication.com
User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.6) Gecko/20040113
...

HTTP/1.1 200 OK
P3P: policyref="http://www.googleadservices.com/pagead/p3p.xml", CP="NOI DEV PSA PSD IVA PVD OTP OUR OTR IND OTC"
Content-Type: text/html; charset=ISO-8859-1
Content-Encoding: gzip
Server: CAFE/1.0
Cache-control: private, x-gzip-ok=""
Content-length: 1272
Date: Thu, 13 May 2004 10:17:14 GMT

...mmU.x..o....E1...X.l.(.AL.f.....dX..KAh....Q....D...'.!...Bw..{.Y/T...<...GY9J....?;.ww...Ywf..... >6..Ye.X..H_@.X.YM.......#:.....D..~O..STrt..,4....H9W..!E.....&.X.=..P9..a...<...-.O.l.-m....h..p7.(O?.a..:..-knhie...
..g.A.x..;.M..6./...{..9....H.W.a.qz...O.....B..
===================================================================
```


We can also extract files from captured packets and requests:

| **Main Parameter** | **Protocol**                                  | **Target Folder**                | **Additional Parameter** |
| ------------------ | --------------------------------------------- | -------------------------------- | ------------------------ |
| --export-objects   | - DICOM<br>- HTTP<br>- IMF<br>- SMB<br>- TFTP | Target folder to save the files. | -q                       |

The example:

```
# Extract the files from HTTP traffic.
user@ubuntu$ tshark -r demo.pcapng --export-objects http,/home/ubuntu/Desktop/extracted-by-tshark -q

# view the target folder content.
user@ubuntu$ ls -l /home/ubuntu/Desktop/extracted-by-tshark/
total 24
-rw-r--r-- 1 ubuntu ubuntu  'ads%3fclient=ca-pub-2309191948673629&random=1084443430285&lmt=1082467020&format=468x60_as&o
-rw-r--r-- 1 ubuntu ubuntu download.html
```

Pretty cool, actually.

Final for this module is the ability to filter and view credentials that have been captured in cleartext. Using `-z credentials -q`, we are shown that our output would look something like:

```
user@ubuntu$ tshark -r credentials.pcap -z credentials -q
===================================================================
Packet     Protocol         Username         Info            
------     --------         --------         --------
72         FTP              admin            Username in packet: 37
80         FTP              admin            Username in packet: 47
83         FTP              admin            Username in packet: 54
118        FTP              admin            Username in packet: 93
123        FTP              admin            Username in packet: 97
167        FTP              administrator    Username in packet: 133
207        FTP              administrator    Username in packet: 170
220        FTP              administrator    Username in packet: 184
230        FTP              administrator    Username in packet: 193
....
===================================================================
```

Always a little yikes catching admin creds in cleartext but also.. kinda cool to see them😎

Ok, time for work. Our first task here is to use `demo.pcapng` to follow "UDP stream 0" and report the defanged "Node 0" value. Since we have to defang our answer, we can discern that the value is an IP address.

To locate this value, we'll need structure our command as `tshark -r demo.pcapng -z follow,udp,ascii,0 -q`:

![[tsharkfeatures_task4q1.png]]

Node 0 is `145.254.160.237:3009` or, defanged, `145[.]254[.]160[.]237:3009`. Cool.

Now we need to do the same thing again except we need follow "HTTP stream 1" to find the "Referer" value:

![[tsharkfeatures_task4q2.png]]

Here, we set the query to specify an http stream with `follow,http,ascii` and start following the stream at position `1`. Since web addresses also need to recorded in a defanged format, our answer for this question is `hxxp[://]www[.]ethereal[.]com/download[.]html`.

So, for credentials: we are asked to determine the total number of credentials that were captured. Now, we could try just using `-z credentials -q`:

![[tsharkfeatures_task4q4.1.png]]![[tsharkfeatures_task4q4.2.png]]

..and count *every* line that appears here to tally the total amount of creds. BUT, as we were taught earlier, there is something we can use to make this much easier: `| nl`

![[tsharkfeatures_task4q4.3.png]]
![[tsharkfeatures_task4q4.4.png]]

It's not perfect, but we can see incrementing numbers for each line! 

To determine the true amount of creds shown in the results, we need to subtract lines 1-3 and 79 since they are not captured creds. This leaves us with `79 - 4 = 75` captured credentials in the `credentials.pcapng` file.

---

When in-depth packet filtering is not quite providing the answers that might be needed, we are shown that we can use advanced operators (also found in Wireshark) such as "**contains**" and "**matches**":

| **Filter**   | **Details**                                                                                                                 |
| ------------ | --------------------------------------------------------------------------------------------------------------------------- |
| **Contains** | - Search a value inside packets.<br>- Case sensitive.<br>- Similar to Wireshark's "find" option.                            |
| **Matches**  | - Search a pattern inside packets.<br>- Supports regex.<br>- Case insensitive.<br>- Complex queries have a margin of error. |
(It's noted that we cannot use these filters with fields containing integer values.) 


To further help investigations, we can extract specific pieces of data for further analysis (rather than needing to analyze entire segments just to find and compare the same information):

| **Main Filter** | **Target Field** | **Show Field Name** |
| --------------- | ---------------- | ------------------- |
| -T fields       | -e <field name>  | -E header=y         |
(The `-e` parameter will need to specified for each new field that we want to extract and then display.)

The provided example of this:

```
user@ubuntu$ tshark -r demo.pcapng -T fields -e ip.src -e ip.dst -E header=y -c 5         
ip.src	ip.dst
145.254.160.237	65.208.228.223
65.208.228.223	145.254.160.237
145.254.160.237	65.208.228.223
145.254.160.237	65.208.228.223
65.208.228.223	145.254.160.237
```


Finally, a little more coverage of "contains" and "matches" are given as well:

| Filter      | contains                                                                                                                                     |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| Type        | Comparison operator                                                                                                                          |
| Description | Search a value inside packets. It is case-sensitive and provides similar functionality to the "Find" option by focusing on a specific field. |
| Example     | Find all "Apache" servers.                                                                                                                   |
| Workflow    | List all HTTP packets where the "server" field contains the "Apache" keyword.                                                                |
| Usage       | `http.server contains "Apache"`                                                                                                              |

```
user@ubuntu$ tshark -r demo.pcapng -Y 'http.server contains "Apache"'                          
   38   4.846969 65.208.228.223 ? 145.254.160.237 HTTP/XML HTTP/1.1 200 OK 

user@ubuntu$ tshark -r demo.pcapng -Y 'http.server contains "Apache"' -T fields -e ip.src -e ip.dst -e http.server -E header=y
ip.src	ip.dst	http.server
65.208.228.223	145.254.160.237	Apache 
```


| Filter      | matches                                                                                                       |
| ----------- | ------------------------------------------------------------------------------------------------------------- |
| Type        | Comparison operator                                                                                           |
| Description | Search a pattern of a regular expression. It is case-insensitive, and complex queries have a margin of error. |
| Example     | Find all .php and .html pages.                                                                                |
| Workflow    | List all HTTP packets where the "request method" field matches the keywords "GET" or "POST".                  |
| Usage       | `http.request.method matches "(GET\|POST)"`                                                                   |

```
user@ubuntu$ tshark -r demo.pcapng -Y 'http.request.method matches "(GET|POST)"'               
    4   0.911310 145.254.160.237 ? 65.208.228.223 HTTP GET /download.html HTTP/1.1 
   18   2.984291 145.254.160.237 ? 216.239.59.99 HTTP GET /pagead/ads?client=ca-pub-2309191948673629&random=1084443430285&

user@ubuntu$ tshark -r demo.pcapng -Y 'http.request.method matches "(GET|POST)"' -T fields -e ip.src -e ip.dst -e http.request.method -E header=y
ip.src	ip.dst	http.request.method
145.254.160.237	65.208.228.223	GET
145.254.160.237	216.239.59.99	GET 
```


Now we need to answer 2 new questions:

With the `demo.pcapng` file, what HTTP packet contains the word "CAFE"?

Since we're looking for an HTTP packet, and are told to report the number of it, we'll need to write out our command to include the use of the "contains" operator that were shown before:

![[tsharkfeatures_task5q1.png]]

From this line, we told TShark to filter out HTTP packets that have the keyword "CAFE" in the server field. This reported that packet # 27, as shown, contains "CAFE".

Similar to the previous question, we next need to filter packets that have "GET" and "POST" requests and then get the first time value that we see in our results. From the last example given to use, we're practically given the exact command to run to grab this info. Thus, armed with this knowledge, our command to TShark will be:

`-Y 'http.request.method matches "(GET|POST)"' -T fields -e frame.time -E header=y`

This will instruct TShark to filter packers with fields that match with GET and POST requests and then report the time of the frame (`frame.time`) as a field in the results:

![[tsharkfeatures_task5q2.png]]

Thus, the answer to our question is `May 13, 2004 10:17:08.222534000 UTC` is the first time value that we found.

---

For this last module, there isn't as much like before but we are given some more example of using extraction filters.

Such as extracting hostname info:

```
user@ubuntu$ tshark -r hostnames.pcapng -T fields -e dhcp.option.hostname | awk NF | sort -r | uniq -c | sort -r
     26 202-ac
     18 92-rkd
     14 93-sts-sec
... 
```

| **Query**                                                      | **Purpose**                                                                         |
| -------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| `tshark -r hostnames.pcapng -T fields -e dhcp.option.hostname` | Main query.  <br>Extract the DHCP hostname value.                                   |
| `awk NF`                                                       | Remove empty lines.                                                                 |
| `sort -r`                                                      | Sort recursively before handling the values.                                        |
| `uniq -c`                                                      | Show unique values, but calculate and show the number of occurrences.               |
| `sort -r`                                                      | The final sort process.  <br>Show the output/results from high occurrences to less. |

Extracting DNS queries:

```
user@ubuntu$ tshark -r dns-queries.pcap -T fields -e dns.qry.name | awk NF | sort -r | uniq -c | sort -r
     96 connectivity-check.ubuntu.com.rhodes.edu
     94 connectivity-check.ubuntu.com
      8 3.57.20.10.in-addr.arpa
      4 e.9.d.b.c.9.d.7.1.b.0.f.a.2.0.2.0.0.0.0.0.0.0.0.0.0.0.0.0.8.e.f.ip6.arpa
      4 0.f.2.5.6.b.e.f.f.f.b.7.2.4.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.8.e.f.ip6.arpa
      2 _ipps._tcp.local,_ipp._tcp.local
      2 84.170.224.35.in-addr.arpa
      2 22.2.10.10.in-addr.arpa
```

and getting User Agents (web browsers and the like):

```shell-session
user@ubuntu$ tshark -r user-agents.pcap -T fields -e http.user_agent | awk NF | sort -r | uniq -c | sort -r
      6 Mozilla/5.0 (Windows; U; Windows NT 6.4; en-US) AppleWebKit/534.10 (KHTML, like Gecko) Chrome/8.0.552.237 Safari/534.10
      5 Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:100.0) Gecko/20100101 Firefox/100.0
      5 Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.32 Safari/537.36
      4 sqlmap/1.4#stable (http://sqlmap.org)
      3 Wfuzz/2.7
      3 Mozilla/5.0 (compatible; Nmap Scripting Engine; https://nmap.org/book/nse.html)
```

All good examples of gleaming important stuff from a myriad of captured data.


Time to try it out ourselves. Using the `hostnames.pcapng` that we have located in `excercise-files`, what is the number of unique hostnames?

For this one, we need to not only tell TShark to grab the hostnames but we need to tell it to omit recurring hostnames and only report the amount of hostnames *unique* to themselves:

![[tsharkfeatures_task6q1&2.png]]

In the b first half before the first break (`|`), we're having TShark read the `hostnames.pcapng` file and report only the DHCP hostnames from the filtered packets. Then, after that initial break, we're instructing it to a) remove blank lines, b) sort recursively, c) show only unique values and show the number of occurrences of each, d) sort recursively again (sorting the unique results), and e) provide a numbered list with the results. TShark having processed our command, we see that there 30 unique hostnames in `hostnames.pcapng`.

We can also use this output to solve the amount of times the hostname "prus-pc" makes an appearance. "prus-pc" is located as the 4th entry on the list and has a corresponding value of 12 appearances. Nice.

We need now to move over to `dns-queries.pcap`. Again, the process for arriving at the answer for this question is somewhat similar to that of the previous: we need to have TShark filter and report data from the field `dns.query.name` and then sort the info much like we sorted the hostnames.

![[tsharkfeatures_task6q3.png]]

What is the number of the most common DNS query? Simple: number 1, `db.rhodes.edu`, is the most common DNS query captured with 472 appearances.

From the `user-agents.pcap`, we are told to find the number of "Wfuzz user agents". Since we are looking for HTTP user agents, we'll need something like this:

![[tsharkfeatures_task6q4.png]]

Before we can answer, we see that entries 1 and 6 BOTH contain Wfuzz user agents. As we need to report the *total* amount of these agents, simply doing `9 + 3` gives us 12 times the Wfuzz user agent appears.

The last question of this room needs us to locate the HTTP hostname for the captured nmap scans and then report it in defanged format. Here's what I did:

![[tsharkfeatures_task6q5.png]]

From the first command, I ran TShark on the `user-agents.pcap` file and told it to filter and report the HTTP user agents (and sort them so it's easier to read). However, while this gives me a good view of the captured user agents, it does *not* show me a hostname or IP address associated with the agent.

This means that I needed to specifiy another field, as shown in the second command: `http.host`. Doing so not only has TShark show my the user agents (like before) but now it will also show me the *hostnames* associated with each agent. With this shown (on both results, really), we see that there's only 1 user agent that mentions Nmap scanning and, from the second result, it belongs to `172[.]16[.]172[.]129`.

---

Like the previous room I did before it, this we fun and educational to learn more about using TShark to filter packets and extract data from said packets. Turned out to be a little more interesting to as it was diving into some slightly more technical uses of TShark than the initial room on the basics.