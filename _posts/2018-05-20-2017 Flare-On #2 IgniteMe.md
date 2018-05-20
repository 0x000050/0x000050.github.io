---
layout: post
title: 2017 Flare-On Challenge 2 IgniteMe.exe
tags: [Flare_On, Reverse]
---
>Challenge #2 by Nhan Huynh

* [Challenge download](https://github.com/0x000050/CTF/blob/master/2017_Flare-On/02_IgniteMe/IgniteMe.exe)

This is a crack me challenge. IgniteMe.exe expects to run without any command line argument. It asks the player to input the	 flag and checks whether it's correct or not.

```
C:\Users\nnyl\Desktop\flareon4\Challenge2>IgniteMe.exe
G1v3 m3 t3h fl4g:  flareon
N0t t00 h0t R we? 7ry 4ga1nz plzzz!
```

* Analysis:

![](https://i.imgur.com/3rA8jZy.png)

`sub_4010F0` function is used to read user input data，and stored the user input data at `0x403078`.

The `sub_401050` function is flag validation function, focusing on code block between `0x401088` and `0x4010AD`, you can see where the input is processed by xor loop, and checks it against some data `enc` (at `0x0403000`).

![](https://i.imgur.com/hoqtTCR.png)

`sub_401000` function is used to generate key, we can get an initial seed to be the number 4.

* Solutions:

In the first loop, `IgniteMe.exe` XOR's 0x04 with the last character of user input data.  The result is then XOR'd with the second from last char, and it keeps on going.  The global variable `byte_403180` stores the encoded user input data. 

Upon completion of encoded user input data, checks for correctness of the decrypted data (`enc`). if it correct print `G00d j0b!`.

Therefore, we can bruteforced the `enc`

```
.data:00403000 ; char enc[]
.data:00403000 enc dd 4549260Dh
.data:00403004 dd 4478172Ah
.data:00403008 dd 5E5D6C2Bh
.data:0040300C dd 172F1245h
.data:00403010 dd 6E6F442Bh
.data:00403014 dd 455F0956h
.data:00403018 dd 0A267347h
.data:0040301C dd 4817130Dh
.data:00403020 dd 4D400142h
.data:00403024 dd 69020Ch
```

The following script obtains the flag.

```python
enc = [0x0D, 0x26, 0x49, 0x45, 0x2A, 0x17, 0x78, 0x44, 0x2B, 0x6C, 0x5D, 0x5E, 0x45, 0x12, 0x2F, 0x17, 0x2B, 0x44, 0x6F, 0x6E, 0x56, 0x09, 0x5F, 0x45, 0x47, 0x73, 0x26, 0x0A, 0x0D, 0x13, 0x17, 0x48, 0x42, 0x01, 0x40, 0x4D, 0x0C, 0x02, 0x69]

import sys
v4 = 4                    #key
ans = ''
for i in range(len(enc)):
    v4 = v4 ^ enc[-i-1]
    ans += chr(v4)
    print hex(enc[-i-1])
print ans[::-1]
```

Flag: `R_y0u_H0t_3n0ugH_t0_1gn1t3@flare-on.com`
