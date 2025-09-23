
For some time now, as I've been browsing my YouTube looking for my next project (or even maybe another hobby to invest into?), I started seeing video recommendations of people taking old iPod Classics (mostly the [5th generation as they're the easiest to get into](https://en.wikipedia.org/wiki/IPod_Classic#/media/File:Ipod_5th_Generation_white.jpg)) and modifying them by giving it a new, higher-capacity battery, new front shell, replacing the old miniature disk drive with an SD card, and even going as far as adding Bluetooth! 

Now, I enjoy listening to music leisurely. That is, to say, that I'm not *always* listening to music but usually whenever it feels like a good time to do so. Over the years, I've also curated my own personal library with music that I've hand selected over the years (Synth-wave tends be what I've gravitated to mostly, now that I think about but I also enjoy a variety of other music as well). As such, this somewhat reinforces my enjoyment of listening to music, knowing that I've got my own personal library of songs and tracks that I know I'll enjoy.

Going back to the iPod project: after watching probably almost a dozen (if not more) videos ranging from people demonstrating or showing them off to instructional videos on how to perform different mods on the iPod. One of the biggest selling points, for me, was that it could  run [Rockbox](https://www.rockbox.org/), community-made custom software for old media/music players. I have an old SanDisk MP3 player that mother got for me waaaaay back before I even had a smartphone and I recall that, at the time, the Rockbox community had working support for that old media player. At the time, it couldn't do much in retrospect, but it absolutely appealed to my desire to make my stuff my own by applying my flair to it and I loved having an MP3 player with custom firmware (nerdy, I know).

So, after mulling on it for a little bit, I decided that I would set out to build one myself.

---

Of course, like any project, I needed to know what my options were. I looked around and decided that:

1. I don't really need Bluetooth. As cool as it would be to have a wireless old iPod, I don't anticipate I'd daily-drive the iPod that much to warrant Bluetooth (there's also Bluetooth adapters that plug into the 30-pin connection at the bottom or even 3.5mm adapters as well, should I decide to go that route)

2. [3000mAh battery (specifically designed to work with the iPod's power supply connection)](https://www.aliexpress.us/item/3256808014514928.html?spm=a2g0o.order_list.order_list_main.40.15b318025D24N3&gatewayAdapt=glo2usa) This, to me, seemed like the most bang-for-my-buck compared to getting a 2000mAh or less battery. Since I intend to run more complicated audio formats like FLAC, for example, I know that having a lot of juice to pull from meant longer up-time.

3. [A clear green front shell/face plate](https://www.aliexpress.com/item/3256809158080463.html?spm=a2g0o.order_list.order_list_main.47.15b318025D24N3). On the used iPod that I later bought from eBay, I could tell immediately from the photos that the front of the device was pretty scratched up. That and a clear green front shell would look reaaaaally cool (I had seem some on r/ipodclassic and I was sold 😁)

4. [The iFlash quad microSD adapter](https://www.iflash.xyz/store/iflash-quad/). Now this one was interesting as I only intend to be using one SD card of 256G B in the device as my storage (my current music library doesn't even exceed 30 GB) but decided to go with it because I had read and heard from YouTube videos that it was most compatible with a 3000mAh adapter. It also leaves me with the option to upgrade storage in the future, should the need arise.

5. [256 GB MicroSD card](https://www.amazon.com/dp/B0DRG4J8CK?ref=ppx_yo2ov_dt_b_fed_asin_title&th=1). Self explanatory and I made sure to pick up one that was confirmed compatible with Rockbox and the adapter as reported by the iFlash community.

For visual reference, here are the parts I purchased to use with my iPod:

![[ipodparts.png]]
![[iflashquad.png]]

While I waited for the parts to arrive, I spent some time using the Rockbox software on the iPod to get a feel of its functionality and learn a little bit of navigation and what kind of settings I can tweak. Aside from the new ability to play higher quality audio formats (like FLAC and Opus), I was also very happy to see many options for themes and a generally great amount of settings that can be adjusted to get things configured to my liking. All that was left now was to wait for the new stuff arrive.

---

As with all things shipping, my parts came in pretty much one after the other. I first got the green faceplate, followed by the SD card adapter, then the SD card itself, with the new battery being the final component (for some reason it took longer that I expected to clear customs).

The overall process, in retrospect, was pretty simple and not really all that difficult. However, the time dissembling, cleaning, reassembling, and getting the CFW set up again (since it now has a new drive) did take me some time. I should note that, while I was going through the process, I was making sure to pay attention at certain times during [a YouTube video on disassembly and reassembly](https://www.youtube.com/watch?v=l8b6X6pN5-Q) of the iPod from a channel that's dedicated to repairing and modifying old iPods as a service. There are other videos on the author's main channel that would serve as better introduction point but I had already seem them so I just needed some footage of taking apart and putting back together the iPod.

Once the Rockbox firmware was installed and the device was closed up, it was time to transfer music over. Now, Rockbox is a little tricky in how it handles music or, more specifically in my case, album/track art. Rockbox is designed to work with a decent amount of old portable media players and these old players don't have lot of power to read and render album/track art from high-quality audio track like my Android phone can. As such, I really had two options for this:

1. Convert all my music to MP3 and convert the track into non-progressive JPEGs and embed them back into each audio file, or
2. Extract art from each track, convert them into non-progressive JPEGs, and give them the same filename as the source file so Rockbox just reads the track art separately.

I'm not sure why I didn't think of the other option first but I decided to, with the help of Grok, create a bash script to use ffmpeg and ImageMagick to convert the tracks, grab the track art, and re-embed the art in the newly converted track in a manner that will enable Rockbox to read said art. It worked, but I didn't really care for the fact that a bunch of my lossless audio had to be converted to a lower-quality format. 

So then I went with option 2: I once again, assisted by Grok, created a bash script that will use ffmpeg and ImageMagick again to extract and convert the art but, this time, instead of converting the audio file and embedding the converted art into it, I decided to just have the newly made JPEG have the same exact filename as the file it was pulled from. This means that Rockbox doesn't have to try and find embedded art and can instead just detect an image file with the same name as the track's filename and then just display that as album/track art after loading the track. 

Trust me when I say that going through this particular process just to get track art to show on the screen when playing audio took far longer than actually modding the iPod itself. BUT.. it was worth it and I really enjoyed working with Grok to create a script in bash to automate the process (and, in turn, save me a HUGE amount of time!)

With that all said and done, I had completed what I had set out to do and now I have my own iPod Classic capable of playing all of my music no matter the format! I had a ton of fun working on this and, during my course of interacting and taking apart the iPod, I gained a sense of appreciation for the engineering that went into its design. I can see now why it was such a hit for so long before it was replaced eventually by the iPhone (or, really, modern smartphones today).

![[completedipodclassic.jpg]]
