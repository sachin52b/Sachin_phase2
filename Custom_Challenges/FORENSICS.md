# 1.Hide and seek

Sakamoto’s at it again with a game of hide and seek. This time the challenge hinted that some secret data was hidden inside an image, and the hint clearly nudged towards stegseek. So from the start, I was thinking this wasn’t about guessing blindly but about using the right steganography tool.

## Solution:
I began by locating the downloaded challenge folder. Since I was using WSL, the image was actually inside my Windows Downloads directory. So the first thing I did was move into that folder from Linux, making sure to handle the spaces in the directory name properly.
```
cd "/mnt/c/Users/Admin/Downloads/Hide and Seek-20251222T143957Z-3-001/Hide and Seek"
```

Once I was inside the folder, I checked that the image file was present. The hint mentioned hide and seek and explicitly referenced stegseek, which made it clear that the image likely contained steghide data protected by a password.

I already had stegseek installed and had manually downloaded the rockyou.txt wordlist since Ubuntu does not include it by default like Kali does. With everything ready, I ran stegseek against the image using the rockyou wordlist.
```
stegseek sakamoto.jpg /usr/share/wordlists/rockyou.txt
```

After running for a short time, stegseek successfully found the password. It printed out that the passphrase was iloveyou1 and also mentioned that the original hidden file was named flag.txt. Stegseek then extracted the hidden data automatically into a new file.

To confirm, I listed the directory contents and saw a new file called sakamoto.jpg.out. This told me the extraction had worked.
```
ls
```

Finally, I opened the extracted file to see what was inside.
```
cat sakamoto.jpg.out
```

This revealed the flag, confirming that the hidden data inside the image had been successfully recovered.

## Flag:

```
nite{h1d3_4nd_s33k_but_w1th_st3g_sdfu9s8}
```

## Concepts learnt:
The main thing I learned from this challenge was how steganography is often combined with weak passwords to hide data inside images. Even if an image looks completely normal, tools like steghide can embed files inside it without visible changes.

I also learned how powerful stegseek is for CTFs. Instead of manually trying passwords or guessing, it automates steghide detection and brute forces the password using a wordlist, which makes solving these challenges much faster and more realistic.

Another important concept reinforced here was navigating Windows files from WSL, especially understanding how Windows drives are mounted under /mnt and how to deal with spaces in directory names.

## Incorrect Tangents

1.Initially, I tried to think if the flag might be hidden using basic techniques like strings or metadata inspection, but those approaches didn’t give anything useful. The hint pointing to stegseek made it clear that a password based steghide attack was the right direction.

2.I also assumed at first that rockyou.txt would already be available on my system. After running into missing file errors, I realized that on Ubuntu and WSL setups, common wordlists need to be downloaded manually.

## Resources:

https://github.com/RickdeJager/StegSeek

https://ctf101.org/steganography/overview/


***

# 2. Nutrela Chunks


One of my favorite foods is soya chunks. But as I was enjoying some Nutrela today, I noticed a few chunks weren’t quite right. Seems like something’s off with their structure. Could you help me fix these broken chunks so I can enjoy my meal again?


## Solution:
After reading the challenge description, I realized that I needed to edit the hex values of a .png file.

At first, I was confused about what exactly needed to be changed—whether it was the image width/height, the header, or something else. When I tried opening the file, I noticed that it was corrupted. This led me to research how PNG files are structured.

While reading about PNG internals, I learned that PNG files are made up of chunks, and that three of the most important ones are:

IHDR

IDAT

IEND

I also discovered a tool called pngcheck, which can be used to verify PNG file integrity. I installed it and ran it on nutrela.png:

```
 pngcheck nutrela.png
zlib warning:  different version (expected 1.2.13, using 1.3)

nutrela.png  this is neither a PNG or JNG image nor a MNG stream
ERROR: nutrela.png
```

This error indicated that the PNG header itself was incorrect. I fixed it by modifying the first 4 bytes:

```
Original bytes: 89 70 6E 67

Correct bytes: 89 50 4E 47
```

After making this change, I ran pngcheck again:
```
 pngcheck nutrela.png
nutrela.png  first chunk must be IHDR
ERROR: nutrela.png
```

This showed that the IHDR chunk was invalid. I fixed it by correcting the chunk name:
```
Original: 69 68 64 72

Correct: 49 48 44 52
```
Running pngcheck again produced another error:
```
nutrela.png  illegal reserved-bit-set chunk idat
ERROR: nutrela.png
```

I fixed the IDAT chunk in the same way:
```
Original: 69 64 61 74

Correct: 49 44 41 54
```
After that, I received a similar error for the IEND chunk:
```
nutrela.png  illegal reserved-bit-set chunk iend
ERROR: nutrela.png
```

So I corrected it as well:
```
Original: 69 65 6E 64

Correct: 49 45 4E 44
```
Finally, after running pngcheck one last time:
```
pngcheck nutrela.png
OK: nutrela.png (1000x1000, 24-bit RGB, non-interlaced, 82.0%).
```

The PNG file was now valid. When I opened the image, Martin Scorsese and the flag was visible.


## Flag:

```
nite{n0w_y0u_kn0w_ab0ut_PNG_chunk5}
```

## Concepts learnt:
1.PNG files are strictly structured and chunk names are case-sensitive (IHDR, IDAT, IEND must be uppercase).

2.pngcheck is useful for step-by-step debugging of corrupted PNG files.


## Incorrect Tangents

1.Initially suspected wrong width/height values instead of chunk corruption.

2.Assumed the flag was hidden using steganography, but it was visible after fixing the file structure.

## Resources:

https://www.w3.org/TR/PNG/

http://www.libpng.org/pub/png/apps/pngcheck.html

***

# 3. RAR of the Abyss

Two philosophers peer into the networked abyss and swap a secret.
Use the secret to decrypt the Abyss’ RAwR and pull your flag from the void.

## Solution:
I opened the provided PCAP file in Wireshark. Since the challenge mentioned a conversation and a secret being exchanged, I expected the important information to be present in packets that actually carried data, rather than in empty control packets.

To focus only on packets with useful content, I applied the display filter:

tcp.len > 0


This filter hides TCP packets that do not contain any payload, such as pure acknowledgements, and shows only packets where data is being transmitted. As a result, unnecessary traffic was removed, and the packets containing meaningful communication became easier to identify.

After filtering, I followed the TCP stream:

Right-click → Follow → TCP Stream


The full conversation and binary data were constructed from multiple packets across the stream. Reassembling the stream revealed readable ASCII messages resembling a philosophical dialogue:

Camus: One must imagine Sisyphus happy but are we happy ?

Nietzsche: You will be happy after reading my latest work


Immediately after this dialogue, the stream contained a large block of non-ASCII binary data, starting with:

Rar!


The Rar! string is the magic header of a RAR archive, indicating that a compressed file was transmitted across the TCP stream. The data for this archive was spread across multiple packets, not just a single one, requiring full stream reconstruction.

Later in the conversation, the password was shared:

Camus: whats the password ?

Nietzsche: b3y0ndG00dand3vil


This confirmed that the binary blob was a password-protected RAR file, and the password was explicitly given in the stream.

To extract the archive:

In Follow TCP Stream, I set:

Show data as → Raw


This ensured all binary bytes from multiple packets were preserved correctly.

I clicked Save As and saved the stream as:

abyss.rar


RAR tools safely ignore extra data before the archive header, so saving the full stream from multiple packets was fine.

I opened abyss.rar in WinRAR and entered the password:

b3y0ndG00dand3vil


WinRAR successfully decrypted and extracted the archive contents.

Finally, inside the extracted files, I found the flag.

## Flag:

```
nite{thus_sp0k3_th3_n3tw0rk_f0r3ns1cs_4n4lyst}
```

## Concepts learnt:
1.How to filter TCP packets by payload (tcp.len > 0) to focus on meaningful data.

2.How to reconstruct a TCP stream and identify embedded files (like RAR) from network traffic.

## Incorrect Tangents
1.Initially checking all packets without filtering, which made it hard to spot the actual data.

2.Thinking the conversation itself contained the flag instead of realizing the binary payload after the dialogue was the target.

## Resources:

https://www.wireshark.org/docs/dfref/t/tcp.html
 
Looks like I got a little too clever and hid the flag as a password in Firefox, tucked away like one of NineTails’ many tails. Recover the "logins" and the "key4" and let it guide you to the flag.

***

# 4. NineTails

Looks like I got a little too clever and hid the flag as a password in Firefox, tucked away like one of NineTails’ many tails. Recover the "logins" and the "key4" and let it guide you to the flag.


## Solution:
I started with the AD1 image. I Opened it in FTK Imager . Once mounted, I could see the file system and there it was, hiding a RAR file somewhere in the user profile folders.

Extracting the RAR revealed a Firefox profile directory. Inside, the files that mattered were obvious: logins.json and key4.db. These are Firefox’s way of keeping secrets. The challenge hinted at a master password, and the profile name itself was the clue: j4gjesg4.

I knew now that the next step was to decrypt the saved passwords. That’s where firefox_decrypt comes in. It takes logins.json and key4.db, unlocks the encrypted credentials using the master password, and prints them out. Running it with the profile path and the master password:
```
python3 firefox_decrypt.py /mnt/c/Users/Admin/OneDrive/Desktop/j4gjesg4.default-release
```

was almost anticlimactic at first, because it warned about profile.ini missing but continued anyway. Then the list of saved logins appeared. Most were decoys like 'CHEEEEE..' . But picking out the meaningful parts, the flag was split across entries. One password had:

GCTF{m0zarella


and another had:

p4ssw0rd}

The Facebook login that said SIKE was clearly a troll. Ignoring the decoys, I reconstructed the flag by combining the relevant fragments. At first I questioned if _f1ref0x_ should go in the middle but at last it kind of made sense. 

So after  ignoring the decoys, and trusting the logical reconstruction, the flag emerged.

## Flag:

```
GCTF{m0zarella_f1ref0x_p4ssw0rd}
```

## Concepts learnt:
1.How Firefox stores passwords: logins.json + key4.db and the role of a master password.

2.Using firefox_decrypt to extract credentials from a Firefox profile.

3.Identifying decoy entries versus real flag fragments and reconstructing split flags.

4.Handling AD1 forensic images and extracting nested RARs to get to relevant files.

## Incorrect Tangents
1.Initially thinking the master password itself was the flag. It was only used to unlock the credentials.

2.Considering every decrypted string as potential flag material, which would have led to overthinking.

## Resources:

https://github.com/unode/firefox_decrypt
 
***

# 4. Re:Draw

Her screen went black and a strange command window flickered to life, lines of text flashed across before everything went silent. Moments later, the system crashed. By sheer luck, we recovered a memory dump. 

Note: There are three stages to this challenge and you will find three flags.

What we know: just before the crash, a black command window flickered across the screen, something in its output might still be visible if you dig through memory. She was drawing when it happened, and remnants of a painting program linger, which could reveal more if inspected in the right way. Finally, a mysterious archive hides deeper in memory, likely holding the last piece of her work.


## Solution:

### STAGE 1
First of all I listed the processes:
```
python2 vol.py -f MemoryDump_Lab1.raw --profile=Win7SP1x64 pslist
```

In the output I noticed:
```
cmd.exe      PID 1984
conhost.exe  PID 2692
```

This made sense. cmd.exe is the command prompt, and conhost.exe handles the actual console window. If the flag flashed on screen, it would be in the console buffer handled by these.

Next would be to Dump the console output

Volatility has a plugin called consoles that reads the screen buffers of command prompts. So I ran:
```
python2 vol.py -f MemoryDump_Lab1.raw --profile=Win7SP1x64 consoles
```

This printed out the text that was shown in command windows at the time of the memory capture. Scanning through the output, I found something that looked exactly like a CTF flag:
```
flag{th1s_1s_th3_1st_st4g3!!}
```

That was Stage 1 solved. The challenge description was right, the black command window really did contain the first flag.


### STAGE 2

The challenge mentioned remnants of a painting program. Based on this, I found *mspaint.exe* in the process list. I used *memdump* to dump the memory of the *mspaint.exe* process:

```
./volatility_2.6_lin64_standalone -f MemoryDump_Lab1.raw --profile=Win7SP1x64 memdump -p 2424 -D ./output
```

This saved the memory dump as *2424.dmp. I renamed it to **2424.data* and tried to open it using *GIMP* as raw image data. After several attempts adjusting offsets and dimensions, I finally managed to rotate and flip the image 180 degrees, revealing the second flag: 
```
flag{G00d_BoY_good_girL}
```

### STAGE 3 


First things first, locate the archive

I scanned all file objects in memory and filtered for archive types:
```
python2 vol.py -f MemoryDump_Lab1.raw --profile=Win7SP1x64 filescan | grep -Ei "rar|zip|7z"
```

This returned:
```
\Users\Alissa Simpson\Documents\Important.rar
```

So there it was. A RAR archive sitting in Alissa’s Documents folder, exactly matching the description.

I dumped it from memory:
```
python2 vol.py -f MemoryDump_Lab1.raw --profile=Win7SP1x64 dumpfiles -Q <offset> -D extracted
```

After extracting, I had Important.rar locally.

Next would be to understand the password hint

When I tried to open it:
```
unrar x Important.rar
```

it asked for a password and the challenge told me:
"
Password is NTLM hash (uppercase) of Alissa’s account password
"
So the task was not to crack her password, but to recover her NTLM hash from memory.

That pointed directly to LSASS, the Windows process that stores credentials in memory.

Next would be to find and dump LSASS

First I found LSASS:
```
python2 vol.py -f MemoryDump_Lab1.raw --profile=Win7SP1x64 pslist | grep -i lsass
```

Which gave:
```
lsass.exe   PID 492
```

Then I dumped its memory:
```
python2 vol.py -f MemoryDump_Lab1.raw --profile=Win7SP1x64 memdump -p 492 -D lsass_dump
```

This produced:
```
lsass_dump/492.dmp
```
Next extract NTLM hashes

The proper way to get NTLM hashes from a Windows 7 memory image is using Volatility’s hashdump plugin, which pulls credentials from the SAM and SYSTEM hives.

After fixing Volatility’s missing dependencies, I ran:
```
python2 vol.py -f MemoryDump_Lab1.raw --profile=Win7SP1x64 hashdump
```

The output looked like:
```
Alissa Simpson:1003:aad3b435b51404eeaad3b435b51404ee:f4ff64c8baac57d22f22edc681055ba6:::
```

I copied the NTLM hash part and converted it to uppercase, exactly as the archive asked for.

Final step is to Unlock the archive

Now I tried again:
```
unrar x Important.rar
```

When it asked for the password, I pasted the NTLM hash in uppercase. This time it worked and extracted a file called:

flag3.png emerged I opened it and the flag was revealed:
```
flag{w3ll_3rd_stage_was_easy}
```
## Flag:

```
1. flag{th1s_1s_th3_1st_st4g3!!}
2. flag{G00d_BoY_good_girL}
3. flag{w3ll_3rd_stage_was_easy}
```

## Concepts learnt:
1.Windows does not store passwords in plain text, it stores NTLM hashes and tools like Volatility can reconstruct those hashes from memory and registry hives which can then be used directly as secrets.

2.Files and user activity often survive in RAM even when they are not visible on disk, so memory forensics lets you recover archives, command output and credentials that never existed as normal files.

## Incorrect Tangents
I initially tried to brute force and string search the LSASS dump for Alissa’s password or hash, but Windows stores credentials in structured memory so raw grepping was misleading and mostly noise.

I also tried using pypykatz on a raw LSASS memory dump, not realizing it only works on proper Windows minidumps, which taught me that dump format matters as much as the data itself.

## Resources:

https://volatilityfoundation.org/2016/11/22/hashes-in-memory

https://www.forensicfocus.com/articles/extracting-clipboard-data-from-memory-dumps/

***
