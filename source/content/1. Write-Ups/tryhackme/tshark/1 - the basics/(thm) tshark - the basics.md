[This room, provided by TryHackMe,](https://tryhackme.com/room/tsharkthebasics) serves a brief, ground-level intro into using TShark. Created as an open source, CLI-only variant of a popular network traffic analyzer, [Wireshark](https://en.wikipedia.org/wiki/Wireshark), it states that it offers almost the same set of features as Wireshark while also offering the ability to be used like [tcpdump](https://en.wikipedia.org/wiki/Tcpdump) (another tool for analyzing network traffic).

Since interaction with the tool is restricted to the command line, any results will need to be obtained by manually typing commands in a terminal (as opposed to Wireshark's GUI navigation most of the time).

Beginning with the first module, we have a small chart listing the most common tools used in TShark (pasted from room):

| **Tool/Utility** | **Purpose and Benefit**                                                                                                                                |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **capinfos**     | A program that provides details of a specified capture file. It is suggested to view the summary of the capture file before starting an investigation. |
| **grep**         | Helps search plain-text data.                                                                                                                          |
| **cut**          | Helps cut parts of lines from a specified data source.                                                                                                 |
| **uniq**         | Filters repeated lines/values.                                                                                                                         |
| **nl**           | Views the number of shown lines.                                                                                                                       |
| **sed**          | A stream editor.                                                                                                                                       |
| **awk**          | Scripting language that helps pattern search and processing.                                                                                           |

Pretty neat.

The first series of questions is pretty simple: navigate to the folder with the files the room will be using for this room and paste the "RIPEMD160" value from `demo.pcapng`:

After we move into the correct folder with `cd Desktop/exercise-files`, we can then run `capinfos demo.pcapng` to give us the following info:

![[tsharkbasics_task1q2.png]]

From this we can see that the value of `RIPEMD160` is `6ef5f0c165a1db4a3cad3116b0c5bcc0cf6b9ab7`. Woo!

---

In the next module, we're given a list of parameters and their example use-cases:

| **Parameter**    | **Purpose**                                                                                |
| ---------------- | ------------------------------------------------------------------------------------------ |
| -h               | Display the help page with the most common features.<br><br>- `tshark -h`                  |
| -v               | Show version info.<br><br>- `tshark -v`                                                    |
| -D               | List available sniffing interfaces.<br><br>- `tshark -D`                                   |
| -i               | Choose an interface to capture live traffic.<br><br>- `tshark -i 1`<br>- `tshark -i ens55` |
| **No Parameter** | Sniff the traffic like tcpdump.<br><br>- `tshark`                                          |

Again, pretty simple stuff; we see here that we can view the version of TShark, get the help page (handy!), see and choose what interfaces (hardware and software) that we can use, or simply use the program like tcpdump.

Knowing this, we are asked to a) report the version of TShark that we're using and b) list the number of interfaces we can get traffic from:

![[tsharkfeatures_task2q1.png]]

As we can see, we are using TShark 3.2.3. Now, how many interfaces are we able to possibly use?

![[tsharkbasics_task2q2.png]]

12 total. That's a lot of interfaces.

---

The next module is an extension of the previous one:

| **Parameter** | **Purpose**                                                                                                                                                       |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| -r            | Read/input function. Read a capture file.<br><br>- `tshark -r demo.pcapng`                                                                                        |
| -c            | Packet count. Stop after capturing a specified number of packets.<br>E.g. stop after capturing/filtering/reading 10 packets.<br><br>- `tshark -c 10`              |
| -w            | Write/output function. Write the sniffed traffic to a file.<br><br>- `tshark -w sample-capture.pcap`                                                              |
| -V            | Verbose.<br>Provide detailed information **for each packet**. This option will provide details similar to Wireshark's "Packet Details Pane".<br><br>- `tshark -V` |
| -q            | Silent mode.<br>Suspress the packet outputs on the terminal.<br><br>- `tshark -q`                                                                                 |
| -x            | Display packet bytes.<br>Show packet details in hex and ASCII dump for each packet.<br><br>- `tshark -x`                                                          |

Now we're getting a little more in depth. We can see that we have options to read a capture file and, write functions, and view results in more detail.

One particular image I found informative was the comparison between TShark and Wireshark in with the same outputs:

![[tsharkbasics_tsharkvswireshark.png]]

Depends on your personal preference but I do appreciate the simplicity of having all the info laid out instead of contained within sections that can toggle visibility of categorized info. Can visually be a lot but it also reduces time on navigation in that manner.

For this one, we first need to read the `demo.pcapng` file to find what TCP flags are shown with packet 29. Utilizing the command `tshark -r demo.pcapng`, we get a long list of packets including:

![[tsharkbasics_task3q1.png]]

Since we know that common TCP flags are either `SYN`, `ACK`, `RST`, `PSH`, `URG` and `FIN`, we can clearly see that the 29th packet contains both the `PSH` and `ACK` flags.

Now, to answer the module's next question: what is the "Ack" value of the 25th packet?

![[tsharkbasics_task3q2.png]]

Following past the `ACK` flag, we see can see that the value of "Ack" in this packet is `12421`.

Lastly, we are asked to locate the "Windows size value" in the 9th packet:

![[tsharkbasics_task3q3.png]]

As before, right after our "Ack" value do we see that the "Windows value size", or "Win" as it appears, is equal to `9660`.

---

This next part deals with TShark's ability to capture packets and either stop at a specific point or continue running in a loop.

| **Parameter** | **Purpose**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
|               | Define capture conditions for a single run/loop. **STOP** after completing the condition. Also known as "Autostop".                                                                                                                                                                                                                                                                                                                                                                                    |
| -a            | **Duration:** Sniff the traffic and stop after X seconds. Create a new file and write output to it.  <br>    <br>- `tshark -w test.pcap -a duration:1`<br><br>**Filesize:** Define the maximum capture file size. Stop after reaching X file size (KB).<br><br>- `tshark -w test.pcap -a filesize:10`<br><br>**Files:** Define the maximum number of output files. Stop after X files.<br><br>- `tshark -w test.pcap -a filesize:10 -a files:3`                                                        |
|               | **Ring buffer control options**. Define capture conditions for multiple runs/loops. (INFINITE LOOP).                                                                                                                                                                                                                                                                                                                                                                                                   |
| -b            | **Duration:** Sniff the traffic for X seconds, create a new file and write output to it.   <br><br>- `tshark -w test.pcap -b duration:1`<br><br>**Filesize:** Define the maximum capture file size. Create a new file and write output to it after reaching filesize X (KB).<br><br>- `tshark -w test.pcap -b filesize:10`<br><br>**Files:** Define the maximum number of output files. Rewrite the first/oldest file after creating X files.<br><br>- `tshark -w test.pcap -b filesize:10 -b files:3` |

It should be noted that these capture parameters cannot be ran on `pcap` files since they're intended to employed during live capture (doing so results in error).

---

A quick note from the next section states that there are two fields of packet filtering within TShark: live (capture) and post-capture (display).  A quick recap from the Wireshark room on Packet Operations tells us:

| Dimension           | Description                                                                                                                                                           |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Capture Filters** | Live filtering options. The purpose is to **save** only a specific part of the traffic. It is set before capturing traffic and is not changeable during live capture. |
| **Display Filters** | Post-capture filtering options. The purpose is to investigate packets by **reducing the number of visible packets**, which is changeable during the investigation.    |

These filters come in handy when one needs to only capture and view specific packets rather than capturing everything and having to sort through to find the ones needed for analysis.

| **Parameter** | **Purpose**                                                          |
| ------------- | -------------------------------------------------------------------- |
| -f            | Capture filters. Same as BPF syntax and Wireshark's capture filters. |
| -Y            | Display filters. Same as **Wireshark's display filters.**            |

---

Now that capture filters have been covered, it's time to put them into action. Syntax for basic filtering is as follows:

| **Qualifier** | **Details and Available Options**                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Type**      | Target match type. You can filter IP addresses, hostnames, IP ranges, and port numbers. Note that if you don't set a qualifier, the "host" qualifier will be used by default.<br><br>- host \| net \| port \| portrange<br><br>Filtering a host<br><br>- `tshark -f "host 10.10.10.10"`<br><br>Filtering a network range <br><br>- `tshark -f "net 10.10.10.0/24"`<br><br>Filtering a Port<br><br>- `tshark -f "port 80"`<br><br>Filtering a port range<br><br>- `tshark -f "portrange 80-100"` |
| **Direction** | Target direction/flow. Note that if you don't use the direction operator, it will be equal to "either" and cover both directions.<br><br>- src \| dst<br><br>Filtering source address<br><br>- `tshark -f "src host 10.10.10.10"`<br><br>Filtering destination address<br><br>- `tshark -f "dst host 10.10.10.10"`                                                                                                                                                                              |
| **Protocol**  | Target protocol.<br><br>- arp \| ether \| icmp \| ip \| ip6 \| tcp \| udp<br><br>Filtering TCP<br><br>- `tshark -f "tcp"`<br><br>Filtering MAC address<br><br>- `tshark -f "ether host F8:DB:C5:A2:5D:81"`<br><br>You can also filter protocols with IP Protocol numbers assigned by IANA.<br>Filtering IP Protocols 1 (ICMP)<br><br>- `tshark -f "ip proto 1"`<br>- [**Assigned Internet Protocol Numbers**](https://www.iana.org/assignments/protocol-numbers/protocol-numbers.xhtml)         |

Some practice examples from TryHackMe:

| **Capture Filter Category** | **Details**                                                                                                                                                                                                                                                                                                                                                                      |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Host Filtering**          | Capturing traffic to or from a specific host.<br><br>Traffic generation with cURL. This command sends a default HTTP query to a specified address.<br><br>- `curl tryhackme.com`<br><br>TShark capture filter for a host<br><br>- `tshark -f "host tryhackme.com"`                                                                                                               |
| **IP Filtering**            | Capturing traffic to or from a specific port. We will use the Netcat tool to create noise on specific ports.<br><br>Traffic generation with Netcat. Here Netcat is instructed to provide details (verbosity), and timeout is set to 5 seconds.<br><br>- `nc 10.10.10.10 4444 -vw 5`<br><br>TShark capture filter for specific IP address<br><br>- `tshark -f "host 10.10.10.10"` |
| **Port Filtering**          | Capturing traffic to or from a specific port. We will use the Netcat tool to create noise on specific ports.<br><br>Traffic generation with Netcat. Here Netcat is instructed to provide details (verbosity), and timeout is set to 5 seconds.<br><br>- `nc 10.10.10.10 4444 -vw 5`<br><br>TShark capture filter for port 4444<br><br>- `tshark -f "port 4444"`                  |
| **Protocol Filtering**      | Capturing traffic to or from a specific protocol. We will use the Netcat tool to create noise on specific ports.<br><br>Traffic generation with Netcat. Here Netcat is instructed to use UDP, provide details (verbosity), and timeout is set to 5 seconds.<br><br>- `nc -u 10.10.10.10 4444 -vw 5`<br><br>TShark capture filter for<br><br>- `tshark -f "udp"`                  |

With these good examples before us, let's see what we can find out using the commands provided above in our VM. 

The first question for this module asks us to determine the number of packets with `SYN` bytes. From reading the material prior, we can do this with the command `tshark -f "host 10.10.10.10`:

![[tsharkbasics_task7q1.png]]

From the 14 packets that were captured, we can see that packets 3 and 4 both contain the `SYN` flag.

Now, what about the number of packets that were sent to the `10.10.10.10` address?

By modifying our previous command slight to use the "dst host" instead of "host", we can see the number of packets sent directly to the host:

![[tsharkbasics_task7q2.png]]

It appears that 7 packets were sent directly to that host `10.10.10.10`.

The last question for this module needs us to revisit the results for the 1st question and report the amount of packets that have `ACK` bytes. 

![[tsharkbasics_task7q1.png]]

Seems like we have a total of 8 packets with `ACK` bytes. Nice.

---

The final module for this room covers the other half of packet filtering: display filters. The ones to note for this room are as follows:

| **Display Filter Category** | **Details and Available Options**                                                                                                                                                                                                                                                                                                      |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Protocol: IP**            | Filtering an IP without specifying a direction<br><br>- `tshark -Y 'ip.addr == 10.10.10.10'`<br><br>Filtering a network range<br><br>- `tshark -Y 'ip.addr == 10.10.10.0/24'`<br><br>Filtering a source IP<br><br>- `tshark -Y 'ip.src == 10.10.10.10'`<br><br>Filtering a destination IP<br><br>- `tshark -Y 'ip.dst == 10.10.10.10'` |
| **Protocol: TCP**           | Filtering TCP port<br><br>- `tshark -Y 'tcp.port == 80'`<br><br>Filtering source TCP port<br><br>- `tshark -Y 'tcp.srcport == 80'`                                                                                                                                                                                                     |
| **Protocol: HTTP**          | Filtering HTTP packets<br>    <br>- `tshark -Y 'http'`<br><br>Filtering HTTP packets with response code "200"<br><br>- `tshark -Y "http.response.code == 200"`                                                                                                                                                                         |
| **Protocol: DNS**           | Filtering DNS packets<br>    <br>- `tshark -Y 'dns'`<br><br>Filtering all DNS "A" packets<br><br>- `tshark -Y 'dns.qry.type == 1'`                                                                                                                                                                                                     |

With this in hand, let's see what else we cant find out from `demo.pcapng` file.

Question 1 is requesting that we locate the number of packets that have the IP address of `65.208.228.223`. 

We can accomplish this by building our command as `tshark -r demo.pcapng -Y "ip.addr == 65.208.228.223" | nl`. This is telling TShark to read the `demo.pcapng` file and show only the packets that have the IP address we need:

![[tsharkbasics_task8q1.png]]
![[tsharkbasics_task8q1.2.png]]

From our output, the first column on the left of the results shows us the number of packets our filter returned while the second column on the right of it shows the number of packets assigned to each entry. That said, number of packets with the IP address `65.208.228.223` is 34. 👌

Next thing is to determine the number packets that have TCP port 3371. Again, similar to the answer before, our command for this answer will be `tshark -r demo.pcapng -Y "tcp.port == 3371" | nl` with the results being:

![[tsharkbasics_task8q2.png]]

Cool. There are 7 packets that contain TCP port 3371.

Next question needs to know the total number of packets that have the IP address `145.254.160.237` as the source address. For this one, our command should be `tshark -r demo.pcapng -Y 'ip.src == 145.254.160.237' | nl`.  From that, we get:

![[tsharkbasics_task8q3.png]]

Just like with the first question of this module, the incrementing column of numbers on the left in the results tell us how many packets have the IP address `145.254.160.237`. This means that there are 20 packets with that IP as the source address. 

However, our last question needs us to locate the amount of packets contain a "Duplicate" packet. Thankfully, it's pretty simple: packet 17 has 37 packets that contain the  "Duplicate" packet. This much is evident by region where the TCP flag(s) are reported in packet 17: `[TCP Dup ACK 28#1]`.

---

This was a pretty cool room. I've covered some basic usage with Wireshark on TryHackMe and found this to a good way of brushing up on some of the basics for Wireshark while also getting to know/try something a little different. 👍