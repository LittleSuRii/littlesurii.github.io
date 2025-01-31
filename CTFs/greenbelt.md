---
layout: default
---
Last modified:  
2025-01-23  

# 感想  

全部clear了再补。

## Sandboxing lvl18

lvl18这道题卡了一天, 一开始我的解题思路是如何为一个mounted mother folder为nosuid的情况下为子文件夹添加逃避nosuid的flag以达成chmod u+s能够完成suid的执行逃逸, 想来想去看来是思路太离谱了。  
Live Session 2以及 [2024.11.07](https://www.youtube.com/watch?v=Qp6cdnkZXWQ&ab_channel=pwn.college) 的视频提供了非常多的解题方式, 但我没能使用capset去为一个在nosuid情况下的elf提供更多的权限以解题。  
考虑了很多最后在题目的提示以及路径判断里找到了答案。整体上的解题思路可以说是几乎照抄视频的方法了。可是我没能在shellcode里成功执行execve /bin/bash, 即便如此我还是成功脱狱了。  

## Race Conditions lvl9-11

卡得比较久的是lvl9.1以及lvl10.1。这两道题的核心在于如何创建共享内存信息的子进程。没想到在B站找到了[救命视频](https://www.bilibili.com/video/BV1dc411i7Cs)。
An intresing thing I found in here is in the Angr of challenge 11.1, the Pseudocode of the string judement shows " if str equals to 'receive message', then do fwrite Unrecognized choice ".
But in IDA, it works correctly.
![Angr](https://littlesurii.github.io/imgs/pwn/pwn11.1_Angr.png)
![IDA](https://littlesurii.github.io/imgs/pwn/pwn11.1_IDA.png)