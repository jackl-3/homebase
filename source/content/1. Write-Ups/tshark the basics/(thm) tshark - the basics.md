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

After we move into the correct folder with`cd /Desktop/exercise-files`, we can then run `capinfos demo.pcapng` to give us the following info:

![[task1q2.png]]

From this we can see that the value of `RIPEMD160` is `6ef5f0c165a1db4a3cad3116b0c5bcc0cf6b9ab7`. Woo!

---

