---
layout: post
title: 2017 Flare-On Challenge 4 Notepad.exe
tags: [Flare-On, Reverse]
---

`Notepad.exe` is a Windows x86 executable, it seems to be a modified version of Microsoft’s `Notepad.exe`. Let's launch this binary and it looks nothing special.

![](https://i.imgur.com/ZtdIJB9.png)

* Analysis:

IDA can properly apply Microsoft’s PDB for `Notepad.exe` from Microsoft’s symbol server. But the entry point of executable has been modified to `0x1013a00`.

![](https://i.imgur.com/6KjROey.png)

Go to `0x1013a00` function, we can see  interesting thing is entry point in the `.rsrc` section, instead of `.text` section. And we can see interesting strings: `%USERPROFILE%\flareon2016challenge` This is hint means you will need the binaries from the "flareon 2016 challenges". (We can download these files from the [official website](http://www.flare-on.com/)), at the same time,  using stack based strings is also one of the skills of malware.

![](https://i.imgur.com/jWZce3e.png)

Here we can restore these strings:
```
%USERPROFILE%\flareon2016challenge
ImageHlp.dll
CheckSumMappedFile
User32.dll
MessageBoxA
```
And we can get other strings from "Strings window" in IDA 

```
\\key.bin
%USERPROFILE%
\\flareon2016challenge
where's my key file?
what's wrong with my key file?
```

After that, we can see other one skills of malware: Dynamically loading library modules. we can using IDA dynamically debug to understand those library. This is originally version.

```
  v77 = 0x1013C4E;
  v6 = sub_10153D0(0x8FECD63F);
  v89 = sub_1015310(v6, 0x63D6C065);
  v90 = sub_1015310(v6, 0xA5E1AC97);
  v91 = sub_1015310(v6, 0x23545978);
  v93 = sub_1015310(v6, 0x7C0017A5);
  v94 = sub_1015310(v6, 0x56C61229);
  v95 = sub_1015310(v6, 0x7B073C59);
  v96 = sub_1015310(v6, 0xFFD97FB);
  v109 = sub_1015310(v6, 0x10FA6516);
  v97 = sub_1015310(v6, 0xE80A791F);
  v98 = sub_1015310(v6, 0xDF7D9BAD);
  v99 = sub_1015310(v6, 0xB12C56D7);
  v100 = sub_1015310(v6, 0xEC0E4E8E);
  v101 = sub_1015310(v6, 0x7C0DFCAA);
  v103 = sub_1015310(v6, 0xD3324904);
  v104 = sub_1015310(v6, 0xB2089259);
  v105 = sub_1015310(v6, 0xEEB585D8);
  v106 = sub_1015310(v6, 0x3810CB0F);
  v107 = sub_1015310(v6, 0xF02A93BE);
  v108 = sub_1015310(v6, 0xF72A53BA);
```

First, we follow first line `v77 = 0x1013c4e`, that means `v77`'s value stored `EIP`. We can change assembly view to understand it, first line is `$+5` also mean `$pc+5`. In IDA, `$` is beginning of same instruction (which is not the `EIP` which would point to the next instruction). And `call $+5` is probably call to next instruction, and next line for `pop` address, such usage usually used to write shellcode, it used to get EIP address.

```
.rsrc:01013C49 call    $+5              //$pc+5
.rsrc:01013C4E pop     [ebp+var_88]
```

and we can using IDA dynamically debug to get those library's name. 

```
  v77 = 0x1013C4E;                              // $+5
  user32.dll = sub_10153D0(0x8FECD63F);
  kernel32_FindFirstFileA = sub_1015310(user32.dll, 0x63D6C065);
  kernel32_FindNextFileA = sub_1015310(user32.dll, 0xA5E1AC97);
  kernel32_FindClose = sub_1015310(user32.dll, 0x23545978);
  kernel32_CreateFileA = sub_1015310(user32.dll, 0x7C0017A5);
  kernel32_CreateFileMappingA = sub_1015310(user32.dll, 0x56C61229);
  kernel32_MapViewOfFile = sub_1015310(user32.dll, 0x7B073C59);
  kernel32_CloseHandle = sub_1015310(user32.dll, 0xFFD97FB);
  kernel32_ReadFile = sub_1015310(user32.dll, 0x10FA6516);
  kernel32_WriteFile = sub_1015310(user32.dll, 0xE80A791F);
  kernel32_GetFileSize = sub_1015310(user32.dll, 0xDF7D9BAD);
  kernel32_FlushViewOfFile = sub_1015310(user32.dll, 0xB12C56D7);
  kernel32_LoadLibraryA = sub_1015310(user32.dll, 0xEC0E4E8E);
  kernel32_GetProcAddress = sub_1015310(user32.dll, 0x7C0DFCAA);
  kernel32_GetModuleHandleA = sub_1015310(user32.dll, 0xD3324904);
  kernel32_UnmapViewOfFile = sub_1015310(user32.dll, 0xB2089259);
  kernel32_ExpandEnvironmentStringsA = sub_1015310(user32.dll, 0xEEB585D8);
  kernel32_FileTimeToSystemTime = sub_1015310(user32.dll, 0x3810CB0F);
  kernel32_GetTimeFormatA = sub_1015310(user32.dll, 0xF02A93BE);
  kernel32_GetDateFormatA = sub_1015310(user32.dll, 0xF72A53BA);
```

At the end of the entry point’s function, we know the program then proceeds to look for files in `%USERPROFILE%\flareon2016challenge` that have PE-headers using the `FindFirstFileA` / `FindNextFileA` API. When it finds an executable file, calling the function at `0x1014E20` to infect it. This type malware is called `PE infector`. 

At `0x1014E20` function, it will be find MZ haeder, PE header, compares timestamp value and infector other PE file. About PE executable format, we can using "010 editor" tool's PE template [PETemplate.bt](https://www.sweetscape.com/010editor/templates/) to assist analysis PE timestamp. At `0x10146C0`, it compares the compile timestamp value of the PE file that is currently executing against a hard-coded value, then compares the compile timestamp value of the discovered PE against another hard-coded value. This comparison is repeated for several pairs of timestamp values until both are matched. At `0x1014E20`, the infection code at `0x101500B`, it checks for the value `0x8675309` at offset `0x1C` in the PE and does not infect it if found. When infecting, it adds this value to that offset of the PE. This is known as an "infection marker" Make sure the execute file is only infected once.

When a successful match is found the second timestamp value is converted to a string and printed in a message box and generate `key.bin`. About `key.bin`, we can follow `0x10145B0` function, where eight bytes from offset `0x0C` in the PE is appended to a file named `key.bin`. This is first message box pop up.

![](https://i.imgur.com/R2IiXIW.png)

* Solutions:

So far we know that we need some files that meet the requirements (timestamp) in 2016’s FLARE On challenge. After finding it, put those files in the `%USERPROFILE%\flareon2016challenge` directory. Next, `notepad.exe` will check the timestamp of those files. If it match, the file timestamp value is converted to a string and printed in a message box, and where "eight bytes" from offset `0x0C` in the PE is appended to ` Key .bin` file.

In the `0x1014B4B` to `0x1014D0A` code block, it used to check the timestamp of the file and write `key.bin` file, we total need of four files, include the order of file and file timestamp

|Timestamp of infected file| Timestamp of 2016’s FLARE On challenge| challenge name|
|-|-|-|
|0x57D1B2A2| 0x48025287| (Challenge1) challenge1.exe
|0x57D2B0F8 | 0x57d1b2a2| (Challenge2) DudeLocker.exe
|0x49180192| 0x57D2B0F8| (Challenge6) khaki.exe
|0x579E9100| 0x49180192| (Challenge3) unknow

The final timestamp value comparison is only performed on the running executable. If it matches, 32 bytes are read from the `key.bin` file and are XORed against a 32 byte string of unprintable characters stored in a local variable.

Finally, Running the binaries in this order is challenge 1.exe, DudeLocker.exe, khaki.exe, and unknown. We can get the flag in message box.

`bl457_fr0m_th3_p457@flare-on.com`

- - -
* Challenge Author: James T. Bennett (@jtbennettjr)
* [Challenge download](https://github.com/0x000050/CTF/blob/master/2017_Flare-On/04_Notepad/notepad.exe)
