---
layout: default
---
Last modified:  
2024-11-12  

# 感想

Program Security内的四小模块里最有意思的题目基本上是最后一题。  
完成四个模块的内容后让我加深了对栈, 内存, 寄存器, Canary的概念以及对逆向过程的理解。  

## Program Exploitation lvl11.0

Yan85_64的题目是真的非常有趣!Discord频道上有非常多这道题相关的Discussion, 所以在此我只描述一种JIT的解题思路, 因为我没能实现OOB的解题方法。  
以下是我的心路历程:

0. 看不懂忆点。这怎么调用的?  
1. 这什么东西啊? IMM怎么成功的?  
2. 观察寄存器调用时产生的某些影响, 并找到那个影响。  
3. 什么? offset居然是这么保存在内存里的!?  
4. 什么? 内存外居然......  
5. Wow! 这是什么套题的结合! 居然还能用上shellcode injection模块里的......  
6. 我的exploit到底成没成功......欸它怎么loop了!?  
7. 害得是shellcode啊, 越少的Bytes越成功!  
8. 我成功了! 这道题是真的有趣!  
9. lvl11.1>哥们你只是换个顺序的话那这就没有意思了。  

不得不说卡我最久且最让我感受到做题快乐和成长的就是这道[lvl11.0](https://pwn.college/program-security/program-exploitation/)了。