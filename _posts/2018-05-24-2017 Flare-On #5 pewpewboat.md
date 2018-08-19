---
layout: post
title: 2017 Flare-On Challenge 5 pewpewboat.exe
tags: [Flare-On, Reverse]
---

Actually, `pewpewboat.exe` is not a Windows	PE file but an x64 ELF binary. When we launch the binary in linux, we can see that is a the Battleship game.

```
Loading first pew pew map...
   1 2 3 4 5 6 7 8
  _________________
A |_|_|_|_|_|_|_|_|
B |_|_|_|_|_|_|_|_|
C |_|_|_|_|_|_|_|_|
D |_|_|_|_|_|_|_|_|
E |_|_|_|_|_|_|_|_|
F |_|_|_|_|_|_|_|_|
G |_|_|_|_|_|_|_|_|
H |_|_|_|_|_|_|_|_|

Rank: Seaman Recruit

Welcome to pewpewboat! We just loaded a pew pew map, start shootin'!

Enter a coordinate:
```

Let's try to typing some "coordinate" to see what happens. There will be two results: `You missed :( ` and `Nice shot!	Hit!`, if we get all of we need to hit coordinate, we can get message `sunk all the ships`.

```
   1 2 3 4 5 6 7 8
A |_|_|_|_|_|_|_|_|
B |_|_|_|X|X|X|X|_|
C |_|_|_|X|_|_|_|_|
D |_|_|_|X|_|_|_|_|
E |_|_|_|X|X|X|X|_|
F |_|_|_|X|_|_|_|_|
G |_|_|_|X|_|_|_|_|
H |_|_|_|_|_|_|_|_|

Rank: Seaman Recruit

Nice shot! Hit!
You sunk all the ships!!


NotMd5Hash("BNKM") >
```

And we need to typing something in last line `NotMd5Hash("BNKM") >`, when we try a few times, we can know the letters inside the quotation mark is random. So we can patch the verification method, in order for us to continue to the next game.
  
* Analysis:

We can install "gdbserver" tool in ubuntu, and bind ip in `0.0.0.0`. Thus we can dynamically debug with IDA Pro.

`0x403530` function for `notmd5hash`, in this function we can find words and `rand()`.
```
  v7 = 'N';
  v8 = 'o';
  v9 = 't';
  v10 = 'M';
  v11 = 'd';
  v12 = '5';
  v13 = 'H';
  v14 = 'a';
  v15 = 's';
  v16 = 'h';
  v17 = '(';
  v18 = '"';
  v19 = '%';
  v20 = 's';
  v21 = '"';
  v22 = ')';
  v23 = ' ';
  v24 = '>';
  v25 = ' ';
  v26 = '\0';
```

After we patch `0x403BE0 call sub_403530`, we can make the position guess of the coordinates more smoothly, or reversing to find the correct coordinates in the binary.

* Solutions:

I once also solved a challenge about Battleship game, the challenge name was [winmine.exe](http://0x00.tw/2017/08/26/2017-HITCON-CMT-mini-wargame/), one of challenge in HITCON CMT 2017 mini wargame. At that time, I reverse to find correct coordinates in binary. After the game, someone provided a different solution for me, I think this solution is very interesting. That is using [vmware workstation snapshot](https://www.vmware.com/tw/products/workstation-pro.html) to remember the game screen that has been marked. 

We collect letters every turn, when we sunk all the ships, we got the those letters from game screen.

```
F
H
G
U
Z
R
E
J
V
O
```
Reaching the final level, we got those message
```
You sunk all the ships!!
   1 2 3 4 5 6 7 8
A |_|_|_|_|_|_|_|_|
B |_|_|_|_|_|_|_|_|
C |_|_|_|_|_|_|_|_|
D |_|_|_|_|_|_|_|_|
E |_|_|_|_|_|_|_|_|
F |_|_|_|_|_|_|_|_|
G |_|_|_|_|_|_|_|_|
H |_|_|_|_|_|_|_|_|
                   
Rank: Congratulation!
                     
Aye!PEWYouPEWfoundPEWsomePEWlettersPEWdidPEWya?PEWToPEWfindPEWwhatPEWyou'rePEWlookingPEWfor,PEWyou'llPEWwantPEWtoPEWre-orderPEWthem:PEW9,PEW1,PEW2,PEW7,PEW3,PEW5,PEW6,PEW5,PEW8,PEW0,PEW2,PEW3,PEW5,PEW6,PEW1,PEW4.
PEWNextPEWyouPEWletPEW13PEWROTPEWinPEWthePEWsea!PEWTHEPEWFINALPEWSECRETPEWCANPEWBEPEWFOUNDPEWWITHPEWONLYPEWTHEPEWUPPERPEWCASE.
Thanks for playing!  
```

After we delete "PEW" words, we can get real messages.

```
Aye! You found some letters did ya? To find what you're looking for, you'll want to
re-order them:
9, 1, 2, 7, 3, 5, 6, 5, 8, 0, 2, 3, 5, 6, 1, 4.

Next you let 13 ROT in the sea! THE FINAL SECRET CAN BE FOUND WITH ONLY THE UPPER CASE.

Thanks for playing!
```

According this message to re-order we collect letters `FHGUZREJVO`. The result is `OHGJURERVFGUREHZ`. Next, let "ROT13" the string to become `BUTWHEREISTHERUM`.

```
   1 2 3 4 5 6 7 8
A |_|_|_|_|_|_|_|_|
B |_|_|_|_|_|_|_|_|
C |_|_|_|_|_|_|_|_|
D |_|_|_|_|_|_|_|_|
E |_|_|_|_|_|_|_|_|
F |_|_|_|_|_|_|_|_|
G |_|_|_|_|_|_|_|_|
H |_|_|_|_|_|_|_|_|

Rank: Seaman Recruit

Welcome to pewpewboat! We just loaded a pew pew map, start shootin'!

Enter a coordinate: BUTWHEREISTHERUM

 very nicely done!  here have this key:  y0u__sUnK_mY__P3Wp3w_b04t@flare-on.com

```

The flag is: `y0u__sUnK_mY__P3Wp3w_b04t@flare-on.com`


- - -
* Challenge Author: Tyler Dean (@spresec)
* [Challenge download](https://github.com/0x000050/CTF/blob/master/2017_Flare-On/05_Pewpewboat/pewpewboat.exe)
