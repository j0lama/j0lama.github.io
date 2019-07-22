---
layout: post
title: Cracking Sublime Text 3
---

Sublime Text is a famous multi-platform free text editor but if you want to get full support and updates you have to pay 70$ license. This is why I decided to make this tutorial where I explain how to crack this useful tool.

The tools I will use to crack it are the Hooper dissasembler and Radare2, a reverse engineering unix framework.

First, It will be necessary to find an error message when we introduce an invalid license in Sublime.
<p align="center">
      <img src="/images/crack-subl/1.png">
</p>

The message is:
```
"That license key doesn't..."
```

With the error message in mind, it can be searched as string in Hooper string list:
<p align="center">
      <img src="/images/crack-subl/2.png">
</p>

With the error message pointer, it will be necessary to see in which parts of the code, this string is used and to do it, Hooper allows us see the cross references as IDA Pro do:
<p align="center">
      <img src="/images/crack-subl/3.png">
</p>

As can be noticed in the image, the error message is used in the address 0x42F1F4. This address corresponds with a *mov* instruction in a code that is accessed from other address with a jump:
<p align="center">
      <img src="/images/crack-subl/4.png">
</p>

The caller of the previous code section at 0x42F1E7 (cmp eax, 0x2) is a *je* (jump if equal) function at 0x42F0FD:
<p align="center">
      <img src="/images/crack-subl/5.png">
</p>

It can be appreciated at 0x42F0EF a *cmp* instruction which is the instruction that decides if the license is valid or not:
<p align="center">
      <img src="/images/crack-subl/6.png">
</p>

If we analyze the function we can see that the *eax* register is compared with the value 0x1 and if the comparison result is true, then that means that the license is valid.

So, if we replace 0x1 in the comparison with the invalid license code then the result will be true if the license is invalid. The code for the invalid license is 0x2 and we just have to replace 0x1 by 0x2 at 0x42F0EF using *radare2*.

After open Sublime with radare2, we move the poiter to the 0x42F0EF to change the value. To check if we are in the correct instruction I use the command *pd 5* to disassembly 5 instructions:
<p align="center">
      <img src="/images/crack-subl/7.png">
</p>

To replace 0x1 I use the command *wx 83F802* (*wx* to write in hex and 83F802 is the binary code of the instruction with the 0x1 value replaced by 0x2). With this done, it's recommended to check if the instruction has been replaced correctly disassembling 5 instructions again.
<p align="center">
      <img src="/images/crack-subl/8.png">
</p>

Now it's time to check if everything works as expected bypassing the license security checking:
<p align="center">
      <img src="/images/crack-subl/9.png">
</p>

License checking bypassed successfully!

#### Conclusions

I hope you learned something new and over all enjoyed reading this post as much I did writing it.

If you want more post like this one make it me know thorugh Twitter or buying me a [Ko-Fi](https://ko-fi.com/jolama)

