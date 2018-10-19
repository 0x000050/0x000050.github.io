---
layout: post
title: Setting up windows kernel-mode debugging with WinDbg and VMware
tags: [Windwos-Kernel]
---


Since I heve recently managed to learn about Windows Kernel Exploit and reverse Windows Driver, I decided to take notes and write down my experience. The article talks about configuring for VMware and WinDbg, setting Windows Boot, WinDbg Command, and WinDbg theme(todo). If you are alos trying to debug Windows Kernel-mode via [The Windows Debugger (WinDbg)](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools), this may be helpful to you. 

WinDbg can be used to debug kernel and user mode code, analyze crash dumps and to examine the CPU registers as code executes. But I just using Windbg to debug Kernel-mode code and analyze crash dumps. If I need to debug User-mode code, I will choose [IDA Pro](https://www.hex-rays.com/products/ida/) or [ollydbg](http://www.ollydbg.de/)/[x64dbg](https://x64dbg.com/#start) to debug it.

## Prerequisites
### Install Windows OS in the VMware. 
* [Method 1] To create a virtual machine from an ISO image. (I'll be using [VMware Workstation](https://my.vmware.com/web/vmware/info/slug/desktop_end_user_computing/vmware_workstation_pro/15_0))
* [Method 2] Free download is also available from [Microsoft VM download page](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/)
### Install [WinDbg](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/debugger-download-tools) in host system

## Configuring for VMware 
We need to communication between Guest system and Host system, so we can using [name piped](https://docs.microsoft.com/en-us/windows/desktop/ipc/named-pipes) to simulate serial port (COM Port). We can add a named pipe serial port for connecting a virtual machine to an application or to another virtual machine that is running on the host system.

* Powered off the virtual machine.
* For the Debugger VM, `Right Click` and selected `Settings` button.
* Clicked `Add` button in the "VMware Machine Settings" dialog box. and selected `Serial Port` and click `Next` in the "Add Hardware Wizard" dialog box.

![](https://i.imgur.com/87ulCRB.png)

* On next page, select `Output to named pipe` and click `Next`.

![](https://i.imgur.com/QiRUCi5.png)

* Set a neme for `Name pipe`, and note it, we will using it for WinDbg. For example: `\\.\pipe\win7_x64`. And select `This end is the server` and `The other end is an application`. Check `Connect at power on` then click `Finish`.

![](https://i.imgur.com/e0nJSBJ.png)

* After clicking Finish, check `Yield CPU on poll`.

![](https://i.imgur.com/uGeCVmF.png)

* SpecialNote: Note that new serial port has number `2` which corresponds to `COM2`. We will use this number when we setting up `BCD`.

![](https://i.imgur.com/2FmI1lR.png)

## Windows Boot (Guest System)
In Windows XP, we can modify `boot.ini` to change Windows startup options. After Windows 7, boot application settings stored in `BCD`, we can using [BCDedit](https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/bcdedit-command-line-options) to modify `BCD` file.

### Windows XP - `boot.ini`
* `boot.ini` control how the operating system is booted and any startup options
### Windows 7 / Windows 10 - `Boot Configuration Data (BCD)`
 
* Run `cmd` as "administrator", and execute the following commands:
* Use Serial Port (COM Port) as the communication between debugger and debuggee. The number of debugport should be base on VMware's Serial Port setting, we just We got Serial Port number is `2`.
```
bcdedit /dbgsettings serial debugport:2    
```
* Enables or disables the kernel debugger for a specified boot entry. You can use the following syntax to enable kernel or boot debug.
```
bcdedit /debug ON
```

* Restart Guest System

SpecialNote: we just modify original boot option, instead of adding a boot option. We can save some time when booting.

## WinDbg
If everythings goes well, We can Start the Debugging Session Using WinDbg at the command line on the Host System, using `cmd.exe` 

```
windbg -b -k com:pipe,port=\\.\pipe\win7_x64,resets=0
```

About that `port`, if the debugger is running on the same computer as the virtual machine, enter the following for Port. The `PipeName` from Serial Port, we just setting. If we forget it, we can find it at VMware (`VM > Settings... > Serial Port2 > Use named pipe`)

```
\\.\pipe\PipeName.
```

For more details, reference [Microsoft page](https://docs.microsoft.com/en-us/windows-hardware/drivers/debugger/attaching-to-a-virtual-machine--kernel-mode-)

### Load Symbols in WinDbg
You can load symbols and saving the symbol path as part of a workspace.

#### [Method 1] Load symbols by WinDbg menu
* `File` > `Symbol File Path...` (or pressing `Ctrl`+`S`) 
* Input symbol path, and check `reload`. We Usually download Microsoft symbols, if you have  Symbols of your application, locally, flat list of PDB files, you also can load its.
```
SRV* c:\Symbols *http://msdl.microsoft.com/download/symbols
```

#### [Method 2] Load symbols by WinDbg Command line 

First time, you need to download symbols from Microsoft. 
```
.sympath srv*c:\Symbols*https://msdl.microsoft.com/download/symbols
```
After that, you just need to set local symbol folder, and reload symbol.
```
.symfix+ c:\symbols
.reload
```

#### [Method 3] Load symbols by Windows environment variable
There is a environment variable called `_NT_SYMBOL_PATH` which can be set to a symbol path as well. 
* SpecialNote: If we setting the environment variable. Not only impact WinDbg, but also Visual Studio, Process Explorer, Process Monitor and potentially other software.

#### [Method 4] Load Symbols by Windows Commands
`WinDbg -y "<symbol path>`

### Enable Output of DbgPrint 

* start / run / `regedit`
* Add the key `Debug Print Filter` in `HKLM\SYSTEM\CCS\Control\Session Manager`
* Under this key(`HKLM\SYSTEM\CCS\Control\Session Manager\Debug Print Filter`), create a value with the name `DEFAULT` Set the value of this key equal to the DWORD value `0x0000ffff`
    * SpecialNote: Don't set the value named `default`
