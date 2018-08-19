---
layout: post
title: 2017 HITCON CMT mini wargame Reverse
tags: [HITCON, Reverse, CTF]
---

今年 HITCON 在 HITCON CMT 研討會期間又舉辦了與前幾年類似的 wargame，相較於 HITCON CTF 是比較簡單的小型競賽。很幸運跟以色列的朋友一起組隊，因為我很 Lazy 而他們是 Hacker，所以隊名就取 `Lazyhacker`。在團隊中我負責解 Reverse 的題型，這次 Reverse 只有兩題，都是 Windows 題，分別是小算盤跟踩地雷，還蠻有趣的，所以想紀錄一下。

- - -

## [100] calc.exe
### 題目描述
毫無反應，就是一個具有計算功能的計算機。

![](https://i.imgur.com/DMo6dMi.png)


### 解法

觀察發現似乎跟 Windows 的小算盤一模一樣，因此拿了 Windows 原版的 `calc.exe` 執行檔來做 diff ( 這裡用 `vimdiff` )，發現其中多了一些看起來很可疑的 code

![](https://i.imgur.com/6NRAkp5.png)
  
放進 IDA Pro 分析 `sub_10136B0()` (可以 search opcode 或計算 offset)，得知是在做某種字串的比對

![](https://i.imgur.com/NAUhSNt.png)

簡化來說，就是取數字框的 `HWND`，讀數字框上面的文字，並驗證這段文字
```
HWND hwnd = GetDlgItem(g_hWnd, 403);  //  403 是數字框的控制項 ID
GetWindowTextA(hwnd, buf, 256);
驗證這段文字
```

[補充] 控制項的 ID 可以透過 Resource Hacker 來看
![](https://i.imgur.com/VPDAWFP.png)


也可以用 ollydbg 追蹤來理解程式的行為，透過追蹤可以知道: 當按下 `=` 時，會跳到這個有被改過的地方，至於要輸入什麼數字才會跳出 FLAG 呢? 可以停在 `101371A` 這裡看他怎麼比較的，他比較的值分別是`0x31323035`, `0x32303536`, `0x352E3631` 與 `0x36313230`

![](https://i.imgur.com/jl2IAu6.png)

對照 ASCII 表並考慮 little endian 的話，就是 `5021650216.50216`

```
0x31323035 --> 5021
0x32303536 --> 6502
0x352E3631 --> 16.5
0x36313230 --> 0216
```

如果用 IDA Pro 來看的話也可以解回來，判斷的部分為
![](https://i.imgur.com/kQuEkdy.png)

因此只要在視窗中輸入`5021650216.50216` 並按下 `=` 
![](https://i.imgur.com/7qEETTe.png)

就會跳出小框框顯示 Flag 
![](https://i.imgur.com/QTbNeYZ.png)
`The flag is: hitcon{50216_is_our_god}`

## [200] winmine.exe
### 題目描述
一個不平凡的踩地雷，開啟就是 40*40 的大小，除了特定的位置外，其他格子都是地雷

![](https://i.imgur.com/f4v3lfE.png)
![](https://i.imgur.com/PbyCTrV.png)

### 解法
踩地雷遊戲，執行後很幸運地踩到了幾次正確位置

![](https://i.imgur.com/16XPA2L.jpg)
![](https://i.imgur.com/jmsMbVy.jpg)

猜想 "正確位置出現的形狀" 可能依序就是整串 FLAG 依序出現的順序( 可能是 `THE flag is hitcon{xxx}` 之類的 )，因此期望在 IDA Pro 之中看到放的位置

在 `sub_962880()` 之中有 `MessageBoxA(0, "Are u a hacker O_o?", "O_o?", 0x20u);` 猜想可能驗證 flag 相關的東西在附近

```
.text:009629E1 loc_9629E1:                             ; CODE XREF: sub_962880+12Dj
.text:009629E1                 mov     ebx, ds:GetTickCount
.text:009629E7                 add     edi, 0FFFFFFFDh
.text:009629EA                 call    ebx ; GetTickCount
.text:009629EC                 xor     edx, edx
.text:009629EE                 mov     ecx, 0FFh
.text:009629F3                 div     ecx
.text:009629F5                 mov     esi, edx
.text:009629F7                 call    sub_965797
.text:009629FC                 add     eax, esi
.text:009629FE                 xor     edx, edx
.text:00962A00                 div     edi
.text:00962A02                 lea     esi, [edx+1]
.text:00962A05                 call    ebx ; GetTickCount
.text:00962A07                 call    sub_965797
.text:00962A0C                 mov     ecx, g_check
.text:00962A12                 mov     edx, 5
.text:00962A17                 shl     ecx, 4
.text:00962A1A                 sub     ecx, g_check
.text:00962A20                 imul    eax, esi, 65h
.text:00962A23                 add     ecx, offset g_enc
.text:00962A29                 add     eax, offset unk_9775E1
.text:00962A2E                 xchg    ax, ax
.text:00962A30

```

最後從 `0x00962A23` 跳過去 `0x00973BD9` 會看到這段


```
.rdata:00973BD8                 db    1
.rdata:00973BD9 g_enc           db    1                 ; DATA XREF: sub_962880+1A3o
.rdata:00973BDA                 db    1
.rdata:00973BDB                 db    0
.rdata:00973BDC                 db    1
.rdata:00973BDD                 db    0
.rdata:00973BDE                 db    0
.rdata:00973BDF                 db    1
.rdata:00973BE0                 db    0
.rdata:00973BE1                 db    0
.rdata:00973BE2                 db    1
.rdata:00973BE3                 db    0
.rdata:00973BE4                 db    0
.rdata:00973BE5                 db    1
.rdata:00973BE6                 db    0
.rdata:00973BE7                 db    1
.rdata:00973BE8                 db    0
.rdata:00973BE9                 db    1
.rdata:00973BEA                 db    1
.rdata:00973BEB                 db    0
.rdata:00973BEC                 db    1
.rdata:00973BED                 db    1
.rdata:00973BEE                 db    1
.rdata:00973BEF                 db    1
.rdata:00973BF0                 db    1
.rdata:00973BF1                 db    0
.rdata:00973BF2                 db    1
.rdata:00973BF3                 db    1
.rdata:00973BF4                 db    0
.rdata:00973BF5                 db    1
.rdata:00973BF6                 db    1
.rdata:00973BF7                 db    1
.rdata:00973BF8                 db    1
.rdata:00973BF9                 db    1
.rdata:00973BFA                 db    0
.rdata:00973BFB                 db    0
.rdata:00973BFC                 db    1
.rdata:00973BFD                 db    1
.rdata:00973BFE                 db    1
.rdata:00973BFF                 db    1
.rdata:00973C00                 db    0
.rdata:00973C01                 db    0
.rdata:00973C02                 db    1
.rdata:00973C03                 db    1
.rdata:00973C04                 db    1
.rdata:00973C05                 db    1
.rdata:00973C06                 db    1
.rdata:00973C07                 db    1
.rdata:00973C08                 db    1
.rdata:00973C09                 db    0
.rdata:00973C0A                 db    0
.rdata:00973C0B                 db    1
.rdata:00973C0C                 db    1
.rdata:00973C0D                 db    1
.rdata:00973C0E                 db    1
.rdata:00973C0F                 db    0
.rdata:00973C10                 db    0
.rdata:00973C11                 db    1
.rdata:00973C12                 db    0
.rdata:00973C13                 db    0
.rdata:00973C14                 db    1
.rdata:00973C15                 db    0
.rdata:00973C16                 db    0
.rdata:00973C17                 db    1
.rdata:00973C18                 db    0
.rdata:00973C19                 db    0
.rdata:00973C1A                 db    1
.rdata:00973C1B                 db    0
.rdata:00973C1C                 db    0
.rdata:00973C1D                 db    1
.rdata:00973C1E                 db    0
.rdata:00973C1F                 db    0
.rdata:00973C20                 db    1
.rdata:00973C21                 db    1
.rdata:00973C22                 db    1
.rdata:00973C23                 db    1
.rdata:00973C24                 db    1
.rdata:00973C25                 db    1
.rdata:00973C26                 db    1
.rdata:00973C27                 db    0
.rdata:00973C28                 db    1
.rdata:00973C29                 db    1
.rdata:00973C2A                 db    1
.rdata:00973C2B                 db    1
.rdata:00973C2C                 db    1
.rdata:00973C2D                 db    0
.rdata:00973C2E                 db    1
.rdata:00973C2F                 db    1
.rdata:00973C30                 db    0
.rdata:00973C31                 db    1
.rdata:00973C32                 db    1
.rdata:00973C33                 db    1
.rdata:00973C34                 db    1
.rdata:00973C35                 db    1
.rdata:00973C36                 db    0
.rdata:00973C37                 db    0
.rdata:00973C38                 db    1
.rdata:00973C39                 db    0
.rdata:00973C3A                 db    1
.rdata:00973C3B                 db    1
.rdata:00973C3C                 db    0
.rdata:00973C3D                 db    1
.rdata:00973C3E                 db    1
.rdata:00973C3F                 db    1
.rdata:00973C40                 db    1
.rdata:00973C41                 db    1
.rdata:00973C42                 db    1
.rdata:00973C43                 db    1
.rdata:00973C44                 db    0
.rdata:00973C45                 db    1
.rdata:00973C46                 db    0
.rdata:00973C47                 db    0
.rdata:00973C48                 db    1
.rdata:00973C49                 db    0
.rdata:00973C4A                 db    0
.rdata:00973C4B                 db    1
.rdata:00973C4C                 db    0
.rdata:00973C4D                 db    1
.rdata:00973C4E                 db    1
.rdata:00973C4F                 db    1
.rdata:00973C50                 db    1
.rdata:00973C51                 db    1
.rdata:00973C52                 db    1
.rdata:00973C53                 db    1
.rdata:00973C54                 db    0
.rdata:00973C55                 db    0
.rdata:00973C56                 db    1
.rdata:00973C57                 db    1
.rdata:00973C58                 db    1
.rdata:00973C59                 db    0
.rdata:00973C5A                 db    0
.rdata:00973C5B                 db    1
.rdata:00973C5C                 db    1
.rdata:00973C5D                 db    1
.rdata:00973C5E                 db    1
.rdata:00973C5F                 db    1
.rdata:00973C60                 db    0
.rdata:00973C61                 db    0
.rdata:00973C62                 db    1
.rdata:00973C63                 db    0
.rdata:00973C64                 db    0
.rdata:00973C65                 db    1
.rdata:00973C66                 db    1
.rdata:00973C67                 db    1
.rdata:00973C68                 db    1
.rdata:00973C69                 db    0
.rdata:00973C6A                 db    1
.rdata:00973C6B                 db    1
.rdata:00973C6C                 db    0
.rdata:00973C6D                 db    1
.rdata:00973C6E                 db    0
.rdata:00973C6F                 db    0
.rdata:00973C70                 db    0
.rdata:00973C71                 db    0
.rdata:00973C72                 db    1
.rdata:00973C73                 db    0
.rdata:00973C74                 db    0
.rdata:00973C75                 db    0
.rdata:00973C76                 db    0
.rdata:00973C77                 db    0
.rdata:00973C78                 db    1
.rdata:00973C79                 db    0
.rdata:00973C7A                 db    0
.rdata:00973C7B                 db    1
.rdata:00973C7C                 db    0
.rdata:00973C7D                 db    0
.rdata:00973C7E                 db    0
.rdata:00973C7F                 db    0
.rdata:00973C80                 db    0
.rdata:00973C81                 db    1
.rdata:00973C82                 db    0
.rdata:00973C83                 db    1
.rdata:00973C84                 db    1
.rdata:00973C85                 db    1
.rdata:00973C86                 db    0
.rdata:00973C87                 db    1
.rdata:00973C88                 db    0
.rdata:00973C89                 db    0
.rdata:00973C8A                 db    1
.rdata:00973C8B                 db    0
.rdata:00973C8C                 db    0
.rdata:00973C8D                 db    0
.rdata:00973C8E                 db    0
.rdata:00973C8F                 db    0
.rdata:00973C90                 db    0
.rdata:00973C91                 db    0
.rdata:00973C92                 db    1
.rdata:00973C93                 db    1
.rdata:00973C94                 db    1
.rdata:00973C95                 db    1
.rdata:00973C96                 db    0
.rdata:00973C97                 db    0
.rdata:00973C98                 db    1
.rdata:00973C99                 db    1
.rdata:00973C9A                 db    1
.rdata:00973C9B                 db    0
.rdata:00973C9C                 db    0
.rdata:00973C9D                 db    0
.rdata:00973C9E                 db    0
.rdata:00973C9F                 db    0
.rdata:00973CA0                 db    0
.rdata:00973CA1                 db    1
.rdata:00973CA2                 db    1
.rdata:00973CA3                 db    1
.rdata:00973CA4                 db    1
.rdata:00973CA5                 db    0
.rdata:00973CA6                 db    1
.rdata:00973CA7                 db    1
.rdata:00973CA8                 db    1
.rdata:00973CA9                 db    1
.rdata:00973CAA                 db    0
.rdata:00973CAB                 db    0
.rdata:00973CAC                 db    0
.rdata:00973CAD                 db    0
.rdata:00973CAE                 db    0
.rdata:00973CAF                 db    0
.rdata:00973CB0                 db    1
.rdata:00973CB1                 db    1
.rdata:00973CB2                 db    0
.rdata:00973CB3                 db    1
.rdata:00973CB4                 db    0
.rdata:00973CB5                 db    1
.rdata:00973CB6                 db    1
.rdata:00973CB7                 db    0
.rdata:00973CB8                 db    1
.rdata:00973CB9                 db    0
.rdata:00973CBA                 db    1
.rdata:00973CBB                 db    1
.rdata:00973CBC                 db    0
.rdata:00973CBD                 db    1
.rdata:00973CBE                 db    0
.rdata:00973CBF                 db    1
.rdata:00973CC0                 db    1
.rdata:00973CC1                 db    0
.rdata:00973CC2                 db    0
.rdata:00973CC3                 db    1
.rdata:00973CC4                 db    0
.rdata:00973CC5                 db    0
.rdata:00973CC6                 db    1
.rdata:00973CC7                 db    1
.rdata:00973CC8                 db    1
.rdata:00973CC9                 db    1
.rdata:00973CCA                 db    0
.rdata:00973CCB                 db    1
.rdata:00973CCC                 db    0
.rdata:00973CCD                 db    1
.rdata:00973CCE                 db    1
.rdata:00973CCF                 db    1
.rdata:00973CD0                 db    1
.rdata:00973CD1                 db    1
.rdata:00973CD2                 db    0
.rdata:00973CD3                 db    1
.rdata:00973CD4                 db    1
.rdata:00973CD5                 db    1
.rdata:00973CD6                 db    1
.rdata:00973CD7                 db    1
.rdata:00973CD8                 db    1
.rdata:00973CD9                 db    1
.rdata:00973CDA                 db    1
.rdata:00973CDB                 db    0
.rdata:00973CDC                 db    1
.rdata:00973CDD                 db    1
.rdata:00973CDE                 db    0
.rdata:00973CDF                 db    1
.rdata:00973CE0                 db    1
.rdata:00973CE1                 db    0
.rdata:00973CE2                 db    1
.rdata:00973CE3                 db    1
.rdata:00973CE4                 db    1
.rdata:00973CE5                 db    1
.rdata:00973CE6                 db    1
.rdata:00973CE7                 db    1
.rdata:00973CE8                 db    1
.rdata:00973CE9                 db    1
.rdata:00973CEA                 db    0
.rdata:00973CEB                 db    1
.rdata:00973CEC                 db    1
.rdata:00973CED                 db    0
.rdata:00973CEE                 db    1
.rdata:00973CEF                 db    1
.rdata:00973CF0                 db    0
.rdata:00973CF1                 db    1
.rdata:00973CF2                 db    1
.rdata:00973CF3                 db    1
.rdata:00973CF4                 db    1
.rdata:00973CF5                 db    1
.rdata:00973CF6                 db    0
.rdata:00973CF7                 db    1
.rdata:00973CF8                 db    1
.rdata:00973CF9                 db    1
.rdata:00973CFA                 db    1
.rdata:00973CFB                 db    1
.rdata:00973CFC                 db    0
.rdata:00973CFD                 db    1
.rdata:00973CFE                 db    1
.rdata:00973CFF                 db    0
.rdata:00973D00                 db    1
.rdata:00973D01                 db    1
.rdata:00973D02                 db    0
.rdata:00973D03                 db    1
.rdata:00973D04                 db    0
.rdata:00973D05                 db    1
.rdata:00973D06                 db    0
.rdata:00973D07                 db    0
.rdata:00973D08                 db    1
.rdata:00973D09                 db    0
.rdata:00973D0A                 db    0
.rdata:00973D0B                 db    1
.rdata:00973D0C                 db    0
.rdata:00973D0D                 db    0
.rdata:00973D0E                 db    0
.rdata:00973D0F                 db    0
.rdata:00973D10                 db    0
.rdata:00973D11                 db    1
.rdata:00973D12                 db    0
.rdata:00973D13                 db    1
.rdata:00973D14                 db    1
.rdata:00973D15                 db    0
.rdata:00973D16                 db    0
.rdata:00973D17                 db    1
.rdata:00973D18                 db    0
.rdata:00973D19                 db    0
.rdata:00973D1A                 db    1
.rdata:00973D1B                 db    1
.rdata:00973D1C                 db    0
.rdata:00973D1D                 db    1
.rdata:00973D1E                 db    0
.rdata:00973D1F                 db    1
.rdata:00973D20                 db    1
.rdata:00973D21                 db    0
.rdata:00973D22                 db    0
.rdata:00973D23                 db    0
```

將每 15 個字為一組，其中 3 個為一行的排法，排出第一個字為

```
111
010
010
010
010
```
看起來就是 `T`，那麼將 22 個字解完就是 `THE FLAG is hitcon{BOOM!}` 

如果出題者對這些字做 XOR 就比較困難了，好險好險
* * * *
最後我們拿了第一名，感謝 Lazyhacker 、HITCON CMT 籌備團隊、mini wargame 出題者以及幫助過我的人

後記: 結束後，跟 `d3vc0r3` 交流，得知 Web 滲透測試專家對於 Reverse 題 `winmine.exe` 的解法相當可愛，每猜中一個字，就利用 VMware 做 snapshot，如此一來，就可以輕鬆快樂的猜下一個字了，真是太睿智XD　
