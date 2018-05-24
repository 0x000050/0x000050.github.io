---
layout: post
title: 2017 Flare-On Challenge 3 Greektome.exe
tags: [Flare-On, Reverse]
---

`Greek_to_me.exe` is a Windows x86 executable. When we launch the binary, we can't input anything, and we don't get any output from the binary. It gets stuck waiting for us to do something. 

* Analysis:

When we open it in IDA, we can see it contains the `socket` function at `0x401151`, the binary using a standard series of Windows API functions: `socket`, `bind`, `listen`, and `accept`. 

So far we know the binary listening for a TCP connection from localhost on port 2222 (0x8AE). It then proceeds to read up to 8-bit register in `sub_401121`, and the input is a 32-bit integer. 

![](https://i.imgur.com/szZCmXr.png)

Additionally, there has some interesting string of the program at virtual address `0x401101`, that is `Congratulations! But wait, where',27h,'s my flag?`

![](https://i.imgur.com/fSeXBij.png)

At `0x401036`, the first byte from the recv buffer is moved into the lower eight bits of the EDX register. Then, focus on `loc_40107C` function and `loc_401039` function, the address `0x40107C` is moved into the `EAX` register, representing the start address for the decoding loop. 

The decoding loop contains some operations. First of all, take a one byte at the address stored in `EAX` (`0x40107C`). Next, XOR the extracted byte with the first byte received over the listening socket, then incremented by 34 (0x22). Use the resulting byte to overwrite the byte extracted in `EAX`.

![](https://i.imgur.com/tx8jK96.png)

Further, `sub_4011E6` function is used to hash, arguments are the start address of the modified code (`0x40107C`) and the length value 121 (0x79). In `sub_4011E6` function, a 16 bits (`AX`) hash data is calculated and checked against 64350 (0xFB5E) for equality. If the hash data matches, the code is executed, without getting an error message `Nope, that’s not it`.

* Solutions:
 
The key is only a single byte, but there are 256 possibilities. Following python code to help us get `0xA2` (`ó`) as the key:

```python
import os 
import socket
import string
import time

#scoket info
HOST = '127.0.0.1'    
PORT = 2222              

for i in range(256):
	#open exe
	filepath = 'greek_to_me.exe'
	os.startfile(filepath)
	time.sleep(0.1s)

	#connect socket
	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
	s.connect((HOST, PORT))
	s.sendall(chr(i))
	data = s.recv(1024)
	s.close()
	if (data == "Congratulations! But wait, where's my flag?"):
		print i
		print chr(i)
		print 'Received', repr(data)
```

Put a breakpoints at `0x401063` with IDA, we can find the flag that got written to the stack was `et_tu_brute_force@flare-on.com`

* Notes:
    * Compared with `127.0.0.1` and `0.0.0.0`: 
        * `127.0.0.1` is only for local interface, when a server listen `127.0.0.1` that means "only communicate within the same host".  
        * when a server listen on `0.0.0.0` that means "listen on every available network interface". 

- - -
* Challenge Author: Matt Williams (@0xmwilliams)
* [Challenge download](https://github.com/0x000050/CTF/blob/master/2017_Flare-On/03_Greek_to_me/greek_to_me.exe)
